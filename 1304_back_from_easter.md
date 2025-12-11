---
ai_optimized: true
author: Jeremy Tammik
complexity_score: 4.6
content_type: qa
optimization_date: '2025-12-11T11:44:15.726206'
original_url: https://thebuildingcoder.typepad.com/blog/1304_back_from_easter.html
post_number: '1304'
reading_time_minutes: 7
series: general
slug: back_from_easter
source_file: 1304_back_from_easter.htm
tags:
- csharp
- doors
- elements
- filtering
- geometry
- levels
- references
- revit-api
- rooms
- walls
- windows
title: Back from Easter Holidays and Various Revit API Issues
word_count: 1485
---

### Back from Easter Holidays and Various Revit API Issues

I hope you had a wonderful time over Easter.

I returned from my
[team meeting](http://thebuildingcoder.typepad.com/blog/2015/04/revit-api-trends-and-team-meeting-in-bretagne.html#2) and
subsequent brief holiday in the snow and am confronted with a long list of unanswered
[Revit API discussion forum](http://forums.autodesk.com/t5/revit-api/bd-p/160) issues.

I answered a dozen and present a summary of a few of the most interesting ones:

- [Determining the distance between floor and bottom of a beam](#2)
- [Consecutive ordering of TopographySurface boundary points](#3)
- [Extracting unique building element geometry vertices](#4)
- [Retrieving all visible elements](#5)

Before I get to them, two pictures relating to the past weekend.

First, a toast in the snow with the [Säntis Mountain](https://en.wikipedia.org/wiki/S%C3%A4ntis) in the background:

![A toast with Falk and Saentis in the background](file:////j/photo/jeremy/2015/2015-04-05_gamserrugg/jeremy_falk_auf_gamserrugg_mit_saentis.jpeg)

Secondly, a snapshot of a brief and slightly sad conversation shortly after Easter:

![Two chokolate Easter bunnies](img/bitten_schoki_easter_bunny.jpeg)

Getting back to the Revit API, here are some of the recent issues raised:

#### Determining the Distance Between Floor and Bottom of a Beam

Raised by David in
[how can get of Beam element 'Level' and 'Upper Level'](http://forums.autodesk.com/t5/revit-api/help-how-can-get-of-a-beam-element-their-quot-level-quot-and/m-p/5567028M9357):

**Question:**
I am trying to get the ELEVATION (double value) the Beam element.
I'm going to subtract values the Upper Elevation and Elevation the element for getting the height of the void space.
For example, height = Level 4 - Level 0 = 4 meters:

![Beam distance to floor](img/beam_distance_to_floor.png)

**Answer:**
You can use the
[ReferenceIntersector](http://thebuildingcoder.typepad.com/blog/2014/09/a-couple-of-recent-issues.html#4) ray
tracing functionality to achieve this.

In fact, the SDK sample MeasureHeight does almost exactly what you are asking for.

It was introduced in the
[Revit 2011 API](http://thebuildingcoder.typepad.com/blog/2010/06/devcamp-session-on-whats-new.html) and
originally provided via the
[FindReferencesByDirection method](http://thebuildingcoder.typepad.com/blog/2010/01/findreferencesbydirection.html).

The example later published as the MeasureHeight SDK sample is described in
[FindReferencesByDirection](http://thebuildingcoder.typepad.com/blog/2010/01/findreferencesbydirection.html) >
Measure distances > Example: Measure Distance with FindReferencesByDirection.

#### Consecutive Ordering of TopographySurface Boundary Points

Raised by Klanea in
[does TopographySurface.GetBoundaryPoints guarantee point ordering?](http://forums.autodesk.com/t5/revit-api/does-topographysurface-getboundarypoints-guarantee-point/m-p/5565632)

**Question:**
I'd like to extract consecutive boundary points of a TopographySurface, i.e., by traversing the returned List[XYZ].

I'm simulating a continuous, vertex-by-vertex walk around the TopographySurface perimeter.

I suspect all of this information must live in the underlying data structure, but I can't find out how to get at it, and the documentation of the
[TopographySurface.GetBoundaryPoints method](http://revitapisearch.com/html/f34dbefb-94de-43e8-967d-8662f6593dac.htm) makes no mention of ordering.

I've also tried probing the Vertices member of the underlying Mesh (with a call to Geometry.GetEnumator()), but this returns boundary and interior points, also in an unknown order.

Anyone know how best to extract this information directly from the API, or else have insight into the order guaranteed by a call to TopographySurface.GetBoundaryPoint()?

**Answer:**
In the far distant past, you could not even ask the topography surface which points were interior or exterior.

Therefore, I implemented a
[topo surface interior and boundary points](http://thebuildingcoder.typepad.com/blog/2011/03/toposurface-interior-and-boundary-points.html) add-in
to determine this.

The interior versus exterior predicate was later added to the API, cf.
[What's New in the Revit 2014 API](http://thebuildingcoder.typepad.com/blog/2013/04/whats-new-in-the-revit-2014-api.html) >
Reading points from a TopographySurface.

However, my old add-in should help provide what you need as well.

#### Extracting Unique Building Element Geometry Vertices

Raised by abhijitdesale in
[extracting coordinates from geometry of room or any other structure](http://forums.autodesk.com/t5/revit-api/extracting-co-ordinates-from-geometry-of-room-or-any-other/td-p/5565050):

**Question:**
I am a newbie in using C# and Revit API programming for creating plugins.

I am trying to retrieve co-ordinates of any structure. Let's say, for a room with four walls, I need to retrieve the co-ordinates of the four corners and write them to a text file.

I have used a wall filter to extract the elements and then applied the get\_Geometry method on each element.

This approach gives me a lot of duplicate co-ordinates.

I have tried the same way for using different filters for roofs, floors, doors and windows.

I don't know for sure whether I am correct or not.

Please let me know if this is a correct way of extracting the co-ordinates of any structure.

If there is a better and more correct method to do it, please let me know.

**Answer:**
Your approach sounds fine to me.

Duplicate coordinates are to be expected, of course.

You can eliminate them in many ways.

The easiest, I find, is to implement a fuzzy point equality comparer and then add the points as keys to a dictionary.

I think I do so in my
[concrete setout point add-in](http://thebuildingcoder.typepad.com/blog/2014/11/concrete-setout-points-for-revit-structure-2015.html).

I also demonstrate this approach in many other samples on The Building Coder.

Search for `XyzEqualityComparer`, for instance.

I implemented the original version of it that back in 2009 to analyse
[nested instance geometry](http://thebuildingcoder.typepad.com/blog/2009/05/nested-instance-geometry.html).

#### Retrieving All Visible Elements

Raised by Sunil in
[getting all elements with HasMaterialQuantities condition](http://forums.autodesk.com/t5/revit-api/getting-all-elements-with-hasmaterialquantities-condition/m-p/5560261):

**Question:**
To retrieve all visible elements, we referred to the previous discussion here on
[retrieving all elements](http://forums.autodesk.com/t5/revit-api/retrieving-all-elements/td-p/3377715).

It suggests using the HasMaterialQuantities method.

But Category::HasMaterialQuantities is returning false for topography, railings, and similar elements.

So they are missing when we try to get all the Revit model elements.

Here is the snippet of managed C++ .NET code that we are trying to use:

```
  FilteredElementCollector^ allInstances
    = gcnew FilteredElementCollector( revitDocument );

  allInstances = allInstances->WhereElementIsNotElementType();

  for each( Element^ element in allInstances )
  {
    if( element != nullptr && element->Category != nullptr
      && !element->Category->Name->Contains(“Legend Components”)
      && element->Category->HasMaterialQuantities )
    {
      GeometryElement^ geomElem = element->Geometry::get(geomOption);
      if( geomElem != nullptr && geomElem->GetEnumerator()->MoveNext() )
      {
        // if this geom element genuinely contains geometry
        // put element in list
      }
    }
  }
```

Any idea why HasMaterialQuantities is skipping visible elements like topography, railings etc.?

If we remove the HasMaterialQuantities condition, then we are getting the topography, railings along with all other elements, but we also get extra elements such as Rooms, which are not visible in the Revit project.

So what should be the right approach so that we get all the visible elements without skipping any valid visible elements like topography, railings etc.?

**Answer:**
I am afraid that you are encountering a basic and incontestable aspect of BIM: it is complex.

It may be hard or impossible to find a simple one-line criteria that satisfies your exact needs.

The Building Coder topic group on
[Filtering for all Elements](http://thebuildingcoder.typepad.com/blog/about-the-author.html#5.9) discusses
this exact issue.

You will probably have to implement some more complex filtering criteria, and possibly handle different groups of elements in different ways.

Here are two examples of simple first stabs at retrieving MEP and structural elements demonstrating such combinations:

- [Retrieving MEP elements and connectors](http://thebuildingcoder.typepad.com/blog/2010/06/retrieve-mep-elements-and-connectors.html)
- [Retrieving structural elements](http://thebuildingcoder.typepad.com/blog/2010/07/retrieve-structural-elements.html)

You will probably have to implement a similar kind of combined filtering algorithm for your needs.

Many more filtering samples are assembled in
[The Building Coder samples](https://github.com/jeremytammik/the_building_coder_samples)
[CmdCollectorPerformance module](https://github.com/jeremytammik/the_building_coder_samples/blob/master/BuildingCoder/BuildingCoder/CmdCollectorPerformance.cs).

Also, I am sure that you understand that the next developer's needs will differ ever so slightly from yours, so there cannot be a simple solution for this challenge.

Please let us know how you end up solving this!

**Response:**
I am now using a
[Custom Exporter](http://thebuildingcoder.typepad.com/blog/about-the-author.html#5.1) to
retrieve all elements and it is serving the purpose.

I followed the samples that you have put on Building Coder.

I agree that special filtering logic will be needed for specific elements and there is no one generic logic that can be used.