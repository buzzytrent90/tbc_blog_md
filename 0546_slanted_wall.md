---
ai_optimized: true
author: Jeremy Tammik
complexity_score: 3.5
content_type: qa
optimization_date: '2025-12-11T11:44:14.145931'
original_url: https://thebuildingcoder.typepad.com/blog/0546_slanted_wall.html
post_number: '0546'
reading_time_minutes: 3
series: general
slug: slanted_wall
source_file: 0546_slanted_wall.htm
tags:
- csharp
- elements
- filtering
- geometry
- revit-api
- transactions
- walls
title: The FaceWall Class and Slanted Walls
word_count: 678
---

### The FaceWall Class and Slanted Walls

We have never previously taken looked at the FaceWall class.
Here is a question on this handled by Joe Ye, which also addresses an old
[comment](http://thebuildingcoder.typepad.com/blog/2009/02/creating-a-wall-with-a-sloped-profile.html?cid=6a00e553e168978833011571944bd1970b#comment-6a00e553e168978833011571944bd1970b) by
Deric Wallace on
[creating a wall with a sloped profile](http://thebuildingcoder.typepad.com/blog/2009/02/creating-a-wall-with-a-sloped-profile.html).
Deric wishes to create a wall like this programmatically:

![Slanted wall](img/slanted_wall.png)

This was not easily achievable in Revit 2010.
The access to slanted walls is improved in Revit 2011, as we can see from Joe's answer below:

**Question:** How can I use the API to distinguish a slanted wall from a normal one?

Once I have a slanted wall, how can I determine its angle?

![Slanted wall angle](img/slanted_wall2.png)

**Answer:** Slanted walls are represented by the FaceWall class, whereas normal walls use the Wall class.
You can thus take advantage of the Revit element's .NET class to distinguish between slanted and non-slanted walls.

With regard to getting the angle of a slanted wall, you can determine one of its side faces, calculate the side face's normal vector, and then determine the angle between that and the vertical vector or Z axis (0,0,1):
![Slanted wall angle calculation](img/slanted_wall3.png)

Now the question remains how to obtain the slanted wall's upside face.
To do so, first retrieve the slanted wall solid using its Geometry property.
Then go through each face of the solid.
If the face area times the wall thickness equals the wall volume, and the face normal vector's Z coordinate is positive, then this face is the upside face we are looking for.

As said, we can then calculate the angle between vertical vector (0,0,1) and the upside face's normal to determine the slanted wall angle.

Here is the code to achieve all of this:
```csharp
using System;
using System.Collections.Generic;
using Autodesk.Revit.ApplicationServices;
using Autodesk.Revit.Attributes;
using Autodesk.Revit.DB;
using Autodesk.Revit.UI;

[TransactionAttribute( TransactionMode.ReadOnly )]
[RegenerationAttribute( RegenerationOption.Manual )]
public class GetSlantedWallAndAngle : IExternalCommand
{
  public Result Execute(
    ExternalCommandData commandData,
    ref string messages,
    ElementSet elements )
  {
    UIApplication app = commandData.Application;
    Document doc = app.ActiveUIDocument.Document;

    FilteredElementCollector collector
      = new FilteredElementCollector( doc )
        .OfCategory( BuiltInCategory.OST\_Walls );

    FaceWall slantedWall = null;

    foreach( Element elem in collector )
    {
      // use the class to identify slanted wall

      if( elem is FaceWall )
      {
        slantedWall = elem as FaceWall;
        break;
      }
    }

    if( slantedWall == null )
    {
      messages = "There is no slanted wall in this model";
      return Result.Failed;
    }

    // retrieve geometry

    Options options = app.Application.Create
      .NewGeometryOptions();

    GeometryElement geoElement
      = slantedWall.get\_Geometry( options );

    // get the only solid

    Solid wallSolid = null;

    foreach( GeometryObject geoObject
      in geoElement.Objects )
    {
      if( geoObject is Solid )
      {
        wallSolid = geoObject as Solid;
        break;
      }
    }

    // get the wall width

    ElementId typeId = slantedWall.GetTypeId();

    WallType type = doc.get\_Element( typeId )
      as WallType;

    double dWidth = type.Width;

    double dTolerance = 0.01;
    double dAngle = 0.0;
    foreach( PlanarFace face in wallSolid.Faces )
    {
      if( dTolerance > Math.Abs(
        face.Area \* dWidth - wallSolid.Volume ) )
      {
        if( face.Normal.Z > dTolerance )
        {
          // calculate the angle

          XYZ verticalVector = XYZ.BasisZ;

          dAngle = verticalVector.AngleTo(
            face.Normal ) \* 180 / 3.1415926;

          TaskDialog.Show(
            "Slanted wall angle to XOY plane",
            dAngle.ToString() );

          break;
        }
      }
    }
    return Result.Succeeded;
  }
}
```

Here is the complete Visual Studio code project
[slanted\_wall.zip](zip/slanted_wall.zip).

Many thanks to Joe for this answer and sample code!

One last-minute idea for a possible enhancement of the filtering code that came to me while editing this post is that since we are searching for walls whose .NET class is FaceWall, the filtering could probably be further optimised by using
```csharp
  FilteredElementCollector collector
    = new FilteredElementCollector( doc )
      .OfClass( typeof( FaceWall ) )
      .OfCategory( BuiltInCategory.OST\_Walls );
```

Then the explicit iteration after setting up the filter could be avoided.
Maybe we could even skip the check for the category, since FaceWall instances presumably always will have the OST\_Walls built-in category.
Eliminating the category check would probably have an absolutely negligible effect, though.