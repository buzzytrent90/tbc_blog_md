---
post_number: "1592"
title: "Disjunct Outer Loops"
slug: "disjunct_outer_loops"
author: "Jeremy Tammik"
tags: ['csharp', 'doors', 'elements', 'geometry', 'references', 'revit-api', 'rooms', 'selection', 'sheets', 'transactions', 'views', 'walls', 'windows']
source_file: "1592_disjunct_outer_loops.md"
original_url: "https://thebuildingcoder.typepad.com/blog/1592_disjunct_outer_loops.html"
---

### Disjunct Planar Face Outer Loops
I am attending the Forge DevCon conference in Darmstadt, Germany, right now, and putting the final touches to my presentation on \*Rational BIM programming using Revit and Forge\* at the European Autodesk University on Wednesday.
We discussed several different approaches to retrieve the outer loop of a planar face.
One useful piece of related functionality was provided early on by
the [point in polygon algorithm](http://thebuildingcoder.typepad.com/blog/2010/12/point-in-polygon-containment-algorithm.html).
Previous discussions of this topic include:
- [Determining Wall Opening Areas per Room](http://thebuildingcoder.typepad.com/blog/2016/04/determining-wall-opening-areas-per-room.html)
- [The `ExporterIFCUtils` method `GetInstanceCutoutFromWall` returns the outer CurveLoop of a window or a door](http://thebuildingcoder.typepad.com/blog/2017/06/copy-local-false-and-ifc-utils-for-wall-openings.html)
Recently, it also turned to the additional complexity of retrieving the multiple outer loops required for disjunct planar faces, raised in
the [Revit API discussion forum](http://forums.autodesk.com/t5/revit-api-forum/bd-p/160) thread
on whether [the first edge loop is still the outer loop](https://forums.autodesk.com/t5/revit-api-forum/is-the-first-edgeloop-still-the-outer-loop/m-p/7225379),
for which Richard [@rpthomas108](https://forums.autodesk.com/t5/user/viewprofilepage/user-id/1035859) Thomas
shared a possible solution:
- [Determining the outer-most EdgeLoop](http://thebuildingcoder.typepad.com/blog/2017/08/edge-loop-point-reference-plane-and-column-line.html)
He now posted a new approach to this question in
the [Revit API discussion forum](http://forums.autodesk.com/t5/revit-api-forum/bd-p/160) thread
on [outer loops of planar face with separate parts](https://forums.autodesk.com/t5/revit-api-forum/outer-loops-of-planar-face-with-separate-parts/m-p/7461348):
Just wanted to report that I've found a more straightforward and likely reliable way of getting outer loops of planar faces (than previous discussed). This method also allows for faces made up of disjointed parts.
The approach is to create some undocumented solid extrusions using GeometryCreationUtilities based on curve loop parts of original face. Then extract the parallel top face from these (now separated out) and use powerful built in functionality of the `PlanarFace` class (`PlanarFace.IsInside`) to check for loops with points not inside other faces. We only have to check one point because curve loops can't be self-intersecting.
It would be nice if we could create `PlanarFace` elements directly with the `GeometryCreationUtilities` (to cut out some of the above steps), but that does not seem possible yet. I've created an idea entry here for this:
[API GeometryCreationUtilities to create faces](https://forums.autodesk.com/t5/revit-ideas/api-geometrycreationutilities-to-create-faces/idi-p/7461357).
I've tested this solution on some interesting slab shapes shown below and results seem reliable.
![Disjunct outer loops](img/disjunct_outer_loops.png)
#### VB Implementation
```vbnet
Public Function GetPlanarFaceOuterLoops(
ByVal commandData As Autodesk.Revit.UI.ExternalCommandData,
ByRef message As String,
ByVal elements As Autodesk.Revit.DB.ElementSet) As Result
Dim IntApp As UIApplication = commandData.Application
Dim IntUIDoc As UIDocument = IntApp.ActiveUIDocument
If IntUIDoc Is Nothing Then Return Result.Failed Else
Dim IntDoc As Document = IntUIDoc.Document
Dim R As Reference = Nothing
Try
R = IntUIDoc.Selection.PickObject(Selection.ObjectType.Face)
Catch ex As Exception
End Try
If R Is Nothing Then Return Result.Cancelled Else
Dim F_El As Element = IntDoc.GetElement(R.ElementId)
If F_El Is Nothing Then Return Result.Failed Else
Dim F As PlanarFace = TryCast(
F_El.GetGeometryObjectFromReference(R), PlanarFace)
If F Is Nothing Then Return Result.Failed Else
'Create individual CurveLoops to compare
' From the orginal CurveLoopArray
'If floor has separate parts these will now be
' separated out into individual faces rather
' than one face with multiple loops.
Dim CLoop As New List(Of Tuple(Of PlanarFace, CurveLoop, Integer))
Dim Ix As Integer = 0
For Each item As CurveLoop In F.GetEdgesAsCurveLoops
Dim CLL As New List(Of CurveLoop)
CLL.Add(item)
'Create a solid extrusion for each CurveLoop
' (we want to get the planarFace from this to
' use built in functionality (.PlanarFace.IsInside).
'Would be nice if you could skip this step and create
' PlanarFaces directly from CuveLoops? Does no appear
' To be possible, I only looked in GeometryCreationUtilities.
'Below creates geometry in memory rather than actual
' Geometry in the document, therefore no transaction required.
Dim S As Solid = GeometryCreationUtilities _
.CreateExtrusionGeometry(CLL, F.FaceNormal, 1)
For Each Fx As Face In S.Faces
Dim PFx As PlanarFace = TryCast(Fx, PlanarFace)
If PFx Is Nothing Then Continue For Else
If PFx.FaceNormal.IsAlmostEqualTo(F.FaceNormal) Then
Ix += 1
CLoop.Add(New Tuple(Of PlanarFace, CurveLoop,
Integer)(PFx, item, Ix))
End If
Next
Next
Dim FirstPointIsInsideFace = Function(
CL As CurveLoop, PFace As PlanarFace) _
As Boolean
Dim Trans As Transform = PFace.ComputeDerivatives(New UV(0, 0))
If CL.Count = 0 Then Return False Else
Dim Pt As XYZ = Trans.Inverse.OfPoint(CL(0).GetEndPoint(0))
Dim Res As IntersectionResult = Nothing
Dim out As Boolean = PFace.IsInside(New UV(Pt.X, Pt.Y), Res)
Return out
End Function
Dim OuterLoops As New List(Of CurveLoop)
'If there is more than one outerloop we know
' the original face has separate parts.
'We could therefore stop the creation of floors
' With separate parts via posting failures etc.
' Or more passively create a geometry checking
' utility To identify them.
Dim InnerLoops As New List(Of CurveLoop)
For Each item As Tuple(Of PlanarFace, CurveLoop, Integer) In CLoop
'To identify an inner loop we just need to see
' If any Then Of it's points are inside another face.
'The exception to this is a loop compared to the
' Face it was taken from. This will also be considered
' inside as the points are on the boundary.
'Therefore give each item an integer ID to ensure it
' isn 't self comparing. An alternative would be to
' look for J=1 instead of J=0 below (perhaps).
Dim J As Integer = CLoop.ToList.FindAll(
Function(z) FirstPointIsInsideFace(item.Item2, z.Item1) _
= True AndAlso z.Item3 <> item.Item3).Count
If J = 0 Then
OuterLoops.Add(item.Item2)
Else
InnerLoops.Add(item.Item2)
End If
Next
Using Tx As New Transaction(IntDoc, "Outer loops")
If Tx.Start = TransactionStatus.Started Then
Dim SKP As SketchPlane = SketchPlane.Create(
IntDoc,
Plane.CreateByThreePoints(F.Origin, F.Origin + F.XVector,
F.Origin + F.YVector))
For Each Crv As CurveLoop In OuterLoops
For Each C As Curve In Crv
IntDoc.Create.NewModelCurve(C, SKP)
Next
Next
Tx.Commit()
End If
End Using
Return Result.Succeeded
End Function
```
#### C# Implementation
```csharp
public Result GetPlanarFaceOuterLoops(
Autodesk.Revit.UI.ExternalCommandData commandData,
ref string message,
Autodesk.Revit.DB.ElementSet elements )
{
UIApplication IntApp = commandData.Application;
UIDocument IntUIDoc = IntApp.ActiveUIDocument;
if( IntUIDoc == null )
return Result.Failed;
Document IntDoc = IntUIDoc.Document;
Reference R = null;
try
{
R = IntUIDoc.Selection.PickObject( ObjectType.Face );
}
catch
{
}
if( R == null )
return Result.Cancelled;
Element F_El = IntDoc.GetElement( R.ElementId );
if( F_El == null )
return Result.Failed;
PlanarFace F = F_El.GetGeometryObjectFromReference( R )
as PlanarFace;
if( F == null )
return Result.Failed;
//Create individual CurveLoops to compare from
// the orginal CurveLoopArray
//If floor has separate parts these will now be
// separated out into individual faces rather
// than one face with multiple loops.
List> CLoop
= new List>();
int Ix = 0;
foreach( CurveLoop item in F.GetEdgesAsCurveLoops() )
{
List CLL = new List();
CLL.Add( item );
//Create a solid extrusion for each CurveLoop
// ( we want to get the planarFace from this
// to use built in functionality (.PlanarFace.IsInside).
//Would be nice if you could skip this step and
// create PlanarFaces directly from CuveLoops?
// Does not appear to be possible, I only looked
// in GeometryCreationUtilities.
//Below creates geometry in memory rather than
// actual geometry in the document, therefore
// no transaction required.
Solid S = GeometryCreationUtilities
.CreateExtrusionGeometry( CLL, F.FaceNormal, 1 );
foreach( Face Fx in S.Faces )
{
PlanarFace PFx = Fx as PlanarFace;
if( PFx == null )
continue;
if( PFx.FaceNormal.IsAlmostEqualTo(
F.FaceNormal ) )
{
Ix += 1;
CLoop.Add( new Tuple( PFx, item, Ix ) );
}
}
}
List OuterLoops = new List();
//If there is more than one outerloop we know the
// original face has separate parts.
//We could therefore stop the creation of floors
// with separate parts via posting failures etc.
// or more passively create a geometry checking
// utility to identify them.
List InnerLoops = new List();
foreach( Tuple item in CLoop )
{
//To identify an inner loop we just need to see
// if any of it's points are inside another face.
//The exception to this is a loop compared to the
// face it was taken from. This will also be
// considered inside as the points are on the boundary.
//Therefore give each item an integer ID to ensure
// it isn't self comparing. An alternative would
// be to look for J=1 instead of J=0 below (perhaps).
int J = CLoop.ToList().FindAll( z
=> FirstPointIsInsideFace( item.Item2, z.Item1 )
== true && z.Item3 != item.Item3 ).Count;
if( J == 0 )
{
OuterLoops.Add( item.Item2 );
}
else
{
InnerLoops.Add( item.Item2 );
}
}
using( Transaction Tx = new Transaction( IntDoc,
"Outer loops" ) )
{
if( Tx.Start() == TransactionStatus.Started )
{
SketchPlane SKP = SketchPlane.Create( IntDoc,
Plane.CreateByThreePoints( F.Origin,
F.Origin + F.XVector, F.Origin + F.YVector ) );
foreach( CurveLoop Crv in OuterLoops )
{
foreach( Curve C in Crv )
{
IntDoc.Create.NewModelCurve( C, SKP );
}
}
Tx.Commit();
}
}
return Result.Succeeded;
}
public bool FirstPointIsInsideFace(
CurveLoop CL,
PlanarFace PFace )
{
Transform Trans = PFace.ComputeDerivatives(
new UV( 0, 0 ) );
if( CL.Count() == 0 )
return false;
XYZ Pt = Trans.Inverse.OfPoint(
CL.ToList()[0].GetEndPoint( 0 ) );
IntersectionResult Res = null;
bool outval = PFace.IsInside(
new UV( Pt.X, Pt.Y ), out Res );
return outval;
}
```
I added the latter
to [The Building Coder samples release 2018.0.134.4](https://github.com/jeremytammik/the_building_coder_samples/releases/tag/2018.0.134.4)
module [CmdSlabBoundaryArea.cs L29-L176](https://github.com/jeremytammik/the_building_coder_samples/blob/master/BuildingCoder/BuildingCoder/CmdSlabBoundaryArea.cs#L29-L176).
Many thanks to Richard for implementing, testing and sharing this improved solution!