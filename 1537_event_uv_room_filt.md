---
post_number: "1537"
title: "Event Uv Room Filt"
slug: "event_uv_room_filt"
author: "Jeremy Tammik"
tags: ['elements', 'filtering', 'geometry', 'levels', 'revit-api', 'rooms', 'sheets', 'views']
source_file: "1537_event_uv_room_filt.md"
original_url: "https://thebuildingcoder.typepad.com/blog/1537_event_uv_room_filt.html"
---

### Events, UV Coordinates and Rooms on Level
A lot of interesting solutions were shared in
the [Revit API discussion forum](http://forums.autodesk.com/t5/revit-api-forum/bd-p/160) and
private email messages during my absence last week, and several exciting events are looming:
- [Forge Accelerator in Gothenburg](#2)
- [AEC Hackathon in Munich](#3)
- [Autodesk University in London](#4)
- [Retrieve and map texture UV coordinates exporting geometry and material](#5)
- [Collect all rooms on a given level](#6)
#### Forge Accelerator in Gothenburg
We have two [Forge accelerators](http://autodeskcloudaccelerator.com/) coming up in Europe
in the [next couple of months](http://autodeskcloudaccelerator.com/prague-2/):
- Gothenburg, Sweden – March 27-30
- Barcelona, Spain – June 12-16
I am planning to attend both and would love to see you there too.
In fact, the complete EMEA DevTech Team will be present to support you over the course of the week.
They take place in the Autodesk offices with lots of space and cool recreation areas.
Attendance is free of charge – all you pay for is your travel, accommodation and living expenses.
You can still apply to participate in either of these two events by sending your Forge App development project ideas to [adn-training-worldwide@autodesk.com](mailto:adn-training-worldwide@autodesk.com).
More information is available at [autodeskcloudaccelerator.com](http://autodeskcloudaccelerator.com).
Check out the videos on the landing page to hear what great results the attendees at previous accelerators achieved after just one week of intensive teamwork and training.
#### AEC Hackathon in Munich
Directly after the Gothenburg accelerator, March 31 - April 2,
the [AEC Hackathon Germany](http://aechackathon-germany.de) is
taking off at the Technical University Munich.
It gives everyone designing, building, and maintaining our built environment the opportunity to collaborate with cutting edge technologies and its developers and designers.
It’s a weekend of geeking at its finest for improving the industries that affect all that live or work in a house or building.
Jaime is going, and unfortunately, I am not...
#### Autodesk University in London
As I mentioned,
[Autodesk University is coming to London](http://thebuildingcoder.typepad.com/blog/2017/01/au-in-london-and-deep-learning.html#2),
to Tobacco Dock, E1 on June 21-22, 2017;
the first English speaking Autodesk University in Northern Europe!
Registration for Autodesk University London 2017 is now open.
Number are limited and an early bird discount is available until April 14.
To get a taste of what we’ve got in store for you this year, check out the venue, view the agenda and timings and find out all you need to know, visit the [AU London website](https://gems.autodesk.com/events/au-london-2017/event-summary-9dba4a429f994dbab348c68dfad1ca6a.aspx).
![AU London 2017](img/2017_au_london_2.png)
#### Retrieve and Map Texture UV Coordinates Exporting Geometry and Material
\*\*Question:\*\* I have a question looking at the rather outdated discussion
on [texture data UV coordinates and FBX](http://thebuildingcoder.typepad.com/blog/2010/02/texture-data-uv-coordinates-and-fbx.html):
I find that the API `Mesh.UVs` is not available in Revit 2014 or later.
How can I retrieve the mesh UVs now?
How can I map texture UV accurately when exporting Revit geometry and materials?
I already tried `Edge.TessellateOnFace(Face)` and `Face.GetBoundBoxingUV`; neither of them works as desired (not accurate).
```csharp
Mesh geomMesh = geomFace.Triangulate();
XYZArray vertices = geomMesh.Vertices;
UVArray uvs = geomMesh.UVs; // This API is not available
```
\*\*Answer:\*\* You should use the `CustomExporter` – the `OnMaterial` and `OnPolymesh` methods are designed for these sorts of export operations.
`XYZArray` is way obsolete, and most custom Revit API collections have been replaced by generic ones in the past few years, e.g., `List`, in this case.
#### Collect all Rooms on a Given Level
From
the [Revit API discussion forum](http://forums.autodesk.com/t5/revit-api-forum/bd-p/160) thread
on [collecting all rooms in level xx](https://forums.autodesk.com/t5/revit-api-forum/collect-all-room-in-leve-xx/m-p/6939202):
\*\*Question:\*\* How can I collect all rooms on a given level?
\*\*Answer 1:\*\* You can't collect `Rooms` directly, you need to collect `SpatialElement` instead, its parent class, and then post-process the results:
```csharp
public List GetRoomFromLevel(
Document document,
Level level )
{
List Rooms
= new FilteredElementCollector( document )
.OfClass( typeof( SpatialElement ) )
.WhereElementIsNotElementType()
.Where( room => room.GetType() == typeof( Room ) )
.ToList();
return new List( Rooms.Where( room
=> document.GetElement( room.LevelId ) == level )
.Select( r => r as Room ) );
}
```
The `Where` and `List<>` stuff comes from the `System.Collections.Generic` namespace.
\*\*Answer 2:\*\* Here is a new method `GetRoomsOnLevel` that I added
to [The Building Coder samples](https://github.com/jeremytammik/the_building_coder_samples)
[release 2017.0.132.8](https://github.com/jeremytammik/the_building_coder_samples/releases/tag/2017.0.132.8) sporting
some small improvements:
```csharp
#region Retrieve all rooms on a given level
///
/// Retrieve all rooms on a given level.
/// summary>
public IEnumerable GetRoomsOnLevel(
Document doc,
ElementId idLevel )
{
return new FilteredElementCollector( doc )
.WhereElementIsNotElementType()
.OfClass( typeof( SpatialElement ) )
.Where( e => e.GetType() == typeof( Room ) )
.Where( e => e.LevelId.IntegerValue.Equals(
idLevel.IntegerValue ) )
.Cast();
}
#endregion // Retrieve all rooms on a given level
```
This is more efficient than the first version due to:
- Elimination of `ToList`
- Elimination of `New List<>`
- Elimination of `doc.GetElement`
The reason you cannot filter directly for `Room` elements is explained in the discussion
on [filtering for a non-native class](http://thebuildingcoder.typepad.com/blog/2010/08/filtering-for-a-nonnative-class.html) and in
the [remarks on the `ElementClassFilter` class](http://www.revitapidocs.com/2017/4b7fb6d7-cb9c-d556-56fc-003a0b8a51b7.htm).