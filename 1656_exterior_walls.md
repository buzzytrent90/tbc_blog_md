---
post_number: "1656"
title: "Exterior Walls"
slug: "exterior_walls"
author: "Jeremy Tammik"
tags: ['elements', 'family', 'filtering', 'geometry', 'levels', 'parameters', 'revit-api', 'rooms', 'selection', 'sheets', 'transactions', 'views', 'walls']
source_file: "1656_exterior_walls.md"
original_url: "https://thebuildingcoder.typepad.com/blog/1656_exterior_walls.html"
---

### FilterRule Use and Retrieving Exterior Walls
Today, we revisit the interesting and generic question on retrieving all exterior walls.
That may be easy in a perfect and complete model.
However, it raises some challenges in an incomplete BIM:
- [Retrieving all exterior walls](#2)
- [Several possible approaches](#3)
- [Using a computational geometry approach](#4)
- [Manually adding the huge surrounding room](#5)
- [Encapsulate transactions and roll back instead of deleting](#6)
- [Determining model extents via wall bounding box](#7)
- [Implementing the huge surrounding room approach](#8)
- [Retrieving family instances satisfying a filter rule](#9)
#### Retrieving All Exterior Walls
This time around, this question was raised
by Feng [@718066900](https://forums.autodesk.com/t5/user/viewprofilepage/user-id/6055195) Wang in
the [Revit API discussion forum](http://forums.autodesk.com/t5/revit-api-forum/bd-p/160) thread
on [how to get all the outermost walls in the model](https://forums.autodesk.com/t5/revit-api-forum/how-do-i-get-all-the-outermost-walls-in-the-model/m-p/7998948).
We already explored some aspects last week,
on [retrieving all exterior walls](http://thebuildingcoder.typepad.com/blog/2018/05/drive-revit-via-a-wcf-service-wall-directions-and-parameters.html#8).
Today, we can present a working solution for an incomplete BIM.
\*\*Question:\*\* How do I get all the outermost walls in the model?
Here is a picture showing what I mean:
![Exterior walls](img/exterior_walls.png)
Here is the [sample model `exterior_walls.rvt`](zip/exterior_walls_2018.rvt).
#### Several Possible Approaches
Several approaches to solve this were already brought up [last week](http://thebuildingcoder.typepad.com/blog/2018/05/drive-revit-via-a-wcf-service-wall-directions-and-parameters.html#8):
- The `DirectionCalculation` Revit SDK sample and The Building Coder discussion of it
on [south facing walls](http://thebuildingcoder.typepad.com/blog/2010/01/south-facing-walls.html) solves
using the built-in wall function parameter `FUNCTION_PARAM` to filter for exterior walls.
However, the function parameter is not always correctly set on the wall type, and the wall type is not always correctly assigned.
- The Revit API provides a `BuildingEnvelopeAnalyzer` class that will retrieve all exterior walls for you.
However, it relies on the building model being properly enclosed, i.e., roof and floor elements must be added to form properly enclosed spaces in order for the analyser to work.
- Workaround suggestion: Temporarily place room separation lines outside the building envelope and create a huge room around the entire building.
Then, it’s just a matter of retrieving the room boundaries, filtering out the separation lines, appending the remaining elements to your list and deleting the room and separation lines again.
As we discovered, the first two approaches above cannot be applied to the incomplete BIM at hand.
For instance, here is the erroneous result of applying the `BuildingEnvelopeAnalyzer` to it:
![BuildingEnvelopeAnalyzer returns wrong walls](img/building_envelope_analayzer_wrong_walls.png)
There is no closure, no ceiling or floor, because I want to determine the outermost walls first to automatically create the ceiling and floor.
Getting the outermost walls first will enable more automation.
If the building is 'open' upward and downward, in theory, all walls are exposed to the outside, and therefore all of them are 'exterior'.
Happily, the third approach above can still be used in this 2D situation.
#### Using a Computational Geometry Approach
It would also be possible to solve this task through a geometric algorithm, of course.
Many different approaches can be taken here as well.
I would love to discover a really reliable one that works under all circumstances.
Here is an idea that comes to mind right now on the fly:
You can easily [determine whether a given point lies within a given polygon](https://www.ics.uci.edu/~eppstein/161/960307.html).
I also implemented a [point containment algorithm for the Revit API](http://thebuildingcoder.typepad.com/blog/2010/12/point-in-polygon-containment-algorithm.html),
and a [room in area predicate using it](http://thebuildingcoder.typepad.com/blog/2012/08/room-in-area-predicate-via-point-in-polygon-test.html).
Now, if you have all your walls, their location curves (if they are non-linear, things get trickier), and endpoints, and are sure that they all form closed polygons, you could determine the maximal polygon enclosing all others by choosing the one that contains the maximum number of wall endpoints.
You might also be able to use some library providing 2D polygon or Boolean operations for this.
Some such libraries, other options and helpful ideas are discussed in The Building Coder topic group
on [2D Booleans and adjacent areas](http://thebuildingcoder.typepad.com/blog/about-the-author.html#5.2).
We also recently discussed [determining the outermost loop of a face](http://thebuildingcoder.typepad.com/blog/2017/10/disjunct-outer-loops-from-planar-face-with-separate-parts.html).
However, in this case, making use of the built-in Revit room generation functionality is probably the easiest way to go.
#### Manually Adding the Huge Surrounding Room
I tried it out manually in the sample model, and it seems to work perfectly!
Add room separation lines around the outside of the building:
![Room separation lines](img/room_separation_lines.png)
Create a room around the building using them:
![Room around building](img/room_around_building.png)
This can easily be achieved programmatically as well.
Now all you need to do is retrieve the room boundary, eliminate the exterior separation line boundary segments, delete the separation lines and room, and you are done.
#### Encapsulate Transactions and Roll Back Instead of Deleting
Feng Wang implemented a solution based on these suggestions in [CmdGetOutermosWallByCreateRoom.zip](zip/CmdGetOutermosWallByCreateRoom.zip).
Here are my initial comments on his code that I keep repeating again and again, and therefore here now yet again:
- [Encapsulate transactions in a `using` statement](http://thebuildingcoder.typepad.com/blog/2012/04/using-using-automagically-disposes-and-rolls-back.html)
- You can encapsulate your multiple transactions in a [transaction group](http://thebuildingcoder.typepad.com/blog/about-the-author.html#5.53)
- Instead of creating objects in the model, extracting information from them, and then deleting them again, you could roll back the outermost transaction group to revert back to the original, unmodified, state. I nicknamed that
the ['temporary transaction trick'](http://thebuildingcoder.typepad.com/blog/about-the-author.html#5.53).
#### Determining Model Extents via Wall Bounding Box
To add a room around the entire building, we need to determine the building extents, or, at least, the maximal extents of all exterior walls.
One approach to achieve that might be to query each wall for its geometry or location curve, extract all their vertices, and construct a bounding box from them.
However, querying a Revit element for its bounding box is much faster and more efficient than accessing and analysing its geometry or location curve.
Moreover, [The Building Coder samples](https://github.com/jeremytammik/the_building_coder_samples)
[`Util` class](https://github.com/jeremytammik/the_building_coder_samples/blob/master/BuildingCoder/BuildingCoder/Util.cs) already
implements a bounding box extension method `ExpandToContain` that we can use here, which expands a given bounding box to encompass another one:
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
With that functionality, we can easily retrieve the maximum extents of all the walls:
```csharp
///
/// Return a bounding box around all the
/// walls in the entire model; for just a
/// building, or several buildings, this is
/// obviously equal to the model extents.
/// summary>
static BoundingBoxXYZ GetBoundingBoxAroundAllWalls(
Document doc,
View view = null )
{
// Default constructor creates cube from -100 to 100;
// maybe too big, but who cares?
BoundingBoxXYZ bb = new BoundingBoxXYZ();
FilteredElementCollector walls
= new FilteredElementCollector( doc )
.OfClass( typeof( Wall ) );
foreach( Wall wall in walls )
{
bb.ExpandToContain(
wall.get_BoundingBox(
view ) );
}
return bb;
}
```
#### Implementing the Huge Surrounding Room Approach
Now we are ready to apply the temporary transaction trick, create the room, query it for its boundary and retrieve the exterior walls.
This is implemented by the following methods:
- `RetrieveWallsGeneratingRoomBoundaries` retrieves all walls that generate boundary segments for the given room using the `BoundarySegment` `ElementId` property.
- `GetOutermostWalls` determines the maximum model extents, temporarily generates room boundary lines and a new room outside them, and retrieves the room boundary walls.
- The external command `Execute` method determines the exterior walls and highlights them for the user by adding them to the current selection set.
```csharp
///
/// 过滤出需要的墙体 --
/// Return all walls that are generating boundary
/// segments for the given room. Includes debug
/// code to compare wall lengths and wall areas.
/// summary>
static List
RetrieveWallsGeneratingRoomBoundaries(
Document doc,
Room room )
{
List ids = new List();
IList> boundaries
= room.GetBoundarySegments(
new SpatialElementBoundaryOptions() );
int n = boundaries.Count;
int iBoundary = 0, iSegment;
foreach( IList b in boundaries )
{
++iBoundary;
iSegment = 0;
foreach( BoundarySegment s in b )
{
++iSegment;
// Retrieve the id of the element that
// produces this boundary segment
Element neighbour = doc.GetElement(
s.ElementId );
Curve curve = s.GetCurve();
double length = curve.Length;
if( neighbour is Wall )
{
Wall wall = neighbour as Wall;
Parameter p = wall.get_Parameter(
BuiltInParameter.HOST_AREA_COMPUTED );
double area = p.AsDouble();
LocationCurve lc
= wall.Location as LocationCurve;
double wallLength = lc.Curve.Length;
ids.Add( wall.Id );
}
}
}
return ids;
}
///
/// 获取当前模型指定视图内的所有最外层的墙体
/// Get all the outermost walls in the
/// specified view of the current model
/// summary>
/// param>
/// 视图,默认是当前激活的视图
/// View, default is currently active viewparam>
public static List GetOutermostWalls(
Document doc,
View view = null )
{
double offset = Util.MmToFoot( 1000 );
if( view == null )
{
view = doc.ActiveView;
}
BoundingBoxXYZ bb = GetBoundingBoxAroundAllWalls(
doc, view );
XYZ voffset = offset \* ( XYZ.BasisX + XYZ.BasisY );
bb.Min -= voffset;
bb.Max += voffset;
XYZ[] bottom_corners = Util.GetBottomCorners(
bb, 0 );
CurveArray curves = new CurveArray();
for( int i = 0; i < 4; ++i )
{
int j = i < 3 ? i + 1 : 0;
curves.Append( Line.CreateBound(
bottom_corners[i], bottom_corners[j] ) );
}
using( TransactionGroup group
= new TransactionGroup( doc ) )
{
Room newRoom = null;
group.Start( "Find Outermost Walls" );
using( Transaction transaction
= new Transaction( doc ) )
{
transaction.Start(
"Create New Room Boundary Lines" );
SketchPlane sketchPlane = SketchPlane.Create(
doc, view.GenLevel.Id );
ModelCurveArray modelCaRoomBoundaryLines
= doc.Create.NewRoomBoundaryLines(
sketchPlane, curves, view );
// 创建房间的坐标点 -- Create room coordinates
double d = Util.MmToFoot( 600 );
UV point = new UV( bb.Min.X + d, bb.Min.Y + d );
// 根据选中点，创建房间 当前视图的楼层 doc.ActiveView.GenLevel
// Create room at selected point on the current view level
newRoom = doc.Create.NewRoom( view.GenLevel, point );
if( newRoom == null )
{
string msg = "创建房间失败。";
TaskDialog.Show( "xx", msg );
transaction.RollBack();
return null;
}
RoomTag tag = doc.Create.NewRoomTag(
new LinkElementId( newRoom.Id ),
point, view.Id );
transaction.Commit();
}
//获取房间的墙体 -- Get the room walls
List ids
= RetrieveWallsGeneratingRoomBoundaries(
doc, newRoom );
group.RollBack(); // 撤销
return ids;
}
}
public Result Execute(
ExternalCommandData commandData,
ref string message,
ElementSet elements )
{
UIApplication uiapp = commandData.Application;
UIDocument uidoc = uiapp.ActiveUIDocument;
Document doc = uidoc.Document;
List ids = GetOutermostWalls( doc );
uidoc.Selection.SetElementIds( ids );
return Result.Succeeded;
}
```
Many thanks to Feng Wang and the development team for helping to sort this out!
#### Retrieving Family Instances Satisfying a Filter Rule
Once again, in a completely unrelated area,
Frank [@Fair59](https://forums.autodesk.com/t5/user/viewprofilepage/user-id/2083518) Aarssen
comes to the rescue, providing a succinct answer to
the [Revit API discussion forum](http://forums.autodesk.com/t5/revit-api-forum/bd-p/160) thread
on ‎[how to filter for elements which satisfy a filter rule](https://forums.autodesk.com/t5/revit-api-forum/how-to-filter-element-which-satisfy-filter-rule/m-p/8021978):
\*\*Question:\*\* I'm trying to get the family instances which satisfy a filter rule as shown in this image:
![Filters form](img/filters_form.png)
So far, I'm able to get the list of a category that has a specific filter name.
However, I'd like to get the family instances of those categories which satisfy the filter rule.
I'm not sure how to do that via API.
\*\*Explanation:\*\* Let's say, in Revit, someone needs to find all the walls on the Level 1 or a wall that has some thickness value xyz; they apply a filter rule, and all the walls that satisfy a filter rule get highlighted.
We need this functionality in an add-in, so we could develop a BIM-Explorer for our modellers to explore and navigate any element easily.
The same idea was implemented by Ideate Software in their explorer add-in, cf. the 12-minute demo on [Auditing Your Revit Project with Ideate Explorer](https://youtu.be/KP7XFv_VL6M):

For further understanding, you can check out the [Boost Your BIM](https://boostyourbim.wordpress.com) explanation
of [Filter Rule data – where is it hiding](https://boostyourbim.wordpress.com/2016/05/11/filter-rule-data-where-is-it-hiding)?
\*\*Answer:\*\* You can filter the document, first for the categories of the filter, then for each filter rule:
```csharp
FilteredElementCollector pfes
= new FilteredElementCollector( doc )
.OfClass( typeof( ParameterFilterElement ) );
foreach( ParameterFilterElement pfe in pfes )
{
#region Get Filter Name, Category and Elements underlying the categories
ElementMulticategoryFilter catfilter
= new ElementMulticategoryFilter(
pfe.GetCategories() );
FilteredElementCollector elemsByFilter
= new FilteredElementCollector( doc )
.WhereElementIsNotElementType()
.WherePasses( catfilter );
foreach( FilterRule rule in pfe.GetRules() )
{
IEnumerable elemsByFilter2
= elemsByFilter.Where( e
=> rule.ElementPasses( e ) );
}
#endregion
```
By the way, `ParameterFilterElement.GetRules` is obsolete in Revit 2019 and can be replaced by `GetElementFilter` in future.
Many thanks to Frank for the solution and to Ali [@imaliasad](https://forums.autodesk.com/t5/user/viewprofilepage/user-id/5242763) Asad for raising the question and explaining it further to me.