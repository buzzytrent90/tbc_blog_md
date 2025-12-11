---
ai_optimized: true
author: Jeremy Tammik
complexity_score: 10.4
content_type: qa
optimization_date: '2025-12-11T11:44:16.518205'
original_url: https://thebuildingcoder.typepad.com/blog/1684_rebar_dimension.html
post_number: '1684'
reading_time_minutes: 12
series: general
slug: rebar_dimension
source_file: 1684_rebar_dimension.md
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
title: Rebar Dimension
word_count: 2428
---

### Rebar, Wall Centreline, Core and Grid Dimensioning
My colleague Zhong John Wu just solved
a [Revit API discussion forum](http://forums.autodesk.com/t5/revit-api-forum/bd-p/160) issue
on how to [create dimension line for rebar](https://forums.autodesk.com/t5/revit-api-forum/create-dimension-line-for-rebar/m-p/8217536).
I took this as a prompt to clean out a bunch of other dimensioning related issues lurking in my infinite and growing to-do list:
- [Create dimension line for rebar](#2)
- [Newly created dimensioning not displayed](#3)
- [Linear reference from surface filtering for all](#3b)
- [Dimension leader remains visible after removal](#4)
- [Dimension wall centreline, centre and faces of core](#5)
- [Grid references for dimensioning](#6)
- [Simple, better grid references for dimensioning](#7)
#### Create Dimension Line for Rebar
\*\*Question:\*\* I'm struggling to create a rebar dimension line because I can't find a way to get the edges of the element in a section view:
![Rebar dimensioning](img/rebar_dimension_1.png)
I can retrieve their edges through their geometry, but the edge I need doesn't have a reference that I can use.
My ultimate goal is to measure the distance from the end of the bar to a level or grid:
![Rebar dimensioning](img/rebar_dimension_2.png)
\*\*Answer:\*\* Similar questions were raised here in the past, to:
- [Dimension rebars](https://forums.autodesk.com/t5/revit-api-forum/dimension-rebars/td-p/5356233), and
- [Create aligned dimension between rebars](https://forums.autodesk.com/t5/revit-api-forum/create-aligned-dimension-between-rebars/m-p/7083248)
The solutions back then include reading the element geometry and the references it provides, just like you describe:
> You can read the geometry data from the rebar by `Rebar.Geometry` property. This property requires an `Option` argument. You need to set the `option.ComputeReferences` to true. Then read the edge of the rebar, and get the curve from the `Edge` object. Finally, get the end point reference from the curve.
\*\*Response:\*\* I already looked at these posts but with no results. When I create the dimension line, a reference is needed. The approach you describe returns a null reference for the edge.
Here are 4 different paths I attempted, with no desired result so far:
PLAN A:
```csharp
ReferenceArray ra = new ReferenceArray();
Line dimension = Line.CreateBound(rebar_top, apoyo_top);
DetailLine line1 = doc.Create.NewDetailCurve(view, dimension) as DetailLine;
ra.Append(line1.GeometryCurve.GetEndPointReference(1));
ra.Append(line1.GeometryCurve.GetEndPointReference(0));
Dimension dim = doc.Create.NewDimension(doc.ActiveView, dimension, ra);
```
PLAN B:
```csharp
XYZ apoyo_top = pAnalisisSupCap + rle.cm_to_ft(200) \* XYZ.BasisZ;
XYZ apoyo_bot = pAnalisisSupCap - rle.cm_to_ft(200) \* XYZ.BasisZ;
XYZ rebar_top = pini + rle.cm_to_ft(200) \* XYZ.BasisZ;
XYZ rebar_bot = pini - rle.cm_to_ft(200) \* XYZ.BasisZ;
Line l_v = Line.CreateBound(apoyo_bot, apoyo_top);
Line l_h = Line.CreateBound(rebar_bot, rebar_top);
Plane p_h = Plane.CreateByNormalAndOrigin(rebar_bot
.CrossProduct(rebar_top), rebar_top);
SketchPlane skplane_h = SketchPlane.Create(doc, p_h);
Plane p_v = Plane.CreateByNormalAndOrigin(apoyo_bot
.CrossProduct(apoyo_top), apoyo_top);
SketchPlane skplane_v = SketchPlane.Create(doc, p_v);
ModelCurve modelcurve1 = doc.Create.NewModelCurve (l_h, skplane_h);
ModelCurve modelcurve2 = doc.Create.NewModelCurve(l_v, skplane_v);
ra.Append(modelcurve1.GeometryCurve.Reference);
ra.Append(modelcurve2.GeometryCurve.Reference);
ra.Append(modelcurve1.GeometryCurve.GetEndPointReference(0));
ra.Append(modelcurve2.GeometryCurve.GetEndPointReference(0));
```
PLAN C:
```csharp
Options opt = new Options();
opt.ComputeReferences = true;
opt.View = doc.ActiveView;
opt.IncludeNonVisibleObjects = true;
GeometryElement geomElem = rebInt.get_Geometry(opt);
foreach (GeometryObject geomObj in geomElem)
{
Solid geomSolid = geomObj as Solid;
if (null != geomSolid)
{
int faces = 0;
double totalArea = 0;
foreach (Face geomFace in geomSolid.Faces)
{
faces++;
faceInfo += "Face " + faces + " area: "
+ geomFace.Area.ToString() + "\n";
totalArea += geomFace.Area;
info = geomFace;
}
faceInfo += "Number of faces: " + faces + "\n";
faceInfo += "Total area: " + totalArea.ToString() + "\n";
int edge = 0;
foreach (Edge geomEdge in geomSolid.Edges)
{
geoobj = geomEdge.AsCurve();
}
}
}
// No Faces/Edges valiuds as references.
```
PLAN D:
```csharp
IList dd = rebInt
.GetRebarConstraintsManager().GetAllHandles();
RebarConstrainedHandle s = null;
RebarConstrainedHandle e = null;
foreach (RebarConstrainedHandle rbh in dd)
{
if (rbh.GetHandleName().ToString() == "Start of Bar")
{
s = rbh;
}
if (rbh.GetHandleName().ToString() == "End of Bar")
{
e = rbh;
}
}
// This objects (handles) doesn't give references.
```
\*\*Answer:\*\* I verified that your plan A throws an exception saying, 'The direction of dimension is invalid'.
This is because the input value `dimension` should lie inside the plan of `doc.ActiveView`.
So, I changed the input value dimension of the following code line to `line1.GeometryCurve as Line`.
It works, but you just need to offset Z of the dimension line to make it looks good:
```csharp
Dimension dim = doc.Create.NewDimension(
doc.ActiveView, line1.GeometryCurve as Line, ra );
```
\*\*Answer and Solution:\*\* With the great support from our Revit engineering team, I got the ideal solution and verified that it works fine to dimension between rebar and grid.
The code contains a command that places a dimension in your model. You will need to select the rebar. The grid line is hard-coded by its element id.
It implements the following steps:
- Get all rebar references.
- Filter them to choose the one that we need.
- Get the grid reference.
- With the rebar ref and grid references, create the dimension.
If you want to only dimension the rebar, you just need to change the code to get the two references of the edges perpendicular to `rebarSeg`:
```csharp
using System;
using System.Collections.Generic;
using Autodesk.Revit.DB;
using Autodesk.Revit.UI;
using Autodesk.Revit.UI.Selection;
using Autodesk.Revit.Attributes;
using Autodesk.Revit.DB.Structure;
namespace TestRebar
{
[TransactionAttribute( TransactionMode.Manual )]
public class TestRebar : IExternalCommand
{
UIApplication m_uiApp;
Document m_doc;
ElementId elementId = ElementId.InvalidElementId;
public Result Execute(
ExternalCommandData commandData,
ref string message,
ElementSet elements )
{
try
{
initStuff( commandData );
if( m_doc == null )
return Result.Failed;
m_uiApp = commandData.Application;
Selection sel = m_uiApp.ActiveUIDocument.Selection;
Reference refr = sel.PickObject( ObjectType.Element );
Rebar rebar = m_doc.GetElement( refr.ElementId ) as Rebar;
Line rebarSeg = null;
bool bOk = getRebarSegment( rebar, out rebarSeg );
if( !bOk )
return Result.Failed;
Options options = new Options();
// the view in which you want to place the dimension
options.View = m_uiApp.ActiveUIDocument.ActiveView;
options.ComputeReferences = true; // produce references
options.IncludeNonVisibleObjects = true;
GeometryElement wholeRebarGeometry
= rebar.get_Geometry( options );
Reference refForEndOfBar = getReferenceForEndOfBar(
wholeRebarGeometry, rebarSeg );
Reference refGrid = getGridRef();
ReferenceArray refArray = new ReferenceArray();
refArray.Append( refForEndOfBar );
refArray.Append( refGrid );
double dist = 10;
// a line parallel with our rebar segment somewhere in space
Line dimLine = rebarSeg.CreateOffset(
dist, XYZ.BasisY ) as Line;
using( Transaction tr = new Transaction( m_doc ) )
{
tr.Start( "Create Dimension" );
m_doc.Create.NewDimension(
m_uiApp.ActiveUIDocument.ActiveView,
dimLine, refArray );
tr.Commit();
}
}
catch( Exception e )
{
TaskDialog.Show( "exception", e.Message );
return Result.Failed;
}
return Result.Succeeded;
}
private Reference getGridRef()
{
ElementId idGrd = new ElementId( 397028 );
Element elemGrid = m_doc.GetElement( idGrd );
Options options = new Options();
// the view in which you want to place the dimension
options.View = m_uiApp.ActiveUIDocument.ActiveView;
options.ComputeReferences = true; // produce references
options.IncludeNonVisibleObjects = true;
GeometryElement wholeGridGeometry
= elemGrid.get_Geometry( options );
IList allRefs = new List();
foreach( GeometryObject geomObj in wholeGridGeometry )
{
Line refLine = geomObj as Line;
if( refLine != null && refLine.Reference != null )
return refLine.Reference;
}
return null;
}
private Reference getReferenceForEndOfBar(
GeometryElement geom,
Line rebarSeg )
{
foreach( GeometryObject geomObj in geom )
{
Solid sld = geomObj as Solid;
if( sld != null )
{
// I'll get the references from curves;
continue;
}
else
{
Line refLine = geomObj as Line;
if( refLine != null && refLine.Reference != null )
{
// We found one reference.
// Let's see if it is the correct one.
// The correct referece need to be perpendicular
// to rebar segement and the end point of rebar
// segment should be on the reference curve.
double dotProd = refLine.Direction.DotProduct(
rebarSeg.Direction );
if( Math.Abs( dotProd ) != 0 )
continue; // curves are not perpendicular.
XYZ endPointOfRebar = rebarSeg.GetEndPoint( 1 );
IntersectionResult ir = refLine.Project(
endPointOfRebar );
if( ir == null )
continue; // end point of rebar segment is not on the reference curve.
if( Math.Abs( ir.Distance ) != 0 )
continue; // end point of rebar segment is not on the reference curve.
return refLine.Reference;
}
}
}
return null;
}
private bool getRebarSegment(
Rebar rebar,
out Line rebarSeg )
{
rebarSeg = null;
IList rebarSegments
= rebar.GetCenterlineCurves( false, true, true,
MultiplanarOption.IncludeOnlyPlanarCurves, 0 );
if( rebarSegments.Count != 1 )
return false;
rebarSeg = rebarSegments[0] as Line;
if( rebarSeg == null )
return false;
return true;
}
void initStuff( ExternalCommandData commandData )
{
m_uiApp = commandData.Application;
m_doc = m_uiApp.ActiveUIDocument.Document;
}
}
}
```
\*\*Response:\*\* Thanks a lot for the response. It is the solution to our problem! We made little adjustments to the code to have plenty of control about what end do we want to dimension, and also we had to project all the curves involved into a shared reference plane due to the fact that if you have more than one rebar (rebar set), the geometry of the rebar is above or beneath the grid line.
Again, thanks a lot for your time and effort.
This thanks is due to Zhong (John) Wu, Boris Shafiro and especially Stefan Dobre.
#### Newly Created Dimensioning Not Displayed
For another issue or two, on newly created dimensioning not being displayed properly,
Frank [@Fair59](https://forums.autodesk.com/t5/user/viewprofilepage/user-id/2083518) Aarssen
once again comes to the rescue:
- [Copy dimensions from a View to another](https://forums.autodesk.com/t5/revit-api-forum/copy-dimensions-from-a-view-to-another/m-p/7226217)
- [NewDimension - dimension created but not visible](https://forums.autodesk.com/t5/revit-api-forum/newdimension-dimension-created-but-not-visible/m-p/6340985)
Fair59 says: The workaround for dimensions to model elements, is rather simple. Create a new Dimension, using the References from the non-visible dimension.
#### Linear Reference from Surface Filtering for All
In yet another visibility question,
on [new dimensions are not visible](https://forums.autodesk.com/t5/revit-api-forum/new-dimension-are-not-visible/m-p/8268048),
Unknowz describes another interesting approach:
\*\*Question:\*\*
I continued to check my code and found an interesting point which is probably at the origin of the problem.
I use this to obtain a reference to `theBiggestFace`:
```csharp
PlanarFace planarFace = theBiggestFace as PlanarFace;
Reference refer = planarFace.Reference;
```
This returns a `Reference` whose `ElementReferenceType` is `REFERENCE_TYPE_SURFACE`.
However, in my case (with the faces parallel to the view), the dimensioning will only work with `REFERENCE_TYPE_LINEAR`.
![Get linear reference from surface](img/get_linear_reference_from_surface.png)
Do you have an idea of how to obtain a `Reference` with a type `Linear` from a `Surface` `Reference`?
It's probably a simple additional line at my existing code, but I couldn't find it yet...
\*\*Solution:\*\*
Yes and No – I couldn’t properly find the reference line from the face, so now I do it another way.
I catch all the geometry objects from the view, both visible and invisible, and filter all these elements to find:
- A reference line parallel to the middle axis of the face
- The closest reference to the middle axis
It seems like a dirty solution, but it works, and, with good filtering, the performance stays acceptable.
#### Dimension Leader Remains Visible After Removal
Another dimensioning display issue that came up
involves [removing dimension leader not visible – only after reselection or reopening](https://forums.autodesk.com/t5/revit-api-forum/removing-dimension-leader-not-visible-only-after-reselection-or/m-p/7545212).
That was resolved by moving an element, as suggested in the bunch of examples illustrating
the [need to regenerate and various ways to achieve that](http://thebuildingcoder.typepad.com/blog/about-the-author.html#5.33):
> `doc.regenerate` and `uidoc.RefreshActiveView` do not seem to do the trick.
> Your suggestion about moving the object is a good one.
> Moving the dimension up and down within the same transaction (no need for two transactions) regenerates the dimension and removes the leaders.
```csharp
ElementTransformUtils.MoveElement( doc, dimRef.Id, XYZ.BasisZ );
ElementTransformUtils.MoveElement( doc, dimRef.Id, -XYZ.BasisZ );
```
#### Dimension Wall Centreline, Centre and Faces of Core
A frequent requirement is
to [create dimension to wall centerline, center of core, faces of core](https://forums.autodesk.com/t5/revit-api-forum/create-dimension-to-wall-centerline-center-of-core-faces-of-core/m-p/5577704)
A lot of powerful and useful functionality addressing this was added in the Revit 2018 API, e.g.:
- [References and selection of subelements](http://thebuildingcoder.typepad.com/blog/2017/04/whats-new-in-the-revit-2018-api.html#2.2.3)
- [API access to FamilyInstance references](http://thebuildingcoder.typepad.com/blog/2017/04/whats-new-in-the-revit-2018-api.html#3.19).
Fair59 added another solution to pinpoint specific core layers, e.g.:
- Overall Centre
- Core Exterior Face
- Core Interior Face
- Core Centre
- Exterior Wall Face
- Interior Wall Face
> You can analyse the references of a dimension that measures the core using `Reference.ConvertToStableRepresentation`.
> Using that information, you can then construct the references as follows:
```csharp
string format = "{0}:{1}:{2}";
string uid = wall.uidId;
int nines = -9999;
refString = string.Format(format,uid,nines,1);
Reference wall_centre = Reference
.ParseFromStableRepresentation(doc,refString);
refString = string.Format(format,uid,nines,2);
Reference core_outer = Reference
.ParseFromStableRepresentation(doc,refString);
refString = string.Format(format,uid,nines,3);
Reference core_inner = Reference
.ParseFromStableRepresentation(doc,refString);
refString = string.Format(format,uid,nines,4);
Reference core_centre = Reference
.ParseFromStableRepresentation(doc,refString);
```
#### Grid References for Dimensioning
Two other discussions deal with dimensioning grids:
One, on
the [failure to get reference for grid for dimension in 2017](https://forums.autodesk.com/t5/revit-api-forum/fail-to-get-reference-for-grid-for-dimension-in-2017/m-p/6636474).
Alexander [@aignatovich](https://forums.autodesk.com/t5/user/viewprofilepage/user-id/1257478) Ignatovich, aka Александр Игнатович solved it like this:
> Instead of getting references from grid curves I used a reference to the entire grid element:
```csharp
var gridRef = Reference.ParseFromStableRepresentation(doc, grid.UniqueId);
```
More solutions were presented for
using [`NewDimension` between grids – invalid number of references](https://forums.autodesk.com/t5/revit-api-forum/newdimension-between-grids-invalid-number-of-references/m-p/7459503), e.g.,
Richard Thomas suggested:
> Basically, you can't use the reference from the curve of the grid (since that is a surface); you have to create a new reference by using the grid element itself: `New Reference(Grid)`. This reference you can then add to the reference array of the `Create.NewDimension` method. It's the same when dimensioning reference planes.
> If you only have the curve information to start with, you need to get the grid element from the curve's reference and then create a new reference from that.
> THE `Create.NewDimension` method expects the reference type of a grid or reference plane to be `REFERENCE_TYPE_NONE` (for element), not `REFERENCE_TYPE_SURFACE`.
#### Simple, Better Grid References for Dimensioning
My colleague Jim Jia adds:
> Please also look at the simple and better code to build reference for `Grid` below; we don't need to retrieve reference from `Grid` geometry. We verified that this works well:
```csharp
///
/// Return a reference built directly from grid
/// summary>
Reference GetGridRef( Document doc )
{
ElementId idGrid = new ElementId( 397028 );
Element eGrid = doc.GetElement( idGrid );
return new Reference( eGrid );
}
```