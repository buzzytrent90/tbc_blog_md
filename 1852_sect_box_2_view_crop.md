---
post_number: "1852"
title: "Sect Box 2 View Crop"
slug: "sect_box_2_view_crop"
author: "Jeremy Tammik"
tags: ['csharp', 'elements', 'filtering', 'geometry', 'levels', 'references', 'revit-api', 'rooms', 'selection', 'sheets', 'transactions', 'views', 'walls']
source_file: "1852_sect_box_2_view_crop.md"
original_url: "https://thebuildingcoder.typepad.com/blog/1852_sect_box_2_view_crop.html"
---

### Section to Crop, Linked Boundary and Intersection
Let's wrap up this week by highlighting two beautiful
[Revit API discussion forum](http://forums.autodesk.com/t5/revit-api-forum/bd-p/160) solutions by
Richard [RPThomas108](https://forums.autodesk.com/t5/user/viewprofilepage/user-id/1035859) Thomas,
who continues sharing new ones at an impressive pace:
- [Set view crop to section box](#2)
- [Room boundaries and intersections in linked models](#3)
Before diving into those, here is a very nice [freecodecamp](https://www.freecodecamp.org/) quote of the week:
> *People worry that computers will get too smart and take over the world.
The real problem is that computers are too stupid and they already have taken over the world – Pedro Domingos*
#### Set View Crop to Section Box
The main result for today is Richard's solution to
the [Revit API discussion forum](http://forums.autodesk.com/t5/revit-api-forum/bd-p/160) thread
on [setting view cropbox to a section box](https://forums.autodesk.com/t5/revit-api-forum/set-view-cropbox-to-a-section-box/m-p/9600049),
originally asked six years ago, in 2014, and now finally resolved:
\*\*Question:\*\* A simple question: how do I set a 3D view crop box to match (the outer corners of) a section box?
\*\*Answer:\*\* The Building Coder discussed how
to [set view section box to match scope box](http://thebuildingcoder.typepad.com/blog/2012/08/set-view-section-box-to-match-scope-box.html).
In this different situation, the two boxes have different coordinate systems, so one has to transformed into the other.
Also consider below that the maximum and minimum points of the section box are not enough alone to properly frame the section box with a crop box.
If you transform P1 and P2 below from the coordinate space of the section box into the coordinate space of the view, you would end up with the red box.
You need to know the missing corners; then you can transform all of the eight corner points into the coordinate space of the view.
You then find the max/min `XYZ` from those and set crop box to green rectangle:
![Set view crop to section box](img/rpt_set_view_crop_to_section_box_1.png "Set view crop to section box")
I wrote this extension method for that:
```csharp
Public Sub AdjustCropToSectionBox(View3D As View3D)
'View3D crop can't have a shape assigned and can't be split
If View3D.CropBoxActive = False Then
View3D.CropBoxActive = True
End If
If View3D.IsSectionBoxActive = False Then
Exit Sub
End If
Dim CropBox As BoundingBoxXYZ = View3D.CropBox
Dim SectionBox As BoundingBoxXYZ = View3D.GetSectionBox
Dim T As Transform = CropBox.Transform
Dim Corners As XYZ() = BBCorners(SectionBox, T)
Dim MinX As Double = Corners.Min(Function(j) j.X)
Dim MinY As Double = Corners.Min(Function(j) j.Y)
Dim MinZ As Double = Corners.Min(Function(j) j.Z)
Dim MaxX As Double = Corners.Max(Function(j) j.X)
Dim MaxY As Double = Corners.Max(Function(j) j.Y)
Dim MaxZ As Double = Corners.Max(Function(j) j.Z)
CropBox.Min = New XYZ(MinX, MinY, MinZ)
CropBox.Max = New XYZ(MaxX, MaxY, MaxZ)
View3D.CropBox = CropBox
End Sub
Private Function BBCorners(
SectionBox As BoundingBoxXYZ,
T As Transform) As XYZ()
Dim SBMx As XYZ = SectionBox.Max
Dim SBMn As XYZ = SectionBox.Min
Dim Btm_LL As XYZ = SBMn 'Lower Left
Dim Btm_LR As New XYZ(SBMx.X, SBMn.Y, SBMn.Z) 'Lower Right
Dim Btm_UL As New XYZ(SBMn.X, SBMx.Y, SBMn.Z) 'Upper Left
Dim Btm_UR As New XYZ(SBMx.X, SBMx.Y, SBMn.Z) 'Upper Right
Dim Top_UR As XYZ = SBMx 'Upper Right
Dim Top_UL As New XYZ(SBMn.X, SBMx.Y, SBMx.Z) 'Upper Left
Dim Top_LR As New XYZ(SBMx.X, SBMn.Y, SBMx.Z) 'Lower Right
Dim Top_LL As New XYZ(SBMn.X, SBMn.Y, SectionBox.Max.Z) 'Lower Left
Dim Out As XYZ() = New XYZ(7) {
Btm_LL, Btm_LR, Btm_UL, Btm_UR,
Top_UR, Top_UL, Top_LR, Top_LL}
For i = 0 To Out.Length - 1
'Transform bounding box corords to model coords
Out(i) = SectionBox.Transform.OfPoint(Out(i))
'Transform bounding box coords to view coords
Out(i) = T.Inverse.OfPoint(Out(i))
Next
Return Out
End Function
```
It is also worth considering that the extents of the section box may not be appropriate for how you wish to crop the view.
The section box below is relatively tight on the wall elements but due to the viewing angle there is a lot of spare space to the right.
When you print that on a sheet it will not be evident why the view content is off centre and what the extra space is for:
![Set view crop to section box](img/rpt_set_view_crop_to_section_box_2.png "Set view crop to section box")
\*\*Response:\*\* This solution worked great; I translated to C# and have posted it here for C# coders:
```csharp
public static void AdjustViewCropToSectionBox(
/\*this\*/ View3D view )
{
if( !view.IsSectionBoxActive )
{
return;
}
if( !view.CropBoxActive )
{
view.CropBoxActive = true;
}
BoundingBoxXYZ CropBox = view.CropBox;
BoundingBoxXYZ SectionBox = view.GetSectionBox();
Transform T = CropBox.Transform;
var Corners = BBCorners( SectionBox, T );
double MinX = Corners.Min( j => j.X );
double MinY = Corners.Min( j => j.Y );
double MinZ = Corners.Min( j => j.Z );
double MaxX = Corners.Max( j => j.X );
double MaxY = Corners.Max( j => j.Y );
double MaxZ = Corners.Max( j => j.Z );
CropBox.Min = new XYZ( MinX, MinY, MinZ );
CropBox.Max = new XYZ( MaxX, MaxY, MaxZ );
view.CropBox = CropBox;
}
private static XYZ[] BBCorners( BoundingBoxXYZ SectionBox, Transform T )
{
XYZ sbmn = SectionBox.Min;
XYZ sbmx = SectionBox.Max;
XYZ Btm_LL = sbmn; // Lower Left
var Btm_LR = new XYZ( sbmx.X, sbmn.Y, sbmn.Z ); // Lower Right
var Btm_UL = new XYZ( sbmn.X, sbmx.Y, sbmn.Z ); // Upper Left
var Btm_UR = new XYZ( sbmx.X, sbmx.Y, sbmn.Z ); // Upper Right
XYZ Top_UR = sbmx; // Upper Right
var Top_UL = new XYZ( sbmn.X, sbmx.Y, sbmx.Z ); // Upper Left
var Top_LR = new XYZ( sbmx.X, sbmn.Y, sbmx.Z ); // Lower Right
var Top_LL = new XYZ( sbmn.X, sbmn.Y, sbmx.Z ); // Lower Left
var Out = new XYZ[ 8 ] {
Btm_LL, Btm_LR, Btm_UL, Btm_UR,
Top_UR, Top_UL, Top_LR, Top_LL };
for( int i = 0, loopTo = Out.Length - 1; i <= loopTo; i++ )
{
// Transform bounding box coords to model coords
Out[ i ] = SectionBox.Transform.OfPoint( Out[ i ] );
// Transform bounding box coords to view coords
Out[ i ] = T.Inverse.OfPoint( Out[ i ] );
}
return Out;
}
```
Many thanks to Richard and Frank for the solution and the C# translation.
I added the new method `AdjustViewCropToSectionBox`
to [release 2021.0.149.2](https://github.com/jeremytammik/the_building_coder_samples/releases/tag/2021.0.149.2)
of [The Building Coder Samples](https://github.com/jeremytammik/the_building_coder_samples),
cf. the [diff to the previous version](https://github.com/jeremytammik/the_building_coder_samples/compare/2021.0.149.1...2021.0.149.2).
#### Room Boundaries and Intersections in Linked Models
Yet another beautiful solution by Richard to determine the column finish area within a room of columns residing partially inside and partially outside the room interior.
Trickier still, this includes handling of columns residing in linked files, copying the relevant elements into the main project file to retrieve the room boundaries there.
Note that there seem to be problems retrieving room boundaries from rooms in linked files, so this transformation process can come in handy in all sorts of similar situations.
Some of the recent issues that this solution helps resolve include:
- Intersection in linked models
- Transformation from linked into main host project and vice versa
- Room boundary retrieval does not work well with linked models
The question was raised in
the [Revit API discussion forum](http://forums.autodesk.com/t5/revit-api-forum/bd-p/160) thread
on [how to calculate the column finish area of room](https://forums.autodesk.com/t5/revit-api-forum/how-to-calculate-the-column-finish-area-of-room/m-p/9576200):
\*\*Question:\*\* We want to calculate the column finish area of columns in a room with columns present half inside the room and half outside; we have to ignore the outside finish area and only want the area which comes inside the room boundary.
This screen snapshot shows a typical situation:
![Column finish area](img/column_finish_area_1.png "Column finish area")
\*\*Answer:\*\* Here is one possible approach:
- Get column base curve.
- Get the room bounding segments and their curves.
- Get the intersections between column curves and the room curves.
- Calculate circumference of the column curve lying in the room boundary.
- Using room height calculate the area.
Possibly slightly more straightforward: the `BoundarySegment` class has an `ElementID` property that identifies the `Element` that generated the boundary segment.
If you review these in RevitLookup, you get the segment length measured to the centre of the wall.
This length however likely depends on how the `GetBoundarySegments` is called, i.e.:
Use `SpatialElementBoundaryOptions` `SpatialElementBoundaryLocation` `=` `Finish` instead of center; then you may get the curve length for each face which you can multiply with the room height.
Alternatively, if you get the lines only to the centre of the wall, you can get the wall thickness and subtract one half of it from the line length.
\*\*Response:\*\* We are trying to follow that approach to get the columns, but from the column edges we are not able to calculate the exact overlapping part with the room interior for the column part inside the room.
In our case, we don't know exactly how large a portion of the column is inside and how much is outside.
Also, the room is present in the active document and the column is in a linked document.
How can we get the column edges that come inside the room?
\*\*Answer:\*\* That is slightly more challenging, but the same principles as noted previously apply.
Summary of the code below:
- Get solid from room using `Room.ClosedShell`
- Transform solid to coord space of link containing columns using `SolidUtils.CreateTransformed`
- Find the columns intersecting the transformed solid using `ElementIntersectsSolidFilter`
- Copy each matching column back to the main doc using `ElementTransformUtils.CopyElements`
- Measure the boundaries
- Roll back the transaction
```vbnet
Private Function TObj75(ByVal commandData As Autodesk.Revit.UI.ExternalCommandData,
ByRef message As String, ByVal elements As Autodesk.Revit.DB.ElementSet) As Result
If commandData.Application.ActiveUIDocument Is Nothing Then Return Result.Cancelled Else
Dim UIDoc As UIDocument = commandData.Application.ActiveUIDocument
Dim Doc As Document = UIDoc.Document
Dim R As Reference = Nothing
Try
R = UIDoc.Selection.PickObject(Selection.ObjectType.Element, "Pick a room any room.")
Catch ex As Exception
End Try
If R Is Nothing Then Return Result.Cancelled Else
Dim Room As Room = TryCast(Doc.GetElement(R), Room)
If Room Is Nothing Then Return Result.Cancelled Else
Dim GeomEl As GeometryElement = Room.ClosedShell
Dim S As Solid = GeomEl(0) 'Assume single solid for brevity
Dim FEClnk As New FilteredElementCollector(Doc)
Dim ECFlnk As New ElementClassFilter(GetType(RevitLinkInstance))
Dim Lnk As List(Of RevitLinkInstance) = FEClnk.WherePasses(ECFlnk).ToElements.Cast(Of RevitLinkInstance).ToList
Dim LinkDoc As Document = Lnk(0).GetLinkDocument() 'Assume single link containing columns for brevity
Dim SolTransformed As Solid = SolidUtils.CreateTransformed(S, Lnk(0).GetTransform.Inverse)
'The solid in the coord system of the link
Dim ElintS As New ElementIntersectsSolidFilter(SolTransformed)
Dim ECF As New ElementCategoryFilter(BuiltInCategory.OST_StructuralColumns)
Dim LandF As New LogicalAndFilter(ElintS, ECF)
Dim FEC As New FilteredElementCollector(LinkDoc)
Dim Els As List(Of ElementId) = FEC.WherePasses(LandF).ToElementIds
Using tx As New Transaction(Doc, "Copy")
If tx.Start = TransactionStatus.Started Then
Dim NewIDs As List(Of ElementId) = ElementTransformUtils.CopyElements(LinkDoc, Els, Doc, Lnk(0).GetTransform, Nothing)
Dim Ops As New SpatialElementBoundaryOptions() With {.SpatialElementBoundaryLocation = SpatialElementBoundaryLocation.Finish}
Dim BoundSegs As IList(Of IList(Of BoundarySegment)) = Room.GetBoundarySegments(Ops)
For i = 0 To BoundSegs.Count - 1
Dim SegLst As IList(Of BoundarySegment) = BoundSegs(i)
Debug.WriteLine("List: " & CStr(i + 1)) 'List 0' just doesn't sound right
For ix = 0 To SegLst.Count - 1
Dim Seg As BoundarySegment = SegLst(ix)
If NewIDs.Contains(Seg.ElementId) = False Then Continue For Else
Debug.WriteLine(CStr(Seg.ElementId.IntegerValue) & ", " & (Seg.GetCurve.Length \* 304.8).ToString("F1"))
'Tried below to see what .LinkedElementId represented (not apparent)
'Debug.WriteLine(CStr(Seg.LinkElementId.IntegerValue) & "," & Seg.GetCurve.Length & "FromLink")
Next
Next
tx.RollBack()
End If
End Using
End Function
```
Proof of concept:
![Column finish area](img/column_finish_area_2.png "Column finish area")
Results:

```
  List: 1
  427532, 400.0
  427532, 750.0
  427532, 400.0
  427536, 275.0
  427536, 200.0
  427534, 200.0
  427534, 750.0
  427534, 200.0
```

This technique was first developed in the user interface.
As Revit users, we had long applied this technique in the UI, i.e., tab into a linked element, copy it to the clipboard and paste it to the same place within the host document.
The `ElementTransformUtils` is definitely a powerful feature of the API which sometimes gets overlooked in terms of how it can be applied to solve problems.
By the way, this solution simultaneously answers a number of other recent forum issues involving intersections and room boundaries in linked documents, including a previous thread
on [how to find the rooms geometry adjacent to walls](https://forums.autodesk.com/t5/revit-api-forum/how-to-find-the-rooms-geometry-adjacent-to-walls-in-revit-api/m-p/9576146),
where an analogous approach can be applied.
It also shows once again how important and useful it is to fully develop and optimise a solution manually in the user interface before beginning to address it programmatically.
Many thanks again to Richard for these nice solutions.
You are helping elevate to Revit developer community to a new level!