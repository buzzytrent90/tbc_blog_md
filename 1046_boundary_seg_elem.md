---
post_number: "1046"
title: "Determining Room Boundary Segment Generating Element"
slug: "boundary_seg_elem"
author: "Jeremy Tammik"
tags: ['csharp', 'elements', 'filtering', 'geometry', 'levels', 'python', 'references', 'revit-api', 'rooms', 'selection', 'views', 'walls', 'windows']
source_file: "1046_boundary_seg_elem.htm"
original_url: "https://thebuildingcoder.typepad.com/blog/1046_boundary_seg_elem.html"
---

### Determining Room Boundary Segment Generating Element

Today we look at how to determine the neighbouring BIM element of a room boundary segment using the ReferenceIntersector to shoot a ray from the room interior into the bounding element.

There should not really be any need to implement such functionality ourselves, because the BoundarySegment class already provides an Element property that should return the BIM element generating the given boundary segment.

However, unfortunately, the BoundarySegment.Element may erroneously return null under certain specific conditions, and we present a nice little Revit API workaround that can be triggered when the Element property fails.

Here are a couple of examples of retrieving room boundaries that we discussed in the past:

- [Accessing room data](http://thebuildingcoder.typepad.com/blog/2011/11/accessing-room-data.html)
- [Graphically display area boundary loops](http://thebuildingcoder.typepad.com/blog/2012/08/graphically-display-area-boundary-loops.html)
- [Room in area predicate via point in polygon test](http://thebuildingcoder.typepad.com/blog/2012/08/room-in-area-predicate-via-point-in-polygon-test.html)
- Analysis of Room and Space 3D geometry in the [Revit 2012 API news](http://thebuildingcoder.typepad.com/blog/2013/02/whats-new-in-the-revit-2012-api.html)
- New GetBoundarySegments method in the [Revit 2013 API news](http://thebuildingcoder.typepad.com/blog/2013/03/whats-new-in-the-revit-2013-api.html)
- [Retrieving plan view room boundary polygon loops](http://thebuildingcoder.typepad.com/blog/2013/03/revit-2014-api-and-room-plan-view-boundary-polygon-loops.html#3)
- [Room neighbours](http://thebuildingcoder.typepad.com/blog/2013/09/room-neighbours.html)

We also looked at several ray shooting examples to discover neighbouring elements, e.g.

- [Identifying wall compound layers and parts](http://thebuildingcoder.typepad.com/blog/2012/01/identifying-wall-compound-layers-and-parts.html)
- The [ReferenceIntersector class](http://thebuildingcoder.typepad.com/blog/2012/03/revit-2013-and-its-api.html#3)
- [Wall footing relationship](http://thebuildingcoder.typepad.com/blog/2012/10/wall-footing-relationship-revisited.html)
- [UIView, Windows Coordinates, ReferenceIntersector and My Own Tooltip](http://thebuildingcoder.typepad.com/blog/2012/10/uiview-windows-coordinates-referenceintersector-and-my-own-tooltip.html)
- [Determining the columns supporting a beam](http://thebuildingcoder.typepad.com/blog/2013/03/supporting-columns.html#2) by
  [ray casting](http://thebuildingcoder.typepad.com/blog/2013/03/supporting-columns.html#8)
- [Space adjacency for heat load calculation](http://thebuildingcoder.typepad.com/blog/2013/07/football-and-space-adjacency-for-heat-load-calculation.html#3)
- [Room neighbours](http://thebuildingcoder.typepad.com/blog/2013/09/room-neighbours.html)

Note that the recent
[room neighbour](http://thebuildingcoder.typepad.com/blog/2013/09/room-neighbours.html) discussion
belongs to both of these two groups and is actually very similar to the bounding element sample we discuss here.

The former shoots a ray from the room through the midpoint of each boundary segment to find the neighbouring room, whereas this example stops a bit shorter and determines the element in between instead.

Rudolf Honke of
[Mensch und Maschine acadGraph](http://www.acadgraph.de) raised
this issue in his interminable list of what he calls unreliable properties:

**Question:** I want to retrieve each Element alongside a room’s boundary.

However, I have an issue with all walls protruding into a room like this:

![Boundary segment lacking Element](img/GetBoundarySegmentElement.png)

The marked boundary segment has no Element associated with it, according to the BoundarySegment.Element property, which returns null for it.

I can reproduce this behaviour in several files, so I think this is the API default behaviour.

**Answer:** Let's look for a workaround.

For instance, if you look at the ordered list of segments going around the room, and the preceding and succeeding segments both belong to the same element, then you can assume that the segment in between with a null element pointer also belongs to that same element?

**Response:** No, there can be situations like this, where predecessor and successor are different elements:

![Boundary segment lacking Element](img/GetBoundarySegmentElement2.png)

**Answer:** OK, I see that your example cannot be solved looking at the predecessor and successor segments.

Next suggestion, also very simple to implement:

1. For each segment with a null element pointer:
2. Determine which direction faces into the room
3. Determine a point in the room, slightly inside, offset from the segment midpoint.
4. Shoot a ray from that point through the segment midpoint.
5. Ensure that an element is hit in the segment midpoint. If not, something is strange.
6. The element hit is the element you are looking for.

**Response:** Thank you for the hint; I already had nearly the same idea of shooting laser rays ;-)

Using your approach, I finally got it.

No need to look at the neighbouring BoundarySegments any more.

#### GetElementByRay Implementation

Rudolf provided this implementation of the suggested algorithm:

```python
  /// <summary>
  /// Return direction turning 90 degrees
  /// left from given input vector.
  /// </summary>
  public XYZ GetLeftDirection( XYZ direction )
  {
    double x = -direction.Y;
    double y = direction.X;
    double z = direction.Z;
    return new XYZ( x, y, z );
  }

  /// <summary>
  /// Return direction turning 90 degrees
  /// right from given input vector.
  /// </summary>
  public XYZ GetRightDirection( XYZ direction )
  {
    return GetLeftDirection( direction.Negate() );
  }

  /// <summary>
  /// Return the neighbouring BIM element generating
  /// the given room boundary curve c, assuming it
  /// is oriented counter-clockwise around the room
  /// if part of an interior loop, and vice versa.
  /// </summary>
  public Element GetElementByRay(
    UIApplication app,
    Document doc,
    View3D view3d,
    Curve c )
  {
    Element boundaryElement = null;

    // Tolerances

    const double minTolerance = 0.00000001;
    const double maxTolerance = 0.01;

    // Height of ray above room level:
    // ray starts from one foot above room level

    const double elevation = 1;

    // Ray starts not directly from the room border
    // but from a point offset slightly into it.

    const double stepInRoom = 0.1;

    // We could use Line.Direction if Curve c is a
    // Line, but since c also might be an Arc, we
    // calculate direction like this:

    XYZ lineDirection
      = ( c.GetEndPoint( 1 ) - c.GetEndPoint( 0 ) )
        .Normalize();

    XYZ upDir = elevation \* XYZ.BasisZ;

    // Assume that the room is on the left side of
    // the room boundary curve and wall on the right.
    // This is valid for both outer and inner room
    // boundaries (outer are counter-clockwise, inner
    // are clockwise). Start point is slightly inside
    // the room, one foot above room level.

    XYZ toRoomVec = stepInRoom \* GetLeftDirection(
      lineDirection );

    XYZ pointBottomInRoom = c.Evaluate( 0.5, true )
      + toRoomVec;

    XYZ startPoint = pointBottomInRoom + upDir;

    // We are searching for walls only

    ElementFilter wallFilter
      = new ElementCategoryFilter(
        BuiltInCategory.OST\_Walls );

    ReferenceIntersector intersector
      = new ReferenceIntersector( wallFilter,
        FindReferenceTarget.Element, view3d );

    // We don't want to find elements in linked files

    intersector.FindReferencesInRevitLinks = false;

    XYZ toWallDir = GetRightDirection(
      lineDirection );

    ReferenceWithContext context = intersector
      .FindNearest( startPoint, toWallDir );

    Reference closestReference = null;

    if( context != null )
    {
      if( ( context.Proximity > minTolerance )
        && ( context.Proximity < maxTolerance
          + stepInRoom ) )
      {
        closestReference = context.GetReference();

        if( closestReference != null )
        {
          boundaryElement = doc.GetElement(
            closestReference );
        }
      }
    }
    return boundaryElement;
  }
```

#### GetBoundarySegmentElement External Command Implementation

I implemented an add-in named GetBoundarySegmentElement and added the following read-only external command implementation to drive it:

```python
public Result Execute(
  ExternalCommandData commandData,
  ref string message,
  ElementSet elements )
{
  UIApplication uiapp = commandData.Application;
  UIDocument uidoc = uiapp.ActiveUIDocument;
  Application app = uiapp.Application;
  Document doc = uidoc.Document;
  Selection sel = uidoc.Selection;

  List<Room> rooms = new List<Room>(
    sel.Elements.Cast<Room>() );

  if( 1 != rooms.Count )
  {
    message = "Please select exactly one room.";

    return Result.Failed;
  }

  View3D view3d
    = new FilteredElementCollector( doc )
      .OfClass( typeof( View3D ) )
      .Cast<View3D>()
      .FirstOrDefault<View3D>(
        e => e.Name.Equals( "{3D}" ) );

  if( null == view3d )
  {
    message = "No 3D view named '{3D}' found.";

    return Result.Failed;
  }

  foreach( Room room in rooms )
  {
    IList<IList<BoundarySegment>> loops
      = room.GetBoundarySegments(
        new SpatialElementBoundaryOptions() );

    int n = loops.Count;

    Debug.Print(
      "Room {0} has {1} loop{2}{3}",
      room.Name, n, PluralSuffix( n ),
      DotOrColon( n ) );

    int i = 0;

    foreach( IList<BoundarySegment> loop in loops )
    {
      n = loop.Count;

      Debug.Print(
        "  Loop {0} has {1} segment{2}{3}",
        i++, n, PluralSuffix( n ),
        DotOrColon( n ) );

      int j = 0;

      foreach( BoundarySegment seg in loop )
      {
        Element e = seg.Element;

        string s = "Element property";

        if( null == e )
        {
          s = "GetElementByRay";

          e = GetElementByRay( uiapp, doc, view3d,
            seg.Curve );
        }

        Debug.Print(
          "    Segment {0}: {1} element {2} returned by {3}",
          j++, CurveString( seg.Curve ),
          ElementDescription( e ), s );
      }
    }
  }
  return Result.Succeeded;
}
```

As always, 45% of the code is input validation, 45% output reporting, and about 10% or less the real thing.
That's life.

Here is the result of running the GetBoundarySegmentElement command on the simple sample model shown above:

```
Room Test 1 has 1 loop:
  Loop 0 has 8 segments:
    Segment 0: line (-13.95,23.02,0) --> (-21.99,23.02,0)
      wall 326988 MW 24.0 WD 12.0 returned by Element property
    Segment 1: line (-21.99,23.02,0) --> (-21.99,13.04,0)
      wall 327085 MW 24.0 WD 12.0 returned by Element property
    Segment 2: line (-21.99,13.04,0) --> (-3.81,13.04,0)
      wall 327055 MW 24.0 WD 12.0 returned by Element property
    Segment 3: line (-3.81,13.04,0) --> (-3.81,23.02,0)
      wall 327018 MW 24.0 WD 12.0 returned by Element property
    Segment 4: line (-3.81,23.02,0) --> (-12.97,23.02,0)
      wall 326988 MW 24.0 WD 12.0 returned by Element property
    Segment 5: line (-12.97,23.02,0) --> (-12.97,17.44,0)
      wall 327196 STB 30.0 returned by Element property
    Segment 6: line (-12.97,17.44,0) --> (-13.95,17.44,0)
      wall 327196 STB 30.0 returned by GetElementByRay
    Segment 7: line (-13.95,17.44,0) --> (-13.95,23.02,0)
      wall 327196 STB 30.0 returned by Element property
```

Note that the BoundarySegment Element property returned null for the next to last segment 6, and the GetElementByRay method was used instead to retrieve the correct wall.

#### SpatialElementBoundaryLocation Enumeration

**Response:** When reading the blog post draft, I see:

```csharp
  foreach( Room room in rooms )
  {
    IList<IList<BoundarySegment>> loops
      = room.GetBoundarySegments(
        new SpatialElementBoundaryOptions() );
```

I would have thought that we need to tell the SpatialElementBoundaryOptions what sort of SpatialElementBoundaryLocation we want; we need Finish:

![SpatialElementBoundaryLocation enumeration](img/GetBoundarySegmentElement3.png)

Here is a suitable code snippet:

```csharp
  SpatialElementBoundaryOptions opt
    = new SpatialElementBoundaryOptions();

  opt.SpatialElementBoundaryLocation
    = SpatialElementBoundaryLocation.Finish;

  IList<IList<BoundarySegment>> loops
    = room.GetBoundarySegments( opt );
```

Of course it may be that SpatialElementBoundaryOptions are already initialized with Finish by default, but the Revit API help file RevitAPI.chm does not mention that in the object’s constructor documentation.

Otherwise, room boundaries might be inside the boundary elements, which would affect the result.

SpatialElementBoundaryLocation.Center might return something like these red lines:

![Room boundaries at SpatialElementBoundaryLocation.Center](img/GetBoundarySegmentElement4.png)

In this case, there would be no line to get the wall’s small front face from.

**Answer:** I added the suggested setting and re-ran the test, producing the following output:

```
Room Test 1 has 1 loop:
  Loop 0 has 8 segments:
    Segment 0: line (-13.95,23.02,0) --> (-21.99,23.02,0)
      wall 326988 MW 24.0 WD 12.0 returned by Element property
    Segment 1: line (-21.99,23.02,0) --> (-21.99,13.04,0)
      wall 327085 MW 24.0 WD 12.0 returned by Element property
    Segment 2: line (-21.99,13.04,0) --> (-3.81,13.04,0)
      wall 327055 MW 24.0 WD 12.0 returned by Element property
    Segment 3: line (-3.81,13.04,0) --> (-3.81,23.02,0)
      wall 327018 MW 24.0 WD 12.0 returned by Element property
    Segment 4: line (-3.81,23.02,0) --> (-12.97,23.02,0)
      wall 326988 MW 24.0 WD 12.0 returned by Element property
    Segment 5: line (-12.97,23.02,0) --> (-12.97,17.44,0)
      wall 327196 STB 30.0 returned by Element property
    Segment 6: line (-12.97,17.44,0) --> (-13.95,17.44,0)
      wall 327196 STB 30.0 returned by GetElementByRay
    Segment 7: line (-13.95,17.44,0) --> (-13.95,23.02,0)
      wall 327196 STB 30.0 returned by Element property
```

The results and the coordinates look exactly the same, so apparently the interior finish is indeed the default setting for the spatial element boundary location.

**Response:** Yes, Finish is the default value.

Just after creating a new SpatialElementBoundaryOptions object, I see the following in the Visual Studio debugger:

![SpatialElementBoundaryLocation.Finish default setting](img/GetBoundarySegmentElement5.png)

#### Download

To see the whole thing in context and try it out for yourself, here is
[GetBoundarySegmentElement.zip](zip/GetBoundarySegmentElement.zip) containing
the full source code, Visual Studio solution and add-in manifest for the GetBoundarySegmentElement external command.

Many thanks to Rudolf for pointing out this flaw in the BoundarySegment.Element property, implementing and testing the workaround, and the fruitful discussion!