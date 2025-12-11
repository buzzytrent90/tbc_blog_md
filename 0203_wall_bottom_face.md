---
ai_optimized: true
author: Jeremy Tammik
complexity_score: 2.6
content_type: qa
optimization_date: '2025-12-11T11:44:13.543538'
original_url: https://thebuildingcoder.typepad.com/blog/0203_wall_bottom_face.html
post_number: '0203'
reading_time_minutes: 2
series: general
slug: wall_bottom_face
source_file: 0203_wall_bottom_face.htm
tags:
- csharp
- elements
- geometry
- revit-api
- views
- walls
- windows
title: Bottom Face of a Wall
word_count: 429
---

### Bottom Face of a Wall

We already went into quite a bit of detail analysing the geometry of walls and other elements, but the topic keeps cropping up again anyway.
An overview of the preceding posts on this topic is given in the discussion of
[cylindrical columns](http://thebuildingcoder.typepad.com/blog/2009/04/cylindrical-column.html).
Here is a case handled by Joe Ye which I thought might complement the previous posts on this topic, since it is so simple and minimal.

**Question:**
How can I find the bottom face of a wall?

**Answer:**
We can query the wall for its solid geometry using the Element.Geometry property, which is accessed by the get\_Geometry method in C#.
From the returned solid, all faces can be accessed.
The bottom face of the wall has a normal vector equal to (0,0,-1), and in most cases this is the unique for the bottom face.
So we just iterate over all faces, and stop when we find one whose normal vector is vertical and has a negative Z coordinate.

I implemented a new external command named CmdWallBottomFace to demonstrate this.
Here is the code of its Execute method.
As always, it returns Failed even if it did in fact succeed, so that no changes are registered in the BIM, since the command does nothing to modify the model:

```csharp
Application app = commandData.Application;
Document doc = app.ActiveDocument;

string s = "a wall, to retrieve its bottom face";

Wall wall = Util.SelectSingleElementOfType(
  doc, typeof( Wall ), s ) as Wall;

if( null == wall )
{
  message = "Please select a wall.";
}
else
{
  Options opt = app.Create.NewGeometryOptions();
  GeoElement e = wall.get\_Geometry( opt );

  foreach( GeometryObject obj in e.Objects )
  {
    Solid solid = obj as Solid;
    if( null != solid )
    {
      foreach( Face face in solid.Faces )
      {
        PlanarFace pf = face as PlanarFace;
        if( null != pf )
        {
          if( Util.IsVertical( pf.Normal, \_tolerance )
            && pf.Normal.Z < 0 )
          {
            Util.InfoMsg( string.Format(
              "The bottom face area is {0},"
              + " and its origin is at {1}.",
              Util.RealString( pf.Area ),
              Util.PointString( pf.Origin ) ) );
            break;
          }
        }
      }
    }
  }
}
return CmdResult.Failed;
```

When the command is executed and a wall selected, the area and origin point of its bottom face is reported in a message box and in the Visual Studio debug output window:

![Wall bottom face area and origin](img/wall_bottom_face.png)

```
The bottom face area is 265.82,
and its origin is at (30.76,20.57,-9.84).
```

Here is
[version 1.1.0.44](zip/bc11044.zip)
of the complete Visual Studio solution including the new command.

Thank you very much Joe for this answer!