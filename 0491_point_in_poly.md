---
post_number: "0491"
title: "Point in Polygon Containment Algorithm"
slug: "point_in_poly"
author: "Jeremy Tammik"
tags: ['csharp', 'elements', 'family', 'geometry', 'python', 'references', 'revit-api', 'rooms', 'views']
source_file: "0491_point_in_poly.htm"
original_url: "https://thebuildingcoder.typepad.com/blog/0491_point_in_poly.html"
---

### Point in Polygon Containment Algorithm

I have been travelling for two nights and a day now and arrived in Tel Aviv this morning, via snowy London yesterday, ready to prepare for the next conference.

I just had a wonderful cappuccino with an almond croissant and some chocolate truffles for breakfast, a swim in the Mediterranean, and a run along the beach.
Now I am ready to post again :-)

One thing that cropped up repeatedly at Autodesk University was the question of determining point containment and 2D polygon intersections independently of Revit.
In fact, the request is not independent of Revit, but...
Some of the geometrical analysis methods provided by the Revit API are not as generic as they might be.

- It provides methods to determine whether a given point is located inside a room or space, but not within an area.- Many of the interesting geometrical methods like Intersect and FindReferencesByDirection do not work on geometry located in linked files.- It lacks a generic 2D polygon intersection method.

Besides, I have this unhealthy tendency to always strive for exaggerated independence.
It doesn't always make life easier, but it certainly does make it more interesting at times.
It also causes occasional fuss and worry.

Anyway, determining whether a 2D point is located within a given 2D polygon is actually very simple, and this is an important instrument to have available in every CAD programmer's toolbox, so why depend on the Revit API implementation when you can roll it yourself?
In conjunction with the Revit independent
[Boolean operations on 2D polygons](http://thebuildingcoder.typepad.com/blog/2009/02/boolean-operations-for-2d-polygons.html),
a lot of powerful analysis can be performed on pure geometry both within Revit and exported into some other external container such as a database.

Actually, one discussion we had dealt with a number of linked files representing the architectural, furniture, MEP, and medical equipment in large hospital projects.
The relevant geometrical properties of these elements can be systematically encoded in such a way that they can be used as keys in an external database, e.g. using encodings produced similarly to my
[XYZ Compare method](http://thebuildingcoder.typepad.com/blog/2009/05/nested-instance-geometry.html) and
[XyzEqualityComparer class](http://thebuildingcoder.typepad.com/blog/2009/05/nested-instance-geometry.html).

This can enable us to manage the equipment and other Revit family instances and maintain their relationship with the appropriate rooms, storing their locations, line segments, and boundary polygons in the database so that they can also be used as keys for identification, searching and sorting.

Here is a typical variant of the point containment question:

**Question:** How can I find out whether a particular point lies inside a given area element?

I need to retrieve a list of rooms bounded by an area, and figure that the only way to do so is to see if all the vertices of the room in the plan are inside the shape made by the area element.

You could make this more of a generic mathematical question – in a 2D environment, how do I know when a point in space is inside a given polygon?

I attempted to work this out using FindReferencesByDirection.
It allows me to cast a ray out and look at intersections, but it only works in a 3D view.
An Area Plan is a 2D view.

How would I cast a ray from a Room origin point and analyze intersections with a containing Area?

**Answer:** I am sure that you are aware of the fact that you can query whether a given point lies within a room or space. Here is a snippet of the What's New section in the Revit 2011 API help file covering this:

#### Rooms and Spaces

The new method Autodesk.Revit.Elements.Room.IsPointInRoom determines if a 3D point is in the volume of a room.

The new method Autodesk.Revit.Elements.Space.IsPointInSpace determines if a 3D point is in the volume of a space.

The new methods Autodesk.Revit.Document. GetRoomAtPoint and GetSpaceAtPoint find the Room or Space that encloses a given 3D point within the project.

The new property Autodesk.Revit.Elements.FamilyInstance.Space allows an API application to determine what in which Space an air terminal or other family instance exists in Revit MEP.

However, you wish to determine which area a given room resides in, so this does not address your need.

As you say, for abstract 2D geometry, this is a pretty standard basic question. I did once implement an algorithm for this, years ago, in C++.
Funnily enough, the algorithm is attributed to Kevin Weiler of Autodesk, although I got it from one of the Graphic Gems books.
My implementation looks like this.
To see the truncated lines in full, please copy and paste to an editor:
```python
//
// ptinpoly.cpp - check whether point is inside polygon
//
// Copyright (C) 1996 by Jeremy Tammik. All rights reserved.
//
// Based on C code from the article
// "An Incremental Angle Point in Polygon Test"
// by Kevin Weiler, kjw@autodesk.com
// in "Graphics Gems IV", Academic Press, 1994
//
// 97-11-28:
//
// Added code to handle (hopefully) boundaries including
// circular arc segments. That is based on code from
// the article "Point in polygon strategies" by Eric Haines
// also in "Graphics Gems IV", Academic Press, 1994.
//

typedef int quadrant\_type;

//
// determine the quadrant of a polygon point relative to the test point:
//
inline quadrant\_type quadrant( const RoPoint2d & vertex, const RoPoint2d & p )
{
  return (vertex.x > p.x)
    ? ((vertex.y > p.y) ? 0 : 3)
    : ( (vertex.y > p.y) ? 1 : 2);
}

//
// determine x intercept of a polygon edge
// with a horizontal line at the y value of the test point:
//
inline double x\_intercept( const RoPoint2d & pt1, const RoPoint2d & pt2, double y )
{
  //roassert( 0 != (pt1.y - pt2.y) );
  return pt2.x - ( (pt2.y - y) \* ((pt1.x - pt2.x) / (pt1.y - pt2.y)) );
}

static void adjust\_delta(
  int & delta,
  const RoPoint2d & vertex,
  const RoPoint2d & next\_vertex,
  const RoPoint2d & p )
{
  switch( delta ) {
      /\* make quadrant deltas wrap around \*/
    case  3:  delta = -1; break;
    case -3:  delta =  1; break;
      /\* check if went around point cw or ccw \*/
    case  2:
    case -2:
      if (x\_intercept(vertex, next\_vertex, p.y) > p.x)
        delta =  - (delta);
      break;
  }
}

bool polygonContains(
  int nPoints,
  const RoPoint2d \* polygon,
  const RoPoint2d & p )
{
  quadrant\_type quad, next\_quad, delta, angle;

      /\* initialize \*/
  const RoPoint2d & vertex = \*polygon;
  quad = quadrant( \*polygon, p );
  angle = 0;
      /\* loop on all vertices of polygon \*/
  for( int i = 0; i < nPoints; ++i ) {
    const RoPoint2d & vertex = polygon[i];
    const RoPoint2d & next\_vertex = polygon[ (i+1 < nPoints) ? i+1 : 0];
    /\* calculate quadrant and delta from last quadrant \*/
    next\_quad = quadrant( next\_vertex, p );
    delta = next\_quad - quad;
    adjust\_delta( delta, vertex, next\_vertex, p );
    /\* add delta to total angle sum \*/
    angle = angle + delta;
    /\* increment for next step \*/
    quad = next\_quad;
  }
      /\* complete 360 degrees (angle of + 4 or -4 ) means inside \*/
  return (angle == +4) || (angle == -4);
      /\* odd number of windings rule \*/
  /\* if (angle & 4) return INSIDE; else return OUTSIDE; \*/
      /\* non-zero winding rule \*/
  /\* if (angle != 0) return INSIDE; else return OUTSIDE; \*/
}
```

This code does not show the definition of the 2D point class, but I think all it needs is a simple container for the X and Y coordinates.

For your convenience and out of personal interest, I ported this code to C#, .NET and the Revit UV class.
The result looks like this:
```python
public class PointInPoly
{
  /// <summary>
  /// Determine the quadrant of a polygon vertex
  /// relative to the test point.
  /// </summary>
  Quadrant GetQuadrant( UV vertex, UV p )
  {
    return ( vertex.U > p.U )
      ? ( ( vertex.V > p.V ) ? 0 : 3 )
      : ( ( vertex.V > p.V ) ? 1 : 2 );
  }

  /// <summary>
  /// Determine the X intercept of a polygon edge
  /// with a horizontal line at the Y value of the
  /// test point.
  /// </summary>
  double X\_intercept( UV p, UV q, double y )
  {
    Debug.Assert( 0 != ( p.V - q.V ),
      "unexpected horizontal segment" );

    return q.U
      - ( ( q.V - y )
        \* ( ( p.U - q.U ) / ( p.V - q.V ) ) );
  }

  void AdjustDelta(
    ref int delta,
    UV vertex,
    UV next\_vertex,
    UV p )
  {
    switch( delta )
    {
      // make quadrant deltas wrap around:
      case 3: delta = -1; break;
      case -3: delta = 1; break;
      // check if went around point cw or ccw:
      case 2:
      case -2:
        if( X\_intercept( vertex, next\_vertex, p.V )
          > p.U )
        {
          delta = -delta;
        }
        break;
    }
  }

  /// <summary>
  /// Determine whether given 2D point lies within
  /// the polygon.
  ///
  /// Written by Jeremy Tammik, Autodesk, 2009-09-23,
  /// based on code that I wrote back in 1996 in C++,
  /// which in turn was based on C code from the
  /// article "An Incremental Angle Point in Polygon
  /// Test" by Kevin Weiler, Autodesk, in "Graphics
  /// Gems IV", Academic Press, 1994.
  ///
  /// Copyright (C) 2009 by Jeremy Tammik. All
  /// rights reserved.
  ///
  /// This code may be freely used. Please preserve
  /// this comment.
  /// </summary>
  bool PolygonContains(
    UVArray polygon,
    UV point )
  {
    // initialize
    Quadrant quad = GetQuadrant(
      polygon.get\_Item( 0 ), point );

    Quadrant angle = 0;

    // loop on all vertices of polygon
    Quadrant next\_quad, delta;
    int n = polygon.Size;
    for( int i = 0; i < n; ++i )
    {
      UV vertex = polygon.get\_Item( i );

      UV next\_vertex = polygon.get\_Item(
        ( i + 1 < n ) ? i + 1 : 0 );

      // calculate quadrant and delta from last quadrant

      next\_quad = GetQuadrant( next\_vertex, point );
      delta = next\_quad - quad;

      AdjustDelta(
        ref delta, vertex, next\_vertex, point );

      // add delta to total angle sum
      angle = angle + delta;

      // increment for next step
      quad = next\_quad;
    }

    // complete 360 degrees (angle of + 4 or -4 )
    // means inside

    return ( angle == +4 ) || ( angle == -4 );

    // odd number of windings rule:
    // if (angle & 4) return INSIDE; else return OUTSIDE;
    // non-zero winding rule:
    // if (angle != 0) return INSIDE; else return OUTSIDE;
  }
}
```

This compiles all right, but I have not tested it yet. I would appreciate if you would set up a sample project, test it, and let me know whether it satisfies your needs. You will need to implement code to extract the boundary polygon of an area, convert it to an abstract geometrical 2D polygon represented by an UVArray, and call PolygonContains with that and a point to test for containment inside the room. I already discussed the
[transformation of arrays of points between 2D and 3D](http://thebuildingcoder.typepad.com/blog/2008/12/polygon-transformation.html).

A nice little exercise based on this would be to implement a Revit command that retrieves room, space or area boundaries, feeds them into this algorithm, and verifies that it really does correctly determine the containment of a number of given points in the resulting boundary polygon, as a sort of simplistic
[unit test](http://thebuildingcoder.typepad.com/blog/2010/11/unit-testing-in-revit.html).
I would love to implement this myself.
If anybody else if faster, please be my guest.

Here is
[PointInPoly.zip](zip/PointInPoly.zip)
containing the entire Visual Studio project implementing the methods listed above.