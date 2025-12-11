---
ai_optimized: true
author: Jeremy Tammik
complexity_score: 9.4
content_type: code_example
optimization_date: '2025-12-11T11:44:16.063724'
original_url: https://thebuildingcoder.typepad.com/blog/1456_bounding_section_box.html
post_number: '1456'
reading_time_minutes: 16
series: geometry
slug: bounding_section_box
source_file: 1456_bounding_section_box.md
tags:
- elements
- filtering
- geometry
- python
- revit-api
- rooms
- selection
- sheets
- views
- walls
- windows
title: The Building Coder
word_count: 3153
---

### Vacation End, Forge News and Bounding Boxes
I am back from a very relaxing vacation, spent in the French Jura countryside on the western Swiss border and in Ticino in the south of Switzerland.
I did next to nothing, and that felt fine.
My greatest achievement was probably climbing the (very slippery welded steel) summit cross on Mount Tamaro – :-)
![Jeremy on Mount Tamaro](img/20160808_152704_jeremy_tamaro_gipfelkreuz_400.jpg)
Meanwhile, obviously, lots of important and exciting Forge community and Revit API related happenings, of which I will just name a first few examples to start off blogging again:
- [PyRevit Blog](#2)
- [Forge DevCon 2016 material and 2017 dates](#3)
- [Forge forums closing in favour of StackOverflow](#4)
- [Forge Accelerator in Munich](#5)
- [Bounding Box `ExpandToContain` Extension Methods](#6)
- [Bounding Box and Lower Left Corner of Rooms](#7)
- [Bounding Box of Selected Elements or Entire Model](#8)
- [Setting 3D section box to selected elements' extents](#9)
#### PyRevit Blog
PyRevit has a blog now, hosted by GitHub pages – :-)

[eirannejad.github.io/pyRevit](http://eirannejad.github.io/pyRevit)
#### Forge DevCon 2016 Material and 2017 Dates
The [Forge DevCon 2016 developer conference material](http://adndevblog.typepad.com/cloud_and_mobile/2016/08/content-from-forge-devcon-2016.html) is
now published and online.
In case you like to plan far ahead, as is common here in Switzerland, please also take note that the dates for the Forge DevCon 2017 developer conference have been settled: June 27-28, 2017.
#### Forge Forums Closing in Favour of StackOverflow
As you probably are aware, all Forge questions are now being discussed on [StackOverflow](http://stackoverflow.com).
Simply search for the StackOverflow tags `autodesk`, `forge`, and the various Forge APIs. Typing the former currently lists the following:
- [autodesk](http://stackoverflow.com/questions/tagged/autodesk)
- [autodesk-forge](http://stackoverflow.com/questions/tagged/autodesk-forge)
- [autodesk-viewer](http://stackoverflow.com/questions/tagged/autodesk-viewer)
- [autodesk-model-derivative](http://stackoverflow.com/questions/tagged/autodesk-model-derivative)
- [autodesk-inventor](http://stackoverflow.com/questions/tagged/autodesk-inventor)
- [autodesk-vault](http://stackoverflow.com/questions/tagged/autodesk-vault)
- [autodesk-data-management](http://stackoverflow.com/questions/tagged/autodesk-data-management)
- [autodesk-designautomation](http://stackoverflow.com/questions/tagged/autodesk-designautomation)
- [autodesk-d-print](http://stackoverflow.com/questions/tagged/autodesk-3d-print)
Accordingly, the Autodesk discussion forums with the obsolete names 'View & Data API' and 'AutoCAD I/O' are now closed for new posts:
- [AutoCAD I/O now closing](http://forums.autodesk.com/t5/autocad-i-o/this-forum-is-closing/td-p/6476336)
- [View & Data now closing](http://forums.autodesk.com/t5/view-and-data-api/this-forum-is-closing/td-p/6476330)
- [Getting help for Forge APIs](http://adndevblog.typepad.com/cloud_and_mobile/2016/08/getting-help-for-forge-apis.html)
By the way, regarding Revit, I hope that you are also aware of the Revit related tags:
- [revit](http://stackoverflow.com/questions/tagged/revit)
- [revit-api](http://stackoverflow.com/questions/tagged/revit-api)
- [revitpythonshell](http://stackoverflow.com/questions/tagged/revitpythonshell)
- [revit-2015](http://stackoverflow.com/questions/tagged/revit-2015)
- [pyrevit](http://stackoverflow.com/questions/tagged/pyrevit)
#### Forge Accelerator in Munich
The next [Forge Accelerator](http://autodeskcloudaccelerator.com) is taking place in Munich, Germany, October 24-28, 2016.
Mark your calendar, and please submit your proposal, if you are interested in participating.
The proposal deadline is in September.
I will be there, and I hope you will too!
#### Bounding Box `ExpandToContain` Extension Methods
Returning to the Revit API, a number of recent developer support issues
and [Revit API discussion forum](http://forums.autodesk.com/t5/revit-api/bd-p/160) threads
address questions related to determining a bounding box for selected elements or the entire model, the lower left corner of rooms, and setting the section box of a 3D view to a selected element's bounding box.
In order to help address these, I started off by implementing two `ExpandToContain` extension methods for the `BoundingBoxXYZ` class, to expand an existing bounding box by adding a `XYZ` point or another bounding box to it:
```csharp
public static class JtBoundingBoxXyzExtensionMethods
{
///
/// Expand the given bounding box to include
/// and contain the given point.
/// summary>
public static void ExpandToContain(
this BoundingBoxXYZ bb,
XYZ p )
{
bb.Min = new XYZ( Math.Min( bb.Min.X, p.X ),
Math.Min( bb.Min.Y, p.Y ),
Math.Min( bb.Min.Z, p.Z ) );
bb.Max = new XYZ( Math.Max( bb.Max.X, p.X ),
Math.Max( bb.Max.Y, p.Y ),
Math.Max( bb.Max.Z, p.Z ) );
}
///
/// Expand the given bounding box to include
/// and contain the given other one.
/// summary>
public static void ExpandToContain(
this BoundingBoxXYZ bb,
BoundingBoxXYZ other )
{
bb.ExpandToContain( other.Min );
bb.ExpandToContain( other.Max );
}
}
```
Check it out live
in [The Building Coder samples](https://github.com/jeremytammik/the_building_coder_samples)
[Util.cs utility class](https://github.com/jeremytammik/the_building_coder_samples/blob/master/BuildingCoder/BuildingCoder/Util.cs), currently
in [lines 1275-1305](https://github.com/jeremytammik/the_building_coder_samples/blob/master/BuildingCoder/BuildingCoder/Util.cs#L1275-L1305).
Here follow some usage examples.
#### Bounding Box and Lower Left Corner of Rooms
Question on [how to generate x and y-axes of rooms in the Revit API in millimetres](http://forums.autodesk.com/t5/revit-api/how-to-generate-x-and-y-axes-of-rooms-in-the-revit-api-in/m-p/6452079):
I'm trying to write code that gives the x and y-coordinates of the bottom-left point of each room in the design, based on the length in millimetre. If for somehow the code sets the bottom-left corner of the design at (0,0) and then from there if wall surrounds all the inner space such that wall width is 290mm, then the first room at the bottom-left corner starts at (291,291) coordinates. The code reads all the rooms and returns the list of coordinates in (mm) for each of them.
I tried to use the following code to achieve this:

```
  Location loc = room.Location;
  LocationPoint lp = loc as LocationPoint;
  XYZ p = (null == lp) ? XYZ.Zero : lp.Point;
  msg.Add(room.Name + " - " + p.X + ", "+ p.Y);
```

However, I found some of the X and Y axes are in minus and moreover they are not in millimetre, which is meaningless for my purpose.
![Room corners](img/room_corners_annotated_design.png)
\*\*Answer:\*\* This is easy to answer and solve.
Two steps are required:
First of all, all Revit database [length measurements are in imperial feet](http://thebuildingcoder.typepad.com/blog/2011/03/internal-imperial-units.html).
There is no way to change that.
Therefore, when you retrieve coordinates via the Revit API, they will always be in feet, and you will have to convert them to millimetres yourself.
That is very easy, of course.
Secondly, if you are interested in the lower left-hand corner of the room, the room location point is of absolutely no use whatsoever.
Instead, you could, for instance, retrieve the room boundary, iterate over all the curves, and determine which endpoint is on the lower left.
If your rooms have straight edges, this is trivial.
If the edges are curved, it might get a bit more complicated.
Third, you might also be able to use the room bounding box. That would be simpler still, but maybe less precise.
I implemented the second option for you
in [The Building Coder samples](https://github.com/jeremytammik/the_building_coder_samples), in the
module [CmdListAllRooms.cs](https://github.com/jeremytammik/the_building_coder_samples/blob/master/BuildingCoder/BuildingCoder/CmdListAllRooms.cs).
In it, I iterate over all the room boundary segments, tessellate them, and use the resulting points to generate an accurate 2D bounding box for the room:
```csharp
///
/// Return bounding box calculated from the room
/// boundary segments. The lower left corner turns
/// out to be identical with the one returned by
/// the standard room bounding box.
/// summary>
BoundingBoxXYZ GetBoundingBox(
IList> boundary )
{
BoundingBoxXYZ bb = new BoundingBoxXYZ();
double infinity = double.MaxValue;
bb.Min = new XYZ( infinity, infinity, infinity );
bb.Max = -bb.Min;
foreach( IList loop in boundary )
{
foreach( BoundarySegment seg in loop )
{
Curve c = seg.GetCurve();
IList pts = c.Tessellate();
foreach( XYZ p in pts )
{
bb.ExpandToContain( p );
}
}
}
return bb;
}
///
/// List some properties of a given room to the
/// Visual Studio debug output window.
/// summary>
void ListRoomData( Room room )
{
SpatialElementBoundaryOptions opt
= new SpatialElementBoundaryOptions();
string nr = room.Number;
string name = room.Name;
double area = room.Area;
Location loc = room.Location;
LocationPoint lp = loc as LocationPoint;
XYZ p = ( null == lp ) ? XYZ.Zero : lp.Point;
BoundingBoxXYZ bb = room.get_BoundingBox( null );
IList> boundary
= room.GetBoundarySegments( opt );
int nLoops = boundary.Count;
int nFirstLoopSegments = 0 < nLoops
? boundary[0].Count
: 0;
BoundingBoxXYZ boundary_bounding_box
= GetBoundingBox( boundary );
Debug.Print( string.Format(
"Room nr. '{0}' named '{1}' at {2} with "
+ "lower left corner {3}, "
+ "bounding box {4} and area {5} sqf has "
+ "{6} loop{7} and {8} segment{9} in first "
+ "loop.",
nr, name, Util.PointString( p ),
Util.PointString( boundary_bounding_box.Min ),
BoundingBoxString2( bb ), area, nLoops,
Util.PluralSuffix( nLoops ), nFirstLoopSegments,
Util.PluralSuffix( nFirstLoopSegments ) ) );
}
```
That generates the following result for as simple test with ten rooms, showing that the result from the room boundary edges is identical with the standard, simpler and more efficiently Revit element accessible bounding box for these simple rectangular samples:

```
Room nr. '1' named 'Room 1' at (-31.85,19.49,0) with lower left corner (-39.74,13.41,0), bounding box ((-39.74,13.41,0),(-20.71,25.88,13.12)) and area 237.236585584282 sqf has 1 loop and 4 segments in first loop.
Room nr. '2' named 'Room 2' at (-31.85,5.67,0) with lower left corner (-39.74,0.29,0), bounding box ((-39.74,0.29,0),(-20.71,12.76,13.12)) and area 237.236585584282 sqf has 1 loop and 4 segments in first loop.
Room nr. '3' named 'Room 3' at (-16.54,19.49,0) with lower left corner (-20.05,13.41,0), bounding box ((-20.05,13.41,0),(-10.87,25.88,13.12)) and area 114.528006833791 sqf has 1 loop and 4 segments in first loop.
Room nr. '4' named 'Room 4' at (-16.54,5.67,0) with lower left corner (-20.05,0.29,0), bounding box ((-20.05,0.29,0),(-10.87,12.76,13.12)) and area 114.528006833791 sqf has 1 loop and 4 segments in first loop.
Room nr. '5' named 'Room 5' at (-4.97,19.49,0) with lower left corner (-10.21,13.41,0), bounding box ((-10.21,13.41,0),(-1.02,25.88,13.12)) and area 114.528006833791 sqf has 1 loop and 4 segments in first loop.
Room nr. '6' named 'Room 6' at (-4.97,5.67,0) with lower left corner (-10.21,0.29,0), bounding box ((-10.21,0.29,0),(-1.02,12.76,13.12)) and area 114.528006833791 sqf has 1 loop and 4 segments in first loop.
Room nr. '7' named 'Room 7' at (3.62,19.49,0) with lower left corner (-0.37,13.41,0), bounding box ((-0.37,13.41,0),(8.82,25.88,13.12)) and area 114.528006833791 sqf has 1 loop and 4 segments in first loop.
Room nr. '8' named 'Room 8' at (3.62,5.67,0) with lower left corner (-0.37,0.29,0), bounding box ((-0.37,0.29,0),(8.82,12.76,13.12)) and area 114.528006833791 sqf has 1 loop and 4 segments in first loop.
Room nr. '9' named 'Room 9' at (14.45,19.49,0) with lower left corner (9.47,13.41,0), bounding box ((9.47,13.41,0),(18.66,25.88,13.12)) and area 114.528006833791 sqf has 1 loop and 4 segments in first loop.
Room nr. '10' named 'Room 10' at (14.45,5.67,0) with lower left corner (9.47,0.29,0), bounding box ((9.47,0.29,0),(18.66,12.76,13.12)) and area 114.528006833789 sqf has 1 loop and 4 segments in first loop.
```

In other words, I believe that the second and third suggestions above will always generate the same results.
#### Bounding Box of Selected Elements or Entire Model
Another issue revolving around bounding boxes for selected elements or the entire model extents was raised in
the [Revit API discussion forum](http://forums.autodesk.com/t5/revit-api/bd-p/160) thread
on [how to get the total width and length of the design](http://forums.autodesk.com/t5/revit-api/how-to-get-the-total-width-and-length-of-the-design-using-revit/m-p/6419530):
\*\*Question:\*\* I want to get the total width (along x-axis) and the total depth/length (along y-axis) of the given design.
Does anyone know what would be the best way to achieve it using Revit API?
\*\*Answer:\*\* The number of possible approaches is infinite.
I would probably start off
by [computing the convex hull](http://thebuildingcoder.typepad.com/blog/2009/06/convex-hull-and-volume-computation.html) of
the entire design, and then determine the width and length of that.
Another approach, which may or may not be faster, depending on many factors:
Skip the convex hull part and just iterate over all building elements, enlarging a bounding box containing all their vertices as you go along:

```
  bounding box B = (+infinity, -infinity)

  for all elements
    for all element geometry vertices P
      if P < B.min: B.min = P
      if P > B.max: B.max = P
```

Another approach, definitely faster than the last:
Use the element bounding box vertices instead of all the element geometry vertices.
All of these approaches would make very nice little samples.
There are certainly more ways to approach this task.
I have a strong personal tendency towards the pure geometric.
Implement them and provide the code and a couple of examples designs to test them on, including extreme cases, and I'll clean it up and publish it for you on The Building Coder.
Funnily enough, a similar question was raised shortly afterwards by another developer:
\*\*Question:\*\* Is there a built-in method to get a bounding box of the entire model, similar to the method `Element.get_BoundingBox`?
The only way I see right now is to use `IExportContext`, go through all the visible elements and get the minimum and maximum coordinates among the all points. But on the large models this method may take a while.
Is there some faster method?
\*\*Answer:\*\* Yes, almost certainly.
I already suggested three different approaches above.
One aspect that I did not mention that makes a huge difference for large models:
The bounding box of all elements is immediately available from the element header information, and thus corresponds to a **quick** filtering method, whereas all the approaches that require the full geometry correspond to a **slow** filter.
Using a custom exporter, like you suggest, is also a good idea, and corresponds to a slow filter.
If you are interested in good performance in large models, I would aim for a quick filter method, e.g., like this, using the bounding box extension methods provided above:
```csharp
#region Get Model Extents
///
/// Return a bounding box enclosing all model
/// elements using only quick filters.
/// summary>
BoundingBoxXYZ GetModelExtents( Document doc )
{
FilteredElementCollector quick_model_elements
= new FilteredElementCollector( doc )
.WhereElementIsNotElementType()
.WhereElementIsViewIndependent();
IEnumerable bbs = quick_model_elements
.Where( e => null != e.Category )
.Select( e
=> e.get_BoundingBox( null ) );
return bbs.Aggregate( ( a, b )
=> { a.ExpandToContain( b ); return a; } );
}
#endregion // Get Model Extents
```
I implemented that for you and included it
in [The Building Coder samples](https://github.com/jeremytammik/the_building_coder_samples)
[release 2017.0.127.6](https://github.com/jeremytammik/the_building_coder_samples/releases/tag/2017.0.127.6).
\*\*Response:\*\* So, in fact I was right and the only method is to get this information from the geometry.
I’ll benchmark with CustomExport (as I suggested) and FilteredElementCollector (as you did) with large models and let you know.
I think your way should be faster, but not correct in some cases:
1. Linked models. It doesn’t consider linked models if they are.
2. Even if we extend this method and get the elements geometry from the linked models, we need to transform the coordinates from linked model to the hosted model, as BoundingBox returns the coordinates in a source model.
3. Your method returns all the elements. In my case I need only the view specific Model BoundingBox. Yes, we can use FilteredElementCollector with ActiveView, but it can be a section view or 3d view bounded by section box.
Therefore, FilteredElementCollector should be faster, but needs to consider all the cases (maybe there are more than I mentioned). As CustomExporter iterate only the visible geometry, exactly like it presented on the view, this way is more reliable.
Anyway, the benchmark will show the results – :-)
#### Setting 3D Section Box to Selected Elements' Extents
Finally, my colleague Jim Jia just answered a related developer support issue on setting the section box of a 3D view to the extents of a selected element:
\*\*Question:\*\* I use `uiDoc.Selection.SetElementIds (elemIds); view3D.IsSectionBoxActive = true;` to show the section box, but that displays the entire project.
What I want is to display a selected element in the section box.
How can this be achieved, please?
\*\*Answer:\*\* You can set the size of the section box yourself.
If you want the section box to display only the currently selected element, simply calculate the appropriately sized `BoundingBoxXYZ` by retrieving the bounding box of the selected element.
Here is an example:
```csharp
#region SetSectionBox
///
/// Set 3D view section box to selected element extents.
/// summary>
private void SectionBox(UIDocument uidoc )
{
Document doc = uidoc.Document;
View view = doc.ActiveView;
double Min_X = double.MaxValue;
double Min_Y = double.MaxValue;
double Min_Z = double.MaxValue;
double Max_X = Min_X;
double Max_Y = Min_Y;
double Max_Z = Min_Z;
ICollection ids
= uidoc.Selection.GetElementIds();
foreach( ElementId id in ids )
{
Element elm = doc.GetElement( id );
BoundingBoxXYZ box = elm.get_BoundingBox( view );
if( box.Max.X > Max_X )
{
Max_X = box.Max.X;
}
if( box.Max.Y > Max_Y )
{
Max_Y = box.Max.Y;
}
if( box.Max.Z > Max_Z )
{
Max_Z = box.Max.Z;
}
if( box.Min.X < Min_X )
{
Min_X = box.Min.X;
}
if( box.Min.Y < Min_Y )
{
Min_Y = box.Min.Y;
}
if( box.Min.Z < Min_Z )
{
Min_Z = box.Min.Z;
}
}
XYZ Max = new XYZ( Max_X, Max_Y, Max_Z );
XYZ Min = new XYZ( Min_X, Min_Y, Min_Z );
BoundingBoxXYZ myBox = new BoundingBoxXYZ();
myBox.Min = Min;
myBox.Max = Max;
( view as View3D ).SetSectionBox( myBox );
}
#endregion // SetSectionBox
```
This method could obviously be simplified a bit by making use of the `ExpandToContain` extension methods above.
There you are.
Now I am starting to get back to normal again.
Everything discussed above is now online and live
in [The Building Coder samples](https://github.com/jeremytammik/the_building_coder_samples)
[release 2017.0.127.8](https://github.com/jeremytammik/the_building_coder_samples/releases/tag/2017.0.127.8).
Lots more stuff coming up.
Have fun!
Dear Bilal,
Thank you for your appreciation.
I summarised, cleaned up and published our conversation on The Building Coder:
http://thebuildingcoder.typepad.com/blog/2016/08/vacation-end-forge-news-and-bounding-boxes.html#8
Cheers,
Jeremy