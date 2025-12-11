---
ai_optimized: true
author: Jeremy Tammik
complexity_score: 3.1
content_type: qa
optimization_date: '2025-12-11T11:44:13.248482'
original_url: https://thebuildingcoder.typepad.com/blog/0025_element_materials.html
post_number: '0025'
reading_time_minutes: 3
series: elements
slug: element_materials
source_file: 0025_element_materials.htm
tags:
- csharp
- doors
- elements
- family
- filtering
- geometry
- revit-api
- selection
- views
- walls
- windows
title: Element Materials
word_count: 652
---

### Element Materials

I recently heard several queries on how to access the surface material of a wall or other building element. There are several different ways to access materials defined on building elements. One method is discussed in the section 16.3 Element Material in the
Revit 2009 API Developer Guide.
Another method goes through the geometry and retrieves the material associated with each face. We will discuss the latter approach in this and the next post.

Retrieving the material associated with element geometry is actually very simple, similar to and simpler than the geometrical traversal that we looked at in the post on
[Wall Dimensions](http://thebuildingcoder.typepad.com/blog/2008/09/wall-dimensions.html).
The geometry displayed in Revit generally has a solid associated with it, each solid has a set of faces, and each face has a material, which can be queried using its MaterialElement property.
Therefore, we can simply query the building element for its geometry, traverse that, pick out all solids, ask each one for its faces, and list their materials. Here is the code to do so:

```csharp
using System;
using System.Collections.Generic;
using System.Diagnostics;
using Autodesk.Revit;
using Autodesk.Revit.Elements;
using Autodesk.Revit.Geometry;
using CmdResult = Autodesk.Revit.IExternalCommand.Result;
using RvtElement = Autodesk.Revit.Element;
using GeoElement = Autodesk.Revit.Geometry.Element;

namespace BuildingCoder
{
  class CmdGetMaterials : IExternalCommand
  {
    public List<string> GetMaterials( GeoElement geo )
    {
      List<string> materials = new List<string>();
      foreach( GeometryObject o in geo.Objects )
      {
        if( o is Solid )
        {
          Solid solid = o as Solid;
          foreach( Face face in solid.Faces )
          {
            string s = face.MaterialElement.Name;
            materials.Add( s );
          }
        }
      }
      return materials;
    }

    public CmdResult Execute(
      ExternalCommandData commandData,
      ref string message,
      ElementSet elements )
    {
      Application app = commandData.Application;
      Document doc = app.ActiveDocument;
      Selection sel = doc.Selection;
      Options opt = app.Create.NewGeometryOptions();
      string msg = string.Empty;
      int i, n;

      foreach( RvtElement e in sel.Elements )
      {
        GeoElement geo = e.get\_Geometry( opt );
        List<string> materials = GetMaterials( geo );

        msg += "\n" + Util.ElementDescription( e );

        n = materials.Count;
        if( 0 == n )
        {
          msg += " has no materials.";
        }
        else
        {
          i = 0;
          msg += string.Format(
            " has {0} material{1}:",
            n, Util.PluralSuffix( n ) );

          foreach( string s in materials )
          {
            msg += string.Format(
              "\n  {0} {1}", i++, s );
          }
        }
      }
      if( 0 == msg.Length )
      {
        msg = "Please select some elements.";
      }
      Util.InfoMsg( msg );
      return CmdResult.Succeeded;
    }
  }
}
```

Here is the little house sample, consisting of four walls, a floor, a roof, a door and two windows, which we can run this code on:

![Little House](img/little_house_3d_view.png)

Here is the result of selecting all elements of the little house and running the command:

```
Walls <127248 Generic - 200mm> has 17 materials:
  0 Default Wall
  1 Default Wall
  2 Default Wall
  3 Default Wall
  . . .
  16 Default Wall
Walls <127249 Generic - 200mm> has 6 materials:
  0 Default Wall
  1 Default Wall
  . . .
  5 Default Wall
Walls <127250 Generic - 200mm> has 6 materials:
  0 Default Wall
  1 Default Wall
  . . .
  5 Default Wall
Walls <127251 Generic - 200mm> has 6 materials:
  0 Default Wall
  . . .
  5 Default Wall
Doors M_Single-Flush <127252 0915 x 2134mm> has no materials.
Windows M_Fixed <127255 0406 x 0610mm> has no materials.
Windows M_Fixed <127258 0406 x 0610mm> has no materials.
Floors <127266 Generic Floor - 400mm> has 6 materials:
  0 Default Roof
  1 Default Roof
  . . .
  5 Default Roof
Roofs <127269 Generic Roof - 300mm> has 12 materials:
  0 Default Roof
  1 Default Roof
  . . .
  11 Default Roof
```

Notice that the sample in its current state does not report any materials for the door or windows. This is because these elements are family instances, and their geometry is nested in a geometry instance, which the current code does not traverse into. In a future instalment, we will rectify that to list the materials nested in family instances as well.

I am adding the complete Visual Studio solution [here](http://thebuildingcoder.typepad.com/blog/files/bc1005.zip). This version 1.0.0.5 includes all commands we discussed so far: CmdListWalls, CmdRelationshipInverter, CmdWallDimensions, CmdFilterPerformance and the new CmdGetMaterials.