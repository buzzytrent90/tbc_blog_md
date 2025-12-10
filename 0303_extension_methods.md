---
post_number: "0303"
title: "GetPolygon Extension Methods"
slug: "extension_methods"
author: "Jeremy Tammik"
tags: ['csharp', 'elements', 'geometry', 'levels', 'python', 'references', 'revit-api', 'rooms', 'views', 'walls']
source_file: "0303_extension_methods.htm"
original_url: "https://thebuildingcoder.typepad.com/blog/0303_extension_methods.html"
---

### GetPolygon Extension Methods

Here is a question from Daren Thomas of ETH, the technical university of Zürich, that has been sitting around in my inbox for quite a while.
Daren works in the group
[Professur für Gebäudetechnik, Institut für Hochbautechnik](http://www.gt.arch.ethz.ch).
One area of their research is detailed building energy analysis, and one piece of basic utility functionality required for this analysis are
[Boolean operations](http://thebuildingcoder.typepad.com/blog/2009/02/boolean-operations-for-2d-polygons.html) on
the polygonal areas of rooms on different levels.

**Question:** We are developing a plug-in for Revit Architecture to perform energy calculations in a building model.
To do so, we need to read the model with its rooms and the relationships between all their surfaces such as walls, floors, ceilings etc.

We are still working on establishing the relationships between the rooms on one hand and floors and roofs or ceilings on the other.
Specifically, we would like to determine the intersections areas on a floor of the rooms below and above it.
You mentioned the
[General Polygon Clipper GPC](http://thebuildingcoder.typepad.com/blog/2009/02/boolean-operations-for-2d-polygons.html) library,
and that looks like a good starting point, although it does require an additional library with the associated administrative overhead and licensing complications.

Can we use the Revit API determine the room boundaries above and below, like we can for the walls on the sides?

**Answer:** Look at the Revit SDK sample
[RoofsRooms](http://thebuildingcoder.typepad.com/blog/2008/09/roomsroofs-sdk.html),
and the
[ClosedShell](http://thebuildingcoder.typepad.com/blog/2009/12/crop-3d-view-to-room.html) property
on the Room class.
You can also make use of the
[FindReferencesByDirection](http://thebuildingcoder.typepad.com/blog/2009/10/analytical-support-tolerance.html) method.
We suggested some further ideas when looking at the related problems of
[room and wall adjacency](http://thebuildingcoder.typepad.com/blog/2009/01/room-and-wall-adjacency.html),
their
[adjacent areas](http://thebuildingcoder.typepad.com/blog/2009/01/room-and-wall-adjacent-area.html), and
[space adjacency](http://thebuildingcoder.typepad.com/blog/2009/07/space-adjacency.html).

**Response:** Currently we are following the GPC route.
The library is really fast.
According to our profiling results, most of the time is being spent in other stuff, unrelated to the polygon clipping.
I expect to attack the problem from the "Revit-API-only" angle sometime next month, when license issues get more complicated – at the moment, we need to release a version quickly for students.
A commercial version is not planned until later.

I will keep you informed if we try the "Revit-API-only" angle and my experiences with it.
One point that doesn't seem clear after looking at RoofsRooms and GeomUtil.cs is how to find the intersecting area – this is something the GPC library makes very easy.

The basic algorithm works as follows:
```csharp
for floor in document:
for roomAbove in rooms\_with\_same\_level\_as\_floor:
aboveIntersection = intersect
floor.GetPolygon() with roomBelow.GetPolygon()
if aboveIntersection is not empty:
for roomBelow in rooms\_with\_level\_just\_below\_floor:
belowIntersection = intersect
aboveIntersection with roomBelow.GetPolygon()
aboveIntersection = difference of
aboveIntersection and belowIntersection
if belowIntersection is not empty:
=> roomAbove and roomBelow share
belowIntersection of floor
if aboveIntersection is not empty:
=> roomAbove has the remaining bits of
aboveIntersection of floor all to itself
```

Note: this algorithm has not been optimized for performance yet, but profiling indicates so far, that it is not a bottleneck and thus negligible for the moment.

I'm not quite sure if the exact C# code is of interest to you, as it works on an abstract building model that is not Revit specific.
I wouldn't mind sharing some nifty stuff with
[Extension Methods](http://msdn.microsoft.com/en-us/library/bb383977.aspx) –
a new technology in C# 3.0 that I find especially useful for working with plug-in APIs, as subclassing cannot be used to add functionality to types:
```python
internal static class RevitFloorExtensions
{
  internal static Polygon GetPolygon(
    this Floor floor )
  {
    // code copied from here:
    // http://thebuildingcoder.typepad.com/blog/2009/02/
    // boolean-operations-for-2d-polygons.html

    Options geomOptions
      = floor.Document.Application
        .Create.NewGeometryOptions();

    Element elem = floor.get\_Geometry( geomOptions );

    var vertices = new List();

    foreach( var obj in elem.Objects )
    {
      var solid = obj as Solid;
      if( null != solid )
      {
        Face face = solid.Faces.get\_Item( 0 );
        EdgeArray loop = face.EdgeLoops.get\_Item( 0 );
        foreach( Edge edge in loop )
        {
          XYZArray edgePts = edge.Tessellate();
          int n = edgePts.Size;
          for( int i = 0; i < n - 1; ++i )
          {
            XYZ p = edgePts.get\_Item( i );
            vertices.Add( new Vertex(
                p.X \* Constant.FeetToMeter,
                p.Y \* Constant.FeetToMeter ) );
          }
        }
        break;
      }
    }
    var vertexList = new VertexList{
      NofVertices = vertices.Count,
      Vertex = vertices.ToArray() };

    var result = new Polygon();
    result.AddContour( vertexList, false );
    return result;
  }
}
```

The above code allows me to go floor.GetPolygon() – just syntactic sugar, but practical nonetheless.

Here is an equivalent method for rooms:
```csharp
/// <summary>
/// Retrieves a polygon describing the room boundaries.
/// </summary>
public static Polygon GetPolygon( this Room room )
{
  var result = new Polygon();
  var vertices = new List<Vertex>();

  foreach( BoundarySegmentArray bsa in room.Boundary )
  {
    foreach( BoundarySegment bs in bsa )
    {
      var revitWall = bs.Element as Wall;

      if( revitWall == null ) continue; // ignore non-walls here

      var X = bs.Curve.get\_EndPoint( 0 ).X
        \* Constant.FeetToMeter;

      var Y = bs.Curve.get\_EndPoint( 0 ).Y
        \* Constant.FeetToMeter;

      vertices.Add( new Vertex( X, Y ) );
    }
    var vertexList = new VertexList
    {
      NofVertices = vertices.Count,
      Vertex = vertices.ToArray()
    };
    result.AddContour( vertexList, false );
  }
  return result;
}
```

Using these, reading the area of a polygon intersection can be done like this:
```csharp
/// <summary>
/// Calculates the area of a polygon. See:
/// http://mathworld.wolfram.com/PolygonArea.html
/// A GpcWrapper polygon is made up of one or more
/// contours.
/// </summary>
internal static double CalculateArea(
  this Polygon polygon )
{
  double area = 0.0;
  for( int i = 0; i < polygon.NofContours; ++i )
  {
    if( polygon.ContourIsHole[i] )
    {
      area -= Math.Abs(
        polygon.Contour[i].CalculateArea() );
    }
    else
    {
      area += Math.Abs(
        polygon.Contour[i].CalculateArea() );
    }
  }
  return area;
}

/// <summary>
/// Calculates the area of a polygon. See:
/// http://mathworld.wolfram.com/PolygonArea.html
/// A GpcWrapper contour is what would generally
/// be called a polygon.
/// </summary>
internal static double CalculateArea(
  this VertexList contour )
{
  double accumulator = 0.0;
  for( int i = 0; i < contour.NofVertices; i++ )
  {
    double x1 = contour.Vertex[i].X;

    double x2 = contour.Vertex[( i + 1 )
      % contour.NofVertices].X; // wrap around

    double y1 = contour.Vertex[i].Y;

    double y2 = contour.Vertex[( i + 1 )
      % contour.NofVertices].Y; // wrap around

    accumulator += x1 \* y2;
    accumulator -= x2 \* y1;
  }
  return accumulator / 2;
}
```

This allows me to perform operations like the ones required in the pseudocode listed above, such as 'belowIntersection.CalculateArea()' – it is especially this functionality that I believe will be rather tricky to implement in the pure Revit API.

#### Holidays

I am going on vacation to Andalusia tomorrow for the next ten days, so I will probably not be posting any more until March.
I wish you all a wonderful time and hope that you will have many pleasant, exciting and illuminating Revit API adventures while I take some time off from them.