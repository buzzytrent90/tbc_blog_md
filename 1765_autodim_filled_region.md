---
post_number: "1765"
title: "Autodim Filled Region"
slug: "autodim_filled_region"
author: "Jeremy Tammik"
tags: ['elements', 'filtering', 'geometry', 'references', 'revit-api', 'sheets', 'transactions', 'views']
source_file: "1765_autodim_filled_region.md"
original_url: "https://thebuildingcoder.typepad.com/blog/1765_autodim_filled_region.html"
---

### Auto-Dimension Filled Region Boundary
I am back from my break in the French Jura and looking at all the
interesting [Revit API forum discussions](http://forums.autodesk.com/t5/revit-api-forum/bd-p/160) again.
One that stands out and that I'll pick up to get back into the blogging rhythm again is Jorge Villarroel's question
about [creating dimensions for a filled region boundary](https://forums.autodesk.com/t5/revit-api-forum/create-dimensions-for-filled-region-boundary/m-p/8926301),
answered by Alexander [@aignatovich](https://forums.autodesk.com/t5/user/viewprofilepage/user-id/1257478) [@CADBIMDeveloper](https://github.com/CADBIMDeveloper) Ignatovich, aka Александр Игнатович:
- [Programmatically creating dimensions for a filled region](#2)
- [Coding suggestion](#3)
- [Final solution](#4)
![Filled regions auto-dimensioned](img/filled_region_dimensions_auto.png)
#### Programmatically Creating Dimensions for a Filled Region
I am working with dimensions for multiple objects.
The dimension creation method needs a `ReferenceArray` to work.
Now, I need to create dimensions for a filled region:
![Filled region](img/filled_region.png)

Filled region

I can create dimensions manually in the user interface using native commands, no API, just clicking, using "Align Dimension":
![Dimensions for the filled region](img/filled_region_dimensions.png)

Dimensions for the filled region

However, I can't retrieve the reference for the boundary curves to create them programmatically.
I used RevitLookup to search for some reference in the Filled Region sub-elements with no results.
Also tried to get the references from the `CurveLoop` curves, but again, with no results.
Any tip of advice will be very well received.
#### Coding Suggestion
Hi!
The trick is to retrieve the filled region geometry using the appropriate view and setting `ComputeReferences` to true.
Try this code:
```csharp
[Transaction( TransactionMode.Manual )]
public class CreateFillledRegionDimensionsCommand : IExternalCommand
{
public Result Execute(
ExternalCommandData commandData,
ref string message,
ElementSet elements )
{
var uiapp = commandData.Application;
var uidoc = uiapp.ActiveUIDocument;
var doc = uidoc.Document;
var view = uidoc.ActiveGraphicalView;
var filledRegions = FindFilledRegions( doc, view.Id );
using( var transaction = new Transaction( doc,
"filled regions dimensions" ) )
{
transaction.Start();
foreach( var filledRegion in filledRegions )
{
CreateDimensions( filledRegion,
-1 \* view.RightDirection );
CreateDimensions( filledRegion, view.UpDirection );
}
transaction.Commit();
}
return Result.Succeeded;
}
private static void CreateDimensions(
FilledRegion filledRegion,
XYZ dimensionDirection )
{
var document = filledRegion.Document;
var view = (View) document.GetElement(
filledRegion.OwnerViewId );
var edgesDirection = dimensionDirection.CrossProduct(
view.ViewDirection );
var edges = FindRegionEdges( filledRegion )
.Where( x => IsEdgeDirectionSatisfied( x, edgesDirection ) )
.ToList();
if( edges.Count < 2 )
return;
var shift = UnitUtils.ConvertToInternalUnits(
-10 \* view.Scale, DisplayUnitType.DUT_MILLIMETERS )
\* edgesDirection;
var dimensionLine = Line.CreateUnbound(
filledRegion.get_BoundingBox( view ).Min
+ shift, dimensionDirection );
var references = new ReferenceArray();
foreach( var edge in edges )
references.Append( edge.Reference );
document.Create.NewDimension( view, dimensionLine,
references );
}
private static bool IsEdgeDirectionSatisfied(
Edge edge,
XYZ edgeDirection )
{
var edgeCurve = edge.AsCurve() as Line;
if( edgeCurve == null )
return false;
return edgeCurve.Direction.CrossProduct(
edgeDirection ).IsAlmostEqualTo( XYZ.Zero );
}
private static IEnumerable FindRegionEdges(
FilledRegion filledRegion )
{
var view = (View) filledRegion.Document.GetElement(
filledRegion.OwnerViewId );
var options = new Options
{
View = view,
ComputeReferences = true
};
return filledRegion
.get_Geometry( options )
.OfType()
.SelectMany( x => x.Edges.Cast() );
}
private static IEnumerable
FindFilledRegions(
Document document,
ElementId viewId )
{
var collector = new FilteredElementCollector(
document, viewId );
return collector
.OfClass( typeof( FilledRegion ) )
.Cast();
}
}
```
It produces something like this:
![Filled regions dimensioned by Alexander's code](img/filled_region_dimensioned_by_ai.png)
Dimensioning in Revit is one of my favorite topics :-)
#### Final Solution
Thanks, @aignatovich. I really appreciate it.
Your suggestion was the solution to my problem!
I extended the approach, so the method asks for the type name (string) of the dimension you want to assign:
```csharp
private void CreateDimensions(
FilledRegion filledRegion,
XYZ dimensionDirection,
string typeName )
{
var document = filledRegion.Document;
var view = (View) document.GetElement(
filledRegion.OwnerViewId );
var edgesDirection = dimensionDirection.CrossProduct(
view.ViewDirection );
var edges = FindRegionEdges( filledRegion )
.Where( x => IsEdgeDirectionSatisfied( x, edgesDirection ) )
.ToList();
if( edges.Count < 2 )
return;
// Se hace este ajuste para que la distancia no
// depende de la escala. <<<<<< evaluar para
// información de acotado y etiquetado!!!
var shift = UnitUtils.ConvertToInternalUnits(
5 \* view.Scale, DisplayUnitType.DUT_MILLIMETERS )
\* edgesDirection;
var dimensionLine = Line.CreateUnbound(
filledRegion.get_BoundingBox( view ).Min + shift,
dimensionDirection );
var references = new ReferenceArray();
foreach( var edge in edges )
references.Append( edge.Reference );
Dimension dim = document.Create.NewDimension(
view, dimensionLine, references );
ElementId dr_id = DimensionTypeId(
document, typeName );
if( dr_id != null )
{
dim.ChangeTypeId( dr_id );
}
}
private static bool IsEdgeDirectionSatisfied(
Edge edge,
XYZ edgeDirection )
{
var edgeCurve = edge.AsCurve() as Line;
if( edgeCurve == null )
return false;
return edgeCurve.Direction.CrossProduct(
edgeDirection ).IsAlmostEqualTo( XYZ.Zero );
}
private static IEnumerable
FindFilledRegions(
Document document,
ElementId viewId )
{
var collector = new FilteredElementCollector(
document, viewId );
return collector
.OfClass( typeof( FilledRegion ) )
.Cast();
}
private static IEnumerable
FindRegionEdges(
FilledRegion filledRegion )
{
var view = (View) filledRegion.Document.GetElement(
filledRegion.OwnerViewId );
var options = new Options
{
View = view,
ComputeReferences = true
};
return filledRegion
.get_Geometry( options )
.OfType()
.SelectMany( x => x.Edges.Cast() );
}
private static ElementId DimensionTypeId(
Document doc,
string typeName )
{
FilteredElementCollector mt_coll
= new FilteredElementCollector( doc )
.OfClass( typeof( DimensionType ) )
.WhereElementIsElementType();
DimensionType dimType = null;
foreach( Element type in mt_coll )
{
if( type is DimensionType )
{
if( type.Name == typeName )
{
dimType = type as DimensionType;
break;
}
}
}
return dimType.Id;
}
```
This code produces dimensioning as shown in the top-most screen snapshot.
Hope this is helpful for others also!
Many thanks to Jorge and Alexander for this nice solution!