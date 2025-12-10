---
post_number: "0810"
title: "Room in Area Predicate via Point in Polygon Test"
slug: "room_in_area"
author: "Jeremy Tammik"
tags: ['csharp', 'elements', 'filtering', 'parameters', 'revit-api', 'rooms', 'windows']
source_file: "0810_room_in_area.htm"
original_url: "https://thebuildingcoder.typepad.com/blog/0810_room_in_area.html"
---

### Room in Area Predicate via Point in Polygon Test

I presented a sample last week showing how to
[access area boundary loops](http://thebuildingcoder.typepad.com/blog/2012/08/graphically-display-area-boundary-loops.html) and
display the result graphically by creating corresponding model curves.

Jeroen van Vlierden of
[ICN Solutions BV](http://www.icn-solutions.nl) took
this idea one step further by combining the area boundary loop extraction with my
[point in polygon containment algorithm](http://thebuildingcoder.typepad.com/blog/2010/12/point-in-polygon-containment-algorithm.html) to
find out whether a given room resides within an area.

He also pointed out that my area boundary sample attempts to use the Area class directly in a filtered element collector, which is not allowed, as I explained when
[accessing room data](http://thebuildingcoder.typepad.com/blog/2011/11/accessing-room-data.html).

Jeroen explains:

I can now implement the point in polygon test.

Thank you very much for solving my problem.
This will help me determine which rooms are part of which area and therefore also which windows belong to which area.
It is part of a program in which we want to determine how many square meters glass is being used in user specified areas.

I implemented your point in polygon test as an extension method of the area element type.
C# is such an elegant language ;-)

I use extension methods a lot to extend the Revit API with my own methods, especially when it comes to dealing with various kinds of parameters etc.

After using them for a while, I will not even remember they are not part of the official API.

The only thing that will not work correctly (yet) is when there is an arc part of the polygon.

One easy way to deal approximately with the arc (and all other non-linear segments as well) would be to make use of the Revit API geometric tessellation functionality, reducing everything to (approximate) straight line segments.
This is extremely simple, since all curve elements implement the
[Tessellate method](http://thebuildingcoder.typepad.com/blog/2010/01/curves.html) returning
a list of XYZ approximation points.
Its use is demonstrated in numerous places in the SDK samples.

I reused your code based on UV points by writing an extension method to convert XYZ to UV and I have added a constructor to the PointInPoly class that allows a List of XYZ points to define the PointInPoly class.

Here is my version of the point in polygon implementation
[PointInPoly.cs](zip/room_in_area_PointInPoly.cs.txt).

Here is the entire code of the various extension methods defined in the module
[PointInPolyExtensions.cs](zip/room_in_area_PointInPolyExtensions.cs.txt):
```csharp
public static class PointInPolyExtensions
{
  /// <summary>
  /// Add new point to list, unless already present.
  /// </summary>
  private static void AddToPunten(
    List<XYZ> XYZarray,
    XYZ p1 )
  {
    var p = XYZarray.Where(
      c => Math.Abs( c.X - p1.X ) < 0.001
        && Math.Abs( c.Y - p1.Y ) < 0.001 )
      .FirstOrDefault();

    if( p == null )
    {
      XYZarray.Add( p1 );
    }
  }

  /// <summary>
  /// Return a list of boundary
  /// points for the given room.
  /// </summary>
  private static List<XYZ> MaakPuntArray(
    Room room )
  {
    SpatialElementBoundaryOptions opt
      = new SpatialElementBoundaryOptions();

    opt.SpatialElementBoundaryLocation
      = SpatialElementBoundaryLocation.Center;

    var boundaries = room.GetBoundarySegments(
      opt );

    return MaakPuntArray( boundaries );
  }

  /// <summary>
  /// Return a list of boundary points
  /// for the given boundary segments.
  /// </summary>
  private static List<XYZ> MaakPuntArray(
    IList<IList<BoundarySegment>> boundaries )
  {
    List<XYZ> puntArray = new List<XYZ>();
    foreach( var bl in boundaries )
    {
      foreach( var s in bl )
      {
        Curve c = s.Curve;
        AddToPunten( puntArray, c.get\_EndPoint( 0 ) );
        AddToPunten( puntArray, c.get\_EndPoint( 1 ) );
      }
    }
    puntArray.Add( puntArray.First() );
    return puntArray;
  }

  /// <summary>
  /// Return a list of boundary
  /// points for the given area.
  /// </summary>
  private static List<XYZ> MaakPuntArray(
    Area area )
  {
    SpatialElementBoundaryOptions opt
      = new SpatialElementBoundaryOptions();

    opt.SpatialElementBoundaryLocation
      = SpatialElementBoundaryLocation.Center;

    var boundaries = area.GetBoundarySegments(
      opt );

    return MaakPuntArray( boundaries );
  }

  /// <summary>
  /// Check whether this area contains a given point.
  /// </summary>
  public static bool AreaContains( this Area a, XYZ p1 )
  {
    bool ret = false;
    var p = MaakPuntArray( a );
    PointInPoly pp = new PointInPoly();
    ret = pp.PolyGonContains( p, p1 );
    return ret;
  }

  /// <summary>
  /// Check whether this room contains a given point.
  /// </summary>
  public static bool RoomContains( this Room r, XYZ p1 )
  {
    bool ret = false;
    var p = MaakPuntArray( r );
    PointInPoly pp = new PointInPoly();
    ret = pp.PolyGonContains( p, p1 );
    return ret;
  }

  /// <summary>
  /// Project an XYZ point to a UV one in the
  /// XY plane by simply dropping the Z coordinate.
  /// </summary>
  public static UV TOUV( this XYZ point )
  {
    UV ret = new UV( point.X, point.Y );
    return ret;
  }
}
```

Many thanks to Jeroen for this discussion and nice example!

For completeness' sake and your convenience, here is an updated version of the
[DisplayBoundary sample](zip/DisplayBoundary2.zip) including

- The fix to filter for spatial elements instead of areas- The point in polygon algorithm- Jeroen's extension methods

Enjoy!