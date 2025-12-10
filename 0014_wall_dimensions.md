---
post_number: "0014"
title: "Wall Dimensions"
slug: "wall_dimensions"
author: "Jeremy Tammik"
tags: ['csharp', 'elements', 'geometry', 'revit-api', 'selection', 'views', 'walls', 'windows']
source_file: "0014_wall_dimensions.htm"
original_url: "https://thebuildingcoder.typepad.com/blog/0014_wall_dimensions.html"
---

### Wall Dimensions

After the diversion exploring the geometry related information and samples provided with the basic Revit SDK, let us finally get into an example of some own geometry related code. We present an algorithm to dimension an arbitrary Revit element containing a solid in its geometry. The implementation is tailored to handle wall elements, but the functionality can be applied to any other element containing geometry as well.

The idea is pretty simple, and possibly not awfully useful for real life, but geometrically elegant and interesting. We analyse the wall geometry and determine the maximum distance between all planar wall faces with parallel normal vectors. The maximum distance for any given normal vector is assumed to be the wall dimension in that direction. Non-planar faces are ignored. For each planar face, we determine its normal vector and origin. For all faces sharing the same normal vector, we collect all the origin points. After completing this collection process, we determine the maximal distance along the normal vector between all the faces sharing this normal. For a quadrilateral wall, the result will be the three dimensions one expects. For irregular walls with planar faces, additional dimensions will be detected in all directions perpendicular to two or more planar faces.

Here is the code for the external command and its support routines:

```csharp
#region Header
//
// CmdWallDimensions.cs - determine wall dimensions
// by iterating over wall geometry faces
//
// Copyright (C) 2008 by Jeremy Tammik,
// Autodesk Inc. All rights reserved.
//
#endregion // Header

#region Namespaces
using System;
using System.Collections.Generic;
using System.Diagnostics;
using Autodesk.Revit;
using Autodesk.Revit.Elements;
using Autodesk.Revit.Geometry;
using CmdResult = Autodesk.Revit.IExternalCommand.Result;
using RvtElement = Autodesk.Revit.Element;
using GeoElement = Autodesk.Revit.Geometry.Element;
using NormalAndOrigins
  = System.Collections.Generic.KeyValuePair
    < Autodesk.Revit.Geometry.XYZ
    , System.Collections.Generic.List
      <Autodesk.Revit.Geometry.XYZ>>;
#endregion // Namespaces

namespace BuildingCoder
{
  /// <summary>
  /// List dimensions for a quadrilateral wall with
  /// openings. In this algorithm, we collect all
  /// the faces with parallel normal vectors and
  /// calculate the maximal distance between any
  /// two pairs of them. This is the wall dimension
  /// in that direction.
  /// </summary>
  class CmdWallDimensions
  {
    #region Geometry
    const double \_eps = 1.0e-9;

    /// <summary>
    /// Check whether two real numbers are equal
    /// </summary>
    static bool DoubleEqual( double a, double b )
    {
      return Math.Abs( a - b ) < \_eps;
    }

    /// <summary>
    /// Check whether two vectors are parallel
    /// </summary>
    static bool XyzParallel( XYZ a, XYZ b )
    {
      double angle = a.Angle( b );
      return \_eps > angle
        || DoubleEqual( angle, Math.PI );
    }
    #endregion // Geometry

    /// <summary>
    /// Retrieve the planar face normal and origin
    /// from all of the solid's planar faces and
    /// insert them into the map mapping face normals
    /// to a list of all origins of different faces
    /// sharing this normal.
    /// </summary>
    /// <param name="m">Map mapping each normal vector
    /// to a list of the origins of all planar faces
    /// sharing this normal direction</param>
    /// <param name="solid">Input solid</param>
    void getFaceData(
      Dictionary<XYZ, List<XYZ>> m,
      Solid solid )
    {
      int i;
      FaceArray faces = solid.Faces;
      foreach( Face face in solid.Faces )
      {
        PlanarFace planarFace = face as PlanarFace;
        if( null != planarFace )
        {
          XYZ normal = planarFace.Normal;
          XYZ origin = planarFace.Origin;
          List<XYZ> normals = new List<XYZ>( m.Keys );
          i = normals.FindIndex(
            delegate( XYZ v )
            {
              return XyzParallel( v, normal );
            } );
          if( -1 == i )
          {
            Debug.WriteLine( string.Format(
              "Face at {0} has new normal {1}",
              Util.PointString( origin ),
              Util.PointString( normal ) ) );
            m.Add( normal, new List<XYZ>() );
            m[normal].Add( origin );
          }
          else
          {
            Debug.WriteLine( string.Format(
              "Face at {0} normal {1} matches {2}",
              Util.PointString( origin ),
              Util.PointString( normal ),
              Util.PointString( normals[i] ) ) );
            m[normals[i]].Add( origin );
          }
        }
      }
    }

    public CmdResult Execute(
      ExternalCommandData commandData,
      ref string message,
      ElementSet elements )
    {
      Application app = commandData.Application;
      Document doc = app.ActiveDocument;
      Selection sel = doc.Selection;
      Options o = app.Create.NewGeometryOptions();
      // map planar face normals to face origins:
      Dictionary<XYZ, List<XYZ>> m
        = new Dictionary<XYZ, List<XYZ>>();
      string s, msg = string.Empty;
      int i;
      foreach( RvtElement e in sel.Elements )
      {
        Wall wall = e as Wall;
        if( null != wall )
        {
          s = string.Format( "Wall <{0} {1}>:",
            wall.Name, wall.Id.Value );
          Debug.WriteLine( s );
          if( 0 < msg.Length ) { msg += "\n\n"; }
          msg += s;
          GeoElement ge = wall.get\_Geometry( o );
          GeometryObjectArray objs = ge.Objects;
          foreach( GeometryObject obj in objs )
          {
            Solid solid = obj as Solid;
            if( null != solid )
            {
              getFaceData( m, solid );
            }
          }
          int j, n;
          double d, dmax;
          foreach( NormalAndOrigins pair in m )
          {
            dmax = 0;
            XYZ normal = pair.Key.Normalized;
            List<XYZ> pts = pair.Value;
            n = pts.Count;
            if( 1 == n )
            {
              s = string.Format(
                "Only one wall face in "
                + "direction {0} found.",
                Util.PointString( normal ) );
            }
            else
            {
              for( i = 0; i < n - 1; ++i )
              {
                for( j = i + 1; j < n; ++j )
                {
                  XYZ v = pts[i].Subtract( pts[j] );
                  d = v.Dot( normal );
                  if( d > dmax )
                  {
                    dmax = d;
                  }
                }
              }
              s = string.Format(
                "Max wall dimension in "
                + "direction {0} is {1} feet.",
                Util.PointString( normal ),
                Util.RealString( dmax ) );
            }
            Debug.WriteLine( s );
            msg += "\n" + s;
          }
          m.Clear();
        }
      }
      if( 0 == msg.Length )
      {
        msg = "Please select some walls.";
      }
      Util.InfoMsg( msg );
      return CmdResult.Succeeded;
    }
  }
}
```

We use regions to structure the code. It allows us to collapse bits of code that we are currently not interested in to reduce screen clutter and improve the overview. The main geometrical analysis is implemented in getFaceData(). It traverses the solid passed in and collects all the planar face normal vectors and origins, using the normals as dictionary keys, and creating lists of origin points as dictionary values. Once this information has been assembled, we traverse it to format it into a string to be presented in a message box, as well as listing it in the Visual Studio debug output window.

The namespace is BuildingCoder, and the command name CmdWallDimensions, so the information to add to Revit.ini is something like this:

```
[ExternalCommands]
ECCount=1
ECName1=Wall Dimensions
ECDescription1=Extract wall solid and list all its dimensions
ECAssembly1=C:\bin\BuildingCoder.dll
ECClassName1=BuildingCoder.CmdWallDimensions
```

Here is a sample wall to run it on, created by the external command defined by Lab2\_0\_CreateLittleHouse in the Revit API introduction labs:

![Little House Wall Selected](img/little_house_3d_wall.png)

Here is the result of selecting the wall and running the command:

![Wall Dimensions](img/wall_dimensions.png)

This is the log data printed to the Visual Studio debug output window generated during the process, to help understand how the faces are analysed one by one to determine whether their normal either matches an existing or defines a new one:

```
Wall <Generic - 200mm 127248>:
Face at (0,-0.33,0) has new normal (0,-1,0)
Face at (12.98,0.33,0) has new normal (-1,0,0)
Face at (0,-0.33,0) has new normal (0,0,-1)
Face at (23.29,0,0) normal (1,0,0) matches (-1,0,0)
Face at (22.97,-0.33,13.12) normal (0,0,1) matches (0,0,-1)
Face at (-0.33,13.12,0) normal (-1,0,0) matches (-1,0,0)
Face at (9.98,0.33,7) normal (1,0,0) matches (-1,0,0)
Face at (12.98,0.33,7) normal (0,0,-1) matches (0,0,-1)
Face at (5.08,-0.33,3.94) normal (1,0,0) matches (-1,0,0)
Face at (5.08,-0.33,5.94) normal (0,0,-1) matches (0,0,-1)
Face at (6.41,-0.33,5.94) normal (-1,0,0) matches (-1,0,0)
Face at (6.41,-0.33,3.94) normal (0,0,1) matches (0,0,-1)
Face at (16.56,-0.33,3.94) normal (1,0,0) matches (-1,0,0)
Face at (16.56,-0.33,5.94) normal (0,0,-1) matches (0,0,-1)
Face at (17.89,-0.33,5.94) normal (-1,0,0) matches (-1,0,0)
Face at (17.89,-0.33,3.94) normal (0,0,1) matches (0,0,-1)
Face at (0,0.33,0) normal (0,1,0) matches (0,-1,0)
Max wall dimension in direction (0,-1,0) is 0.66 feet.
Max wall dimension in direction (-1,0,0) is 18.22 feet.
Max wall dimension in direction (0,0,-1) is 13.12 feet.
Wall <Generic - 200mm 127248>:
Max wall dimension in direction (0,-1,0) is 0.66 feet.
Max wall dimension in direction (-1,0,0) is 18.22 feet.
Max wall dimension in direction (0,0,-1) is 13.12 feet.
```