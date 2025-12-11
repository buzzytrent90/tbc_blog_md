---
ai_optimized: true
author: Jeremy Tammik
complexity_score: 5.7
content_type: qa
optimization_date: '2025-12-11T11:44:17.021561'
original_url: https://thebuildingcoder.typepad.com/blog/1909_paint_stair_col_beam.html
post_number: '1909'
reading_time_minutes: 12
series: structural
slug: paint_stair_col_beam
source_file: 1909_paint_stair_col_beam.md
tags:
- elements
- family
- filtering
- geometry
- levels
- references
- revit-api
- selection
- sheets
- transactions
- views
- walls
- structural
title: Paint Stair Col Beam
word_count: 2303
---

### Painting Stairs and Shooting for the Beams
Here are
two [Revit API discussion forum](http://forums.autodesk.com/t5/revit-api-forum/bd-p/160) threads
that I am currently quite involved in:
- [Painting stairs](#2)
- [Ray tracing vs bounding box to find beams intersecting columns](#3)
![Painted stair](img/painted_stair_in_seoul_south_korea.jpg "Painted stair")
#### Painting Stairs
A long-standing [Revit API discussion forum](http://forums.autodesk.com/t5/revit-api-forum/bd-p/160) thread on how
to [paint stair faces](https://forums.autodesk.com/t5/revit-api-forum/paint-stair-faces/m-p/10388359) was finally answered quite simply
by Bruce [@canyon.des](https://forums.autodesk.com/t5/user/viewprofilepage/user-id/10032309) Hans:
\*\*Question:\*\* Is there any reason why stair faces can be painted in the UI, and not in the API?
The question was raised in 2015 and apparently still exists:
I'm trying to paint some of the faces of a stair (a monolithic stair) through the Revit API.
It appears that it cannot be done programmatically – I get an error message saying that "the element faces cannot be painted" – even if I can achieve this stair face painting manually in the Revit UI.
Is there a way to automatically paint these faces?
\*\*Answer:\*\* For stairs, you need to use `GetStairsLandings` and `GetStairsRuns` to get the `ElementId` to paint landings or runs.
It's not intuitive but works.
Use the same to find out whether the landing or run faces are painted or not.
```csharp
void PaintStairs( UIDocument uidoc, Material mat )
{
Document doc = uidoc.Document;
Selection sel = uidoc.Selection;
//FaceSelectionFilter filter = new FaceSelectionFilter();
Reference pickedRef = sel.PickObject(
ObjectType.PointOnElement,
//filter,
"Please select a Face" );
Element elem = doc.GetElement( pickedRef );
GeometryObject geoObject = elem
.GetGeometryObjectFromReference( pickedRef );
Face fc = geoObject as Face;
if( elem.Category.Id.IntegerValue == -2000120 ) // Stairs
{
bool flag = false;
Stairs str = elem as Stairs;
ICollection landings = str.GetStairsLandings();
ICollection runs = str.GetStairsLandings();
using( Transaction transaction = new Transaction( doc ) )
{
transaction.Start( "Paint Material" );
foreach( ElementId id in landings )
{
doc.Paint( id, fc, mat.Id );
flag = true;
break;
}
if( !flag )
{
foreach( ElementId id in runs )
{
doc.Paint( id, fc, mat.Id );
break;
}
}
transaction.Commit();
}
}
}
```
Many thanks to Bruce for this simple and effective solution.
We have not finished discussing this issue yet, so please refer to the discussion thread for more updates.
#### Ray Tracing vs Location to Find Beams Intersecting Columns
Another recurring topic is how to find intersecting elements.
The thread
on [ray projection not picking up beams](https://forums.autodesk.com/t5/revit-api-forum/ray-projection-not-picking-up-beams/m-p/10388868) ends
up solved with two different approaches demonstrating possible ways to find beams intersecting columns using ray tracing versus the column location line, `Face.IsInside` and bounding box intersection:
\*\*Question:\*\* I'm trying to create a ray projection that finds the closest beam or slab from a column and attach it to the top to beam/slab found by the ray projection.
For some reason, I can't get it to find the beams I want it to attach too.
It only finds the slab.
Any Ideas?
This is what it looks like before I run my current code:
![Beams intersecting columns](img/find_beams_intersecting_column_01.png "Beams intersecting columns")
This is what it looks like afterwards; it only finds the slabs:
![Beams intersecting columns](img/find_beams_intersecting_column_02.png "Beams intersecting columns")
I want it to stop at the bottom of both beams and slabs.
Sooo... I tried to change the code to just pick up the beams:
This is what I have after that; circled in blue is what didn't attach to the beam above.
Some did attach:
![Beams intersecting columns](img/find_beams_intersecting_column_03.png "Beams intersecting columns")
So, why is this?
And, is there anything I can do to fix it.
Thanks ahead of time for any responses!
I think it has something to do with whether the column is centred under the beam, but I didn't think that would matter, because I'm using `FindReferenceTarget.All` in my ray projection.
Example of off-centre column not attaching:
![Beams intersecting columns](img/find_beams_intersecting_column_04.png "Beams intersecting columns")
\*\*Answer:\*\* It absolutely matters. The ray you shoot is an infinitely thin line, so you can easily miss something. You could try using five rays per pillar, e.g., one in the centre and one in each corner. I would suggest that you add some visual debugging code that represents part of your infinite shooting ray with a model line to visualise what is going on and whether a beam is hit or missed.
\*\*Question:\*\* I don't really understand, if the center of my column (which is where the ray is generated from) is within the bounds of the beam how does it not pick up the face of the beam? It seems to only pick up the centreline of the beam... Does `FindReferenceTarget.All` not find the face of beam? And if I add rays to the corners of the column I don't see how that would help if it only finds the beam when you hit the beam centreline straight on. I hope that makes sense. Any ideas?
\*\*Answer:\*\* Normally, the reference intersector is set up so that an infinitely thin ray is shot and all intersections with faces or edges are reported. Just as you say, hitting an edge or a centreline or any other infinitely thin object is infinitely improbable.
\*\*Question:\*\* What's a good way to do multiple rays for a single object?
I also just figured out why the ray bounce isn't working.
It wasn't working on columns that extended past the beam already.
That's because Revit already cuts out the column from the beam.
So, there is no face for it to hit.
It works if the columns are below the beam.
![Beams intersecting columns](img/find_beams_intersecting_column_05.png "Beams intersecting columns")
When I hide the column:
![Beams intersecting columns](img/find_beams_intersecting_column_06.png "Beams intersecting columns")
So, I think that your method of doing rays on the corners of the columns would help possibly pick up the intersecting edge.
I just don't know of a way to assign 5 rays to the column.
Can you point me in the correct direction?
I haven't seen anything on multiple rays per single object.
\*\*Answer:\*\* Simply calculate the four column bottom face corner points and the column centre line direction vector and use that data to define the four rays.
\*\*Question:\*\* lol... I wish it was "simply". Haha!
I have been trying to figure out how to get the bottom column corners with little success...
I have seen your blog about finding the bottom of walls and top of sloped walls but can't find anything on bottom of columns.
Also, once I get the points, where do I place them so it generates multiple rays?
\*\*Answer:\*\* There are ever so many different ways.
Maybe easier to iterate over the faces rather than the edges.
Either way is fine, though.
If you know that the cross section is rectangular and the column is vertical, you know that you have four bottom corners, and you can differentiate them from all other vertices simply by picking the four ones with minimal Z coordinates.
\*\*Answer 2:\*\* This task can actually be done without `ReferenceIntersector`
1) You can extract the Z extents of slabs/beams and columns from bounding box to compare level proximity.
2) You can use `Face.IsInside` to determine if a column location line is within the limits of the bottom face of slab or beam matching (1).
For slanted columns, you'll have to follow a point at base up the vector of the slant to see where point ends up at slab/beam underside.
Multiply `XY` components of vector by height difference between point at column base and underside of beam/slab, then add them to point at base.
Likely however that the `IsInside` will be affected by joins, as the `ReferenceIntersector` is.
You can probably identify a second capture group via bounding box intersection filters, i.e., the cases where the ray missing the faces due to join will be a cases where there is a bounding box intersection between such elements.
Could also use `JoinGeometry.UnjoinGeometry` between columns and slabs/beams prior to investigation.
Note that occasionally attaching column to underside will fail if column profile is not completely covered.
Often, you can partially cover a column and still get it to attach, but there is a limit on that.
\*\*Response:\*\* Thank you both for the responses.
I changed my approach to use bounding boxes like you suggest in Answer 2.
It seems to be working for me.
Here's what worked for me:
```csharp
void AdjustColumnHeightsUsingBoundingBox(
Document doc,
IList ids )
{
View view = doc.ActiveView;
int allColumns = 0;
int successColumns = 0;
if( view is View3D )
{
using( Transaction tx = new Transaction( doc ) )
{
tx.Start( "Adjust Column Heights" );
foreach( ElementId elemId in ids )
{
Element elem = doc.GetElement( elemId );
// Check if element is column
if( (BuiltInCategory) elem.Category.Id.IntegerValue
== BuiltInCategory.OST_StructuralColumns )
{
allColumns++;
FamilyInstance column = elem as FamilyInstance;
// Collect beams and slabs within bounding box
List builtInCats = new List();
builtInCats.Add( BuiltInCategory.OST_Floors );
builtInCats.Add( BuiltInCategory.OST_StructuralFraming );
ElementMulticategoryFilter beamSlabFilter
= new ElementMulticategoryFilter( builtInCats );
BoundingBoxXYZ bb = elem.get_BoundingBox( view );
Outline myOutLn = new Outline( bb.Min, bb.Max + 100 \* XYZ.BasisZ );
BoundingBoxIntersectsFilter bbFilter
= new BoundingBoxIntersectsFilter( myOutLn );
FilteredElementCollector collector
= new FilteredElementCollector( doc )
.WherePasses( beamSlabFilter )
.WherePasses( bbFilter );
List intersectingBeams = new List();
List intersectingSlabs = new List();
if( ColumnAttachment.GetColumnAttachment(
column, 1 ) != null )
{
// Change color of columns to green
Color color = new Color( (byte) 0, (byte) 255, (byte) 0 );
OverrideGraphicSettings ogs = new OverrideGraphicSettings();
ogs.SetProjectionLineColor( color );
view.SetElementOverrides( elem.Id, ogs );
}
else
{
foreach( Element e in collector )
{
if( e.Category.Name == "Structural Framing" )
{
intersectingBeams.Add( e );
}
else if( e.Category.Name == "Floors" )
{
intersectingSlabs.Add( e );
}
}
if( intersectingBeams.Any() )
{
Element lowestBottomElem = intersectingBeams.First();
foreach( Element beam in intersectingBeams )
{
BoundingBoxXYZ thisBeamBB = beam.get_BoundingBox( view );
BoundingBoxXYZ currentLowestBB = lowestBottomElem.get_BoundingBox( view );
if( thisBeamBB.Min.Z < currentLowestBB.Min.Z )
{
lowestBottomElem = beam;
}
}
ColumnAttachment.AddColumnAttachment(
doc, column, lowestBottomElem, 1,
ColumnAttachmentCutStyle.None,
ColumnAttachmentJustification.Minimum,
0 );
successColumns++;
}
else if( intersectingSlabs.Any() )
{
Element lowestBottomElem = intersectingSlabs.First();
foreach( Element slab in intersectingSlabs )
{
BoundingBoxXYZ thisSlabBB = slab.get_BoundingBox( view );
BoundingBoxXYZ currentLowestBB = lowestBottomElem.get_BoundingBox( view );
if( thisSlabBB.Min.Z < currentLowestBB.Min.Z )
{
lowestBottomElem = slab;
}
}
ColumnAttachment.AddColumnAttachment(
doc, column, lowestBottomElem, 1,
ColumnAttachmentCutStyle.None,
ColumnAttachmentJustification.Minimum,
0 );
successColumns++;
}
else
{
// Change color of columns to red
Color color = new Color( (byte) 255, (byte) 0, (byte) 0 );
OverrideGraphicSettings ogs = new OverrideGraphicSettings();
ogs.SetProjectionLineColor( color );
view.SetElementOverrides( elem.Id, ogs );
}
}
}
}
tx.Commit();
}
TaskDialog.Show( "Columns Changed",
string.Format( "{0} of {1} Columns Changed",
successColumns, allColumns ) );
}
else
{
TaskDialog.Show( "Revit", "Run Script in 3D View." );
}
}
```
I'd love to see an example of multiple rays per element if you ever decided to do a blog post about it.
\*\*Answer:\*\* There is nothing special about multiple rays per element at all.
In your code above, you shoot a ray upwards parallel to the Z axis from the element location point:
```csharp
// ray direction for raybounce
XYZ newPP = new XYZ( elemLoc.X, elemLoc.Y, elemLoc.Z + 1 );
XYZ rayd = new XYZ( 0, 0, 1 );
```
You can define any other source point you like, e.g., each of the four bottom corner points in turn, and also any other direction you like, and simply repeat the same process using the same reference intersector in the same view by repeatedly calling
its [Find method with the new source point and direction vector](https://www.revitapidocs.com/2021.1/6abd0586-5d7e-68c6-2e64-46199f457499.htm).
\*\*Response:\*\* I finally figured it out using the ray projection method as well.
Thanks to all your responses; I was super over-complicating it.
Thank you again for all your help.
Here is the working code with ray projection as well:
```csharp
void AdjustColumnHeightsUsingReferenceIntersector(
Document doc,
IList ids )
{
View3D view = doc.ActiveView as View3D;
if( null == view )
{
throw new Exception(
"Please run this command in a 3D view." );
}
int allColumns = 0;
int successColumns = 0;
using( Transaction tx = new Transaction( doc ) )
{
tx.Start( "Attach Columns Tops" );
foreach( ElementId elemId in ids )
{
Element elem = doc.GetElement( elemId );
if( (BuiltInCategory) elem.Category.Id.IntegerValue
== BuiltInCategory.OST_StructuralColumns )
{
allColumns++;
FamilyInstance column = elem as FamilyInstance;
// Collect beams and slabs
List builtInCats = new List();
builtInCats.Add( BuiltInCategory.OST_Floors );
builtInCats.Add( BuiltInCategory.OST_StructuralFraming );
ElementMulticategoryFilter filter
= new ElementMulticategoryFilter( builtInCats );
// Remove old column attachement
if( ColumnAttachment.GetColumnAttachment( column, 1 ) != null )
{
ColumnAttachment.RemoveColumnAttachment( column, 1 );
}
BoundingBoxXYZ elemBB = elem.get_BoundingBox( view );
XYZ elemLoc = (elem.Location as LocationPoint).Point;
XYZ elemCenter = new XYZ( elemLoc.X, elemLoc.Y, elemLoc.Z + 0.1 );
XYZ b1 = new XYZ( elemBB.Min.X, elemBB.Min.Y, elemBB.Min.Z + 0.1 );
XYZ b2 = new XYZ( elemBB.Max.X, elemBB.Max.Y, elemBB.Min.Z + 0.1 );
XYZ b3 = new XYZ( elemBB.Min.X, elemBB.Max.Y, elemBB.Min.Z + 0.1 );
XYZ b4 = new XYZ( elemBB.Max.X, elemBB.Min.Y, elemBB.Min.Z + 0.1 );
List points = new List( 5 );
points.Add( b1 );
points.Add( b2 );
points.Add( b3 );
points.Add( b4 );
points.Add( elemCenter );
ReferenceIntersector refI = new ReferenceIntersector(
filter, FindReferenceTarget.All, view );
XYZ rayd = XYZ.BasisZ;
ReferenceWithContext refC = null;
foreach( XYZ pt in points )
{
refC = refI.FindNearest( pt, rayd );
if( refC != null )
{
break;
}
}
if( refC != null )
{
Reference reference = refC.GetReference();
ElementId id = reference.ElementId;
Element e = doc.GetElement( id );
ColumnAttachment.AddColumnAttachment(
doc, column, e, 1,
ColumnAttachmentCutStyle.None,
ColumnAttachmentJustification.Minimum,
0 );
successColumns++;
}
else
{
// Change color of columns to red
Color color = new Color( (byte) 255, (byte) 0, (byte) 0 );
OverrideGraphicSettings ogs = new OverrideGraphicSettings();
ogs.SetProjectionLineColor( color );
view.SetElementOverrides( elem.Id, ogs );
}
}
}
tx.Commit();
}
}
```
\*\*Answer:\*\* Congratulations on simplifying and solving this.
[Keeping it simple](https://en.wikipedia.org/wiki/KISS_principle) works wonders, doesn't it?
Thank you for sharing the two approaches, and thanks
to Richard [RPThomas108](https://forums.autodesk.com/t5/user/viewprofilepage/user-id/1035859) Thomas
for the non-raytracing suggestion!