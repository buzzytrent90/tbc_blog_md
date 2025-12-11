---
ai_optimized: true
author: Jeremy Tammik
complexity_score: 2.4
content_type: qa
optimization_date: '2025-12-11T11:44:13.334547'
original_url: https://thebuildingcoder.typepad.com/blog/0084_room_wall_adjacent_area.html
post_number: 0084
reading_time_minutes: 4
series: general
slug: room_wall_adjacent_area
source_file: 0084_room_wall_adjacent_area.htm
tags:
- csharp
- elements
- geometry
- revit-api
- rooms
- views
- walls
title: Room and Wall Adjacent Area
word_count: 828
---

### Room and Wall Adjacent Area

I continued the discussion on the
[room and wall adjacency](http://thebuildingcoder.typepad.com/blog/2009/01/room-and-wall-adjacency.html)
with Massimiliano Revelli of
[Informatica System srl](http://www.infosys.it),
because he is actually interested in determining the wall area adjacent to the room, and not just the overlapping curve segment length.

The overlapping area is obviously easy to determine if the wall has a constant height, as I pointed out in the earlier post, but this is often not the case.
For instance, let us look at the upper wall in this floor plan of a simple example demonstrating the difficulties, highlighted in red:

![Room Wall Adjacency Floor Plan](img/adjacency_floor_plan.png)

The 3D view of the highlighted wall looks like this:

![Room Wall Adjacency 3D View](img/adjacency_3d_view.png)

The wall geometry and the solid it provides is not subdivided at the point where it intersects the interior wall, as we can see from the triangles displayed by the element viewer SDK sample:

![Wall displayed in ElementViewer](img/adjacency_wall_in_ElementViewer.png)

To determine the exact adjacent area, we need to determine the inner face of the wall and determine which part of it overlaps the room boundary segments.
We can make use of the solutions on how to determine the
[inner face of the wall](http://thebuildingcoder.typepad.com/blog/2008/11/wall-elevation-profile.html)
and its
[area](http://thebuildingcoder.typepad.com/blog/2008/12/3d-polygon-areas.html) from previous discussions.

Here are three different suggestions on how to approach the problem:

- Implement an algorithm that calculates the area of the portion of a vertical triangle that overlaps a horizontal line segment.
- When determining the wall polygons, truncate them to return only those parts overlapping the horizontal room boundary segments.
- Use the room's ClosedShell property and calculate its intersection area with the wall faces.

Here are slightly more detailed descriptions of each approach:

One approach to solve our current problem would be to implement an algorithm that projects the room boundary curve segments onto the wall face triangles and determines the area of the overlap.

Another approach might start a little bit earlier, modifying the utility methods in the module CmdWallProfile.cs, in the method GetWallProfilePolygons, for instance by modifying the GetProfile method to project its polygons onto the wall location curve, and then chop of the parts that lie outside the room boundary curve segment.
The advantage of this is that our polygon area calculation algorithm could be left unchanged, since it already handles all kinds of polygons, and is not limited to triangles.

A third, completely different approach could make use of the Room object's ClosedShell property. ClosedShell returns a geometry element, which is a solid, which in this specific sample model is a quadrilateral with six faces and twelve edges. We can calculate the overlap of the closed shell faces with the wall faces, which will give us the adjacent surface.

The good thing about the latter approach is that part of the required analysis has already been implemented, in the SDK sample RoofsRooms.
It includes a module GeomUtil.cs which implements a method AreSolidsCut.
This method analyses whether any one room solid face is a subset of any roof solid face, in which case it returns true.
In the RoofsRooms sample, this method is used to determine which roof element is covering which room.
In our case, we would need to expand this method to not just return true if an overlap is found, but actually calculate the overlapping area.
For the moment, this is left as an exercise to the reader.
Submissions of working solutions are gladly accepted.

To implement a generic solution for calculating the area of two overlapping polygons, we need a 2D Boolean operation to calculate the intersection of two polygons.
Unfortunately, the Revit API does not provide such a Boolean operation out of the box.

One might possibly be able to use the some of the functionality provided by various AutoCAD APIs such as AModeler or regions, by
[running AutoCAD in the background](http://thebuildingcoder.typepad.com/blog/2008/11/running-autocad-inside-revit.html)
and calling a helper application.
Of course, this requires AutoCAD to be installed on the same machine as your Revit application, which is not always desirable.

Another and possibly simpler option might be to make use of a free-standing Boolean operation library for polygons. There are a number of them out there, as you can see by googling for "polygon boolean" or by looking at the Wikipedia article on
[Boolean operations on polygons](http://en.wikipedia.org/wiki/Boolean_operations_on_polygons).
I have downloaded the
[Generic Polygon Clipper (gpc)](http://www.cs.man.ac.uk/~toby/alan/software)
as well as the C# wrapper GpcWrapper for it, and that looks promising.
Please let me know if you already have any experience making use of this or any other Boolean libraries within Revit.