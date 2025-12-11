---
ai_optimized: true
author: Jeremy Tammik
complexity_score: 5.5
content_type: qa
optimization_date: '2025-12-11T11:44:16.428396'
original_url: https://thebuildingcoder.typepad.com/blog/1632_filter_intersect.html
post_number: '1632'
reading_time_minutes: 12
series: filtering
slug: filter_intersect
source_file: 1632_filter_intersect.md
tags:
- csharp
- elements
- family
- filtering
- geometry
- references
- revit-api
- rooms
- sheets
- views
title: Filter Intersect
word_count: 2447
---

### Create 2D Arc and Filter for Intersecting Elements
Several questions on filtering for intersecting elements came up recently.
It is pretty easy as long as a bounding box can be used.
However, the bounding box is generally aligned with the cardinal X, Y and Z axes.
If the containing volume of interest is not, too many elements may be selected.
This can be addressed in various ways, e.g., post-processing the bounding box results, or using a more precise intersection filter.
- [Family instances in a diagonal room](#2)
- [Conduits intersecting a junction box](#3)
- [Generate 2D arc from radius, start and end points](#4)
#### Family Instances in a Diagonal Room
Kavitha asked the question in
his [comment](http://thebuildingcoder.typepad.com/blog/2013/03/filter-for-family-instances-in-a-room.html#comment-3781548196)
on [filtering for family instances in a room](http://thebuildingcoder.typepad.com/blog/2013/03/filter-for-family-instances-in-a-room.html):
\*\*Question:\*\* Can someone explain for me: if the room is diagonal, then the bounding box value will be wrong; in that case, how can I get the elements properly?
\*\*Answer:\*\* You are right, of course.
The bounding box will be too large and therefore possibly contain too many elements.
You can eliminate the elements outside the room by many means.
[IsPointInRoom](http://www.revitapidocs.com/2018.1/21d28ce3-3c1a-43cd-9714-0fe7223c5636.htm), for example, determines whether a given point lies within the volume of the room.
If you have an element with some geometry, e.g., a solid, you could test each of its vertices with that method.
If you are interested in the 2D plan view only, you could also implement a more precise test by performing a Boolean operation between a polygon containing the element and the room boundary polygon.
My [RoomEditorApp](https://github.com/jeremytammik/RoomEditorApp) exports
rooms and the family instances they contain to a cloud database; for that, it obviously determines which instances lie within which room, as mentioned in the note
on [retrieving all family instances in a room](http://thebuildingcoder.typepad.com/blog/2017/04/forgefader-ui-lookup-builds-purge-and-room-instances.html#5) elsewhere,
It just uses a filtered element collector based on the room bounding box, though, in the
method [`GetFurniture( Room room )`](https://github.com/jeremytammik/RoomEditorApp/blob/master/RoomEditorApp/CmdUploadRooms.cs#L157-L217),
so it does not answer your question on the diagonal case.
For a precise handling of the diagonal or any other irregularly shaped case, you could also retrieve the room solid from
its [`ClosedShell` property](http://www.revitapidocs.com/2018.1/1a510aef-63f6-4d32-c0ff-a8071f5e23b8.htm) and use
an [`ElementIntersectsSolidFilter`](http://www.revitapidocs.com/2018.1/19276b94-fa39-64bb-bfb8-c16967c83485.htm) based on that, cf. below.
#### Conduits Intersecting a Junction Box
Right after that, I started out discussing another intersection and containment issue with Tiago Cerqueria
in [his](http://thebuildingcoder.typepad.com/blog/2010/12/find-intersecting-elements.html#comment-3783584153)
[comments](http://thebuildingcoder.typepad.com/blog/2010/12/find-intersecting-elements.html#comment-3783587401)
on [finding intersecting elements](http://thebuildingcoder.typepad.com/blog/2010/12/find-intersecting-elements.html)
and StackOverflow question on [bounding box intersection](https://stackoverflow.com/questions/49070566/bounding-box-intersection):
\*\*Question:\*\* To find elements that are intersecting a geometry, I am using the example
to [find intersecting elements](http://thebuildingcoder.typepad.com/blog/2010/12/find-intersecting-elements.html).
The main goal of my add-in was find conduits clashing with a junction box and access all the elements connected with it to insert information.
But the bounding box is always parallel to the cardinal X, Y and Z axes, and this may cause a problem, like returning elements that are not really clashing, because sometimes the bounding box is not coincident with the geometry because the family instance is rotated.
Besides that, there is the problem that the bounding box will consider the geometry of the symbol and not the instance, and will consider the flipped geometry too, meaning that the bounding box is bigger than I am looking for.
Is there a way to get the real geometry that are in the currently view? How can I solve this problem?
\*\*Answer:\*\* There are many ways to address this.
Generally, when performing clash detection, you will always run a super fast pre-processing step first to determine candidate elements, and then narrow down the search step by step more precisely in following steps. In this case, you can consider the bounding box intersection the first step, and then perform post-processing afterwards to narrow down the result to your exact goal.
One important question is: does the bounding box really give you all the elements you need, plus more? Are you sure there are none missing?
Once that is settled, all you need to do is add post-processing steps applying the detailed considerations that you care about to remove superfluous elements.
A simple property to check might be: are all the target element geometry vertices contained in the target volume?
A more complex one might involve retrieving the full solid of the target element and the target volume and performing a Boolean intersection between them to determine completely and exactly whether they intersect, are disjunct, or contained in each other.
Many others are conceivable.
\*\*Response:\*\* I am using another strategy that accesses the geometry of the instance to verify whether the face of the family instance clashes with a closer conduit:
Here follows the code that I created and that works perfectly.
I am accessing the geometry of the family by the method `get_Geometry(options)` to retrieve the instance geometry located in project coordinates. From this I get the face and verify whether there is an intersection:
```csharp
class FindIntersection
{
public Conduit ConduitRun { get; set; }
public FamilyInstance Jbox { get; set; }
public List GetListOfConduits = new List();
public FindIntersection(
FamilyInstance jbox,
UIDocument uiDoc )
{
XYZ jboxPoint = ( jbox.Location
as LocationPoint ).Point;
FilteredElementCollector filteredCloserConduits
= new FilteredElementCollector( uiDoc.Document );
List listOfCloserConduit
= filteredCloserConduits
.OfClass( typeof( Conduit ) )
.ToList()
.Where( x
=> ( ( x as Conduit ).Location as LocationCurve ).Curve
.GetEndPoint( 0 ).DistanceTo( jboxPoint ) < 30
|| ( ( x as Conduit ).Location as LocationCurve ).Curve
.GetEndPoint( 1 ).DistanceTo( jboxPoint ) < 30 )
.ToList();
// getting the location of the box and all conduit around.
Options opt = new Options();
opt.View = uiDoc.ActiveView;
GeometryElement geoEle = jbox.get_Geometry( opt );
// getting the geometry of the element to
// access the geometry of the instance.
foreach( GeometryObject geomObje1 in geoEle )
{
GeometryElement geoInstance = ( geomObje1
as GeometryInstance ).GetInstanceGeometry();
// the geometry of the family instance can be
// accessed by this method that returns a
// GeometryElement type. so we must get the
// GeometryObject again to access the Face of
// the family instance.
if( geoInstance != null )
{
foreach( GeometryObject geomObje2 in geoInstance )
{
Solid geoSolid = geomObje2 as Solid;
if( geoSolid != null )
{
foreach( Face face in geoSolid.Faces )
{
foreach( Element cond in listOfCloserConduit )
{
Conduit con = cond as Conduit;
Curve conCurve = ( con.Location as LocationCurve ).Curve;
SetComparisonResult set = face.Intersect( conCurve );
if( set.ToString() == "Overlap" )
{
//getting the conduit the intersect the box.
GetListOfConduits.Add( con );
}
}
}
}
}
}
}
}
}
```
\*\*Answer:\*\* I considered this problem myself as well in the meantime and thought of two filters that may achieve a similar result more efficiently.
Using those, you do not have to retrieve the conduits and their geometry one by one and analyse them yourself, because the filters already do it for you.
One is quick and compares the axis-aligned bounding box.
The other is slow and intersects the exact element geometry.
It is important to realise and always have in mind
the [important difference between quick and slow filters](http://thebuildingcoder.typepad.com/blog/2015/12/quick-slow-and-linq-element-filtering.html#3).
You may want to try them out yourself:
```csharp
Element e = Util.SelectSingleElement(
uidoc, "a junction box" );
BoundingBoxXYZ bb = e.get_BoundingBox( null );
Outline outLne = new Outline( bb.Min, bb.Max );
// Use a quick bounding box filter - axis aligned
ElementQuickFilter fbb
= new BoundingBoxIntersectsFilter( outLne );
FilteredElementCollector conduits
= new FilteredElementCollector( doc )
.OfClass( typeof( Conduit ) )
.WherePasses( fbb );
// How many elements did we find?
int nbb = conduits.GetElementCount();
// Use a slow intersection filter - exact results
ElementSlowFilter intersect_junction
= new ElementIntersectsElementFilter( e );
conduits = new FilteredElementCollector( doc )
.OfClass( typeof( Conduit ) )
.WherePasses( intersect_junction );
// How many elements did we find?
int nintersect = conduits.GetElementCount();
Debug.Assert( nintersect <= nbb,
"expected element intersection to be stricter"
+ "than bounding box containment" );
```
I implemented a new external
command [CmdIntersectJunctionBox](https://github.com/jeremytammik/the_building_coder_samples/blob/master/BuildingCoder/BuildingCoder/CmdIntersectJunctionBox.cs)
in [The Building Coder samples](https://github.com/jeremytammik/the_building_coder_samples) to try this out, in
[release 2018.0.137.0](https://github.com/jeremytammik/the_building_coder_samples/releases/tag/2018.0.137.0).
Waiting for a suitable sample model to test this on...
Thank you, Thiago, for sharing your alternative approach!
#### Generate 2D Arc from Radius, Start and End Points
Finally, a pure geometric non-filtering question that came up today and was answered very precisely and exhaustively by Scott Wilson,
on [creating an arc when only the radius, start and end point is known](https://forums.autodesk.com/t5/revit-api-forum/create-a-curve-when-only-the-start-point-end-point-amp-radius-is/m-p/7830079):
\*\*Question:\*\* I am attempting to create an arc, but I only know the following properties:
- Radius
- Start point (`XYZ`)
- End point (`XYZ`)
Afaik, there aren't any functions that can create an arc using this information.
\*\*Answer:\*\* Technically speaking, 2 points and a radius are not enough to define a unique arc, as there are up to 4 possible arcs that will fit for any plane of reference. Test this yourself by drawing 2 circles of the same size that overlap each other and then draw a line between the 2 intersecting points. Each side of the line will contain 2 arcs that have the same end points and radius.
You can get away with it if you firstly adopt the convention that all arcs are to be drawn in the same direction from start to end (say, anticlockwise) when looking at them in the negative Z direction of the target plane, and secondly, that you always want either the arc with included angle of less than or equal to 180 degrees or the larger arc of over 180 degrees.
That aside, your best option is to calculate a point that lies on the arc and use the 3 points method that you have already used, but this time you will be giving it the correct 3rd point, and there will be no need to attempt changing the radius afterwards. We will use the anti-clockwise convention as that is what Revit also uses, we will also assume that the smaller arc is desired.
Here is a little bit of code I just whipped up for you; I have done some minimal testing and it appears to be working fine. I have assumed that you have the 3 following variables already defined and in scope:
- `XYZ end0`
- `XYZ end1`
- `Double radius`
I have also assumed that the arc lies on a plane with a normal vector of `XYZ.BasisZ`:
```csharp
Double sagitta = radius - Math.Sqrt(
Math.Pow( radius, 2.0 )
- Math.Pow( ( end1 - end0 ).GetLength()
/ 2.0, 2.0 ) );
XYZ midPointOfChord = ( end0 + end1 ) / 2.0;
XYZ midPointOfArc = midPointOfChord
+ Transform.CreateRotation( XYZ.BasisZ, Math.PI / 2.0 )
.OfVector( ( end0 - end1 ).Normalize()
.Multiply( sagitta ) );
Arc myArc = Arc.Create( end0, end1, midPointOfArc );
```
If you want the larger arc, simply change the sagitta calculation to be `radius +` instead of `radius -`.
If you aren't familiar with the mathematics used to calculate arcs, I would suggest heading over
to the [Math Open Reference sagitta explanation](http://www.mathopenref.com/sagitta.html).
No problems, I've just recently been doing some work on line-based families with arc location lines, so it was all fresh in my head.
I think you may have some issues achieving what you are after though; the solution I gave will only really work accurately for 2D data unless you also plug in the correct normal vector (axis) of each arc, basically everywhere that I have used `XYZ.BasisZ` would need to be replaced with the correct normal vector.
From the data you have it is not possible to determine the correct orientation of the arcs. You will need either a 3rd point on the arc provided for you (which would solve all your problems), or a normal vector of the target plane. Are you sure that there isn't any other information exported for arcs? If not, then the exporter isn't doing a very good job.
A possible solution then could be to see if you can first tessellate the arcs in the source software and then export just the straight line data. You could then rebuild the arcs from the lines as they will give you enough points to define a plane, but the trick would be knowing which lines belong to an arc and which don't. To solve this, you could export 2 sets of data, one with tessellation and one without and then match the end points found in the tessellated data against the other set to determine whether you need to build an arc or not.
Ok, I'll stop rambling now. Long story short: you need more data!
Edit: I just had the thought that if you have a mixture of arcs and lines and the lines are always tangent to the arcs, you could use the lines each side of an arc to calculate the correct plane.
If your data is actually 2D (null Z value), you should be fine unless you also encounter data with Z-values.
Many thanks to Scott for this nice solution and complete explanation!
I added it
to [The Building Coder samples](https://github.com/jeremytammik/the_building_coder_samples) as well, in C#,
in [release 2018.0.137.1](https://github.com/jeremytammik/the_building_coder_samples/releases/tag/2018.0.137.1),
[like this](https://github.com/jeremytammik/the_building_coder_samples/compare/2018.0.137.0...2018.0.137.1):
```csharp
///
/// Create an arc in the XY plane from a given
/// start point, end point and radius.
/// summary>
public static Arc CreateArc2dFromRadiusStartAndEndPoint(
XYZ ps,
XYZ pe,
double radius,
bool largeSagitta = false,
bool clockwise = false )
{
XYZ midPointChord = 0.5 \* ( ps + pe );
XYZ v = pe - ps;
double d = 0.5 \* v.GetLength(); // half chord length
// Small and large circle sagitta:
// http://www.mathopenref.com/sagitta.html
// https://en.wikipedia.org/wiki/Sagitta_(geometry)
double s = largeSagitta
? radius + Math.Sqrt( radius \* radius - d \* d ) // sagitta large
: radius - Math.Sqrt( radius \* radius - d \* d ); // sagitta small
XYZ midPointOffset = Transform
.CreateRotation( XYZ.BasisZ, 0.5 \* Math.PI )
.OfVector( v.Normalize().Multiply( s ) );
XYZ midPointArc = clockwise
? midPointChord + midPointOffset
: midPointChord - midPointOffset;
return Arc.Create( ps, pe, midPointArc );
}
```
![Sagitta](img/sagitta.png)