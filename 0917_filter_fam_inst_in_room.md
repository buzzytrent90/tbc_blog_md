---
ai_optimized: true
author: Jeremy Tammik
complexity_score: 2.7
content_type: documentation
optimization_date: '2025-12-11T11:44:14.886650'
original_url: https://thebuildingcoder.typepad.com/blog/0917_filter_fam_inst_in_room.html
post_number: 0917
reading_time_minutes: 4
series: filtering
slug: filter_fam_inst_in_room
source_file: 0917_filter_fam_inst_in_room.htm
tags:
- csharp
- elements
- family
- filtering
- geometry
- parameters
- revit-api
- rooms
- schedules
- views
title: Filter for Family Instances in a Room
word_count: 756
---

### Filter for Family Instances in a Room

I wish you a
[Good Friday](http://en.wikipedia.org/wiki/Good_Friday).

![El Greco's Jesus Carrying the Cross, 1580](img/Christ_Carrying_the_Cross_1580.jpg)

Continuing the research and development for my
[cloud-based round-trip 2D Revit model editing project](http://thebuildingcoder.typepad.com/blog/2013/03/cloud-mobile-extensible-storage-data-use-in-schedules.html#3),
I need to determine the boundary loop polygons to represent the furniture and equipment family instances for manipulation on the mobile device.

Before I can start extracting their geometry, I need to access the objects themselves, i.e. determine which family instances are contained in the selected room.

As always, we use a filtered element collector to access the Revit database elements.

I try to apply as many quick filters as possible first, and then potentially refine the results adding slow filters and possibly even additional tests in .NET using LINQ or explicit coding.

In this case, I came up with the following sequence of filter tests which seems to fit my needs:

- Not ElementType
- View independent
- Family instance
- Bounding box intersects outline of the room's bounding box

The first two may or may not be superfluous, actually, but they do no harm either.

For the last test, I query the room bounding box, convert it to an Outline, and use that to set up a BoundingBoxIntersectsFilter.
It might be possible to implement something stricter than this, or use a BoundingBoxContainsPointFilter instead.

Currently, the category and other properties are not checked at all.
Once I know more exactly what kind of family instances I need, I would add those.

All of these are quick filters.

Since the room bounding box may be much larger than the room itself, for instance for an irregularly shaped or long, narrow, diagonal one, I definitely need to post-process the results with a more precise containment test.
It could be purely geometric, e.g. using the
[point in polygon containment algorithm](http://thebuildingcoder.typepad.com/blog/2010/12/point-in-polygon-containment-algorithm.html) as
in the
[room in area predicate](http://thebuildingcoder.typepad.com/blog/2012/08/room-in-area-predicate-via-point-in-polygon-test.html),
or based on other data.

I chose to implement this by checking the family instance Room property.
This test could probably also be moved into a filter, e.g. a parameter filter, which would significantly improve its efficiency by avoiding marshaling the data from internal Revit to the external .NET space before checking and potentially rejecting it.
See below for further considerations on using this property.

The method currently looks like this:

```csharp
/// <summary>
/// Return the element ids of all furniture and
/// equipment family instances contained in the
/// given room.
/// </summary>
List<Element> GetFurniture( Room room )
{
  BoundingBoxXYZ bb = room.get\_BoundingBox( null );

  Outline outline = new Outline( bb.Min, bb.Max );

  BoundingBoxIntersectsFilter filter
    = new BoundingBoxIntersectsFilter( outline );

  Document doc = room.Document;

  // Todo: add category filters and other
  // properties to narrow down the results

  FilteredElementCollector collector
    = new FilteredElementCollector( doc )
      .WhereElementIsNotElementType()
      .WhereElementIsViewIndependent()
      .OfClass( typeof( FamilyInstance ) )
      .WherePasses( filter );

  int roomid = room.Id.IntegerValue;

  List<Element> a = new List<Element>();

  foreach( FamilyInstance fi in collector )
  {
    if( fi.Room.Id.IntegerValue.Equals( roomid ) )
    {
      a.Add( fi );
    }
  }
  return a;
}
```

#### Family Instance Room Property Considerations

For the final test whether the family instance lies inside the selected room, I simply use the FamilyInstance.Room property.

I am aware that there may be some
[issues with the Room property](http://thebuildingcoder.typepad.com/blog/2011/03/unreliable-room-properties.html) and
[its phase dependency](http://thebuildingcoder.typepad.com/blog/2011/02/phase-dependent-room-properties.html),
and Patrick Rosendahl implemented a more reliable
[DetermineRoom method](http://thebuildingcoder.typepad.com/blog/2012/02/family-instance-room-phase.html) to
find a containing room for a given element under all circumstances.
In a complex model, I might have to add such enhancements to my approach.

#### Considerations Processing a Complete Model

Another issue is that the filter I present above searches for the family instances on a room by room basis.

This is fine as long as I am just looking at one single room.

If I wished to process all rooms of an entire large model, I would definitely approach this completely differently.
For instance, I might select all family instances of interest in the entire model, determine the containing room for each one of them, and then
[invert that relationship](http://thebuildingcoder.typepad.com/blog/2008/10/relationship-in.html) to
obtain a list of instances for each room.