---
post_number: "1653"
title: "Dwg Layer Visible"
slug: "dwg_layer_visible"
author: "Jeremy Tammik"
tags: ['csharp', 'elements', 'filtering', 'geometry', 'parameters', 'references', 'revit-api', 'selection', 'sheets', 'transactions', 'views']
source_file: "1653_dwg_layer_visible.md"
original_url: "https://thebuildingcoder.typepad.com/blog/1653_dwg_layer_visible.html"
---

### Foreground Image Import and Visible DWG Geometry
Today, we explore how to retrieve visible DWG geometry, i.e., geometry elements contained in a CAD import instance on a layer that is visible in the currently active view, and how to import an image to the foreground instead of the default background setting:
- [Retrieve CAD import geometry on visible layer](#2)
- [Import image using foreground option](#3)
#### Retrieve CAD Import Geometry on Visible Layer
Ryan Goertzen of [Goertzen Enterprises](http://goertzen-ltd.com) shared a nice sample showing how to retrieve visible DWG geometry, i.e., geometry elements contained in a CAD import instance on a layer that is visible in the currently active view, in
the [Revit API discussion forum](http://forums.autodesk.com/t5/revit-api-forum/bd-p/160) thread
on [visibility of DWG layer](https://forums.autodesk.com/t5/revit-api-forum/visibility-of-dwg-layer/m-p/7975284):
\*\*Question:\*\* I am trying to extract geometry from an import instance whose layers are currently visible in the current view.
I pass the view into the geometry options, but it still returns all geometry in the instance.
I can get the name of the DWG layer from:
```csharp
(curDoc.GetElement(curObj.GraphicsStyleId)
as GraphicsStyle).Name;
```
However, I cannot seem to find anywhere in the `View` class to check if this layer is on or off, which is achievable with the UI visibility dialog.
What would be the best way to approach this?
\*\*Answer:\*\* I looked at the description how
to [hide layers in CAD files](http://help.autodesk.com/view/RVT/2019/ENU/?guid=GUID-EBCAFF76-1BF5-45F1-AB51-9130227091D1) (also
in [LT](https://knowledge.autodesk.com/support/revit-lt/learn-explore/caas/CloudHelp/cloudhelp/2019/ENU/RevitLT-Model/files/GUID-EBCAFF76-1BF5-45F1-AB51-9130227091D1-htm.html?_ga=2.222683099.1372620627.1525160338-2130181328.1465883366)).
It refers to 'imported categories'.
Therefore, you may be able to affect the visibility in the Revit view by controlling the category visibility.
The following two methods do so:
- [GetCategoryHidden](http://www.revitapidocs.com/2018.1/52ce4cea-6f27-9e85-f82a-115e308eebfc.htm) – Checks if elements of the given category are set to be invisible (hidden) in this view
- [SetCategoryHidden](http://www.revitapidocs.com/2018.1/87a1e1e2-ee81-1a73-19d7-895b1fa10158.htm) – Sets if elements of the given category will be visible in this view
To test, you might call `GetCategoryHidden`, make a note of what it returns, modify the setting for a specific layer manually, and check the return value again to see whether it changed.
If so, it may be possible to use `SetCategoryHidden` to control the layer visibility.
\*\*Response:\*\* I see now that the layer categories that I was after to pass into `GetCategoryHidden` are nested one step further down in `GraphicsStyle` → `GraphicsStyleCategory` → `Id`.
In my testing, I was thinking that the API wanted a built-in category. For that, all I could find was the built-in category `OST_ImportObjectStyles`.
The following code works as expected, only adding the DWG geometry that is currently visible in view to the list:
```csharp
foreach( GeometryObject gObject in gElement )
{
gInstance = (GeometryInstance) gObject;
gElement2 = gInstance.GetInstanceGeometry();
foreach( GeometryObject obj in gElement2 )
{
var gStyle = curDoc.GetElement( obj.GraphicsStyleId )
as GraphicsStyle;
// Try Catch because i was getting some Object
// Instance errors with the graphics style object.
try
{
// Add object to list
if( !view.GetCategoryHidden( gStyle.GraphicsStyleCategory.Id ) )
list.Add( obj );
}
catch { continue; }
}
}
```
This required cleaning it up a bit to avoid calling `GetInstanceGeometry` without first checking whether `gInstance` is null, and eliminate
the [evil catch-all exception handler](http://thebuildingcoder.typepad.com/blog/2017/05/prompt-cancel-throws-exception-in-revit-2018.html#5).
To try it out yourself at home, here is
a [simple demonstration project, `dwg_visible_geo_2018.rvt`](zip/dwg_visible_geo_2018.rvt)
([also for Revit 2019](zip/zip/dwg_visible_geo_2019.rvt)).
The project includes a basic macro that will draw a detailed region around the visible layer in
the [linked CAD file, `dwg_visible_geo.dwg`](zip/dwg_visible_geo.dwg):
```csharp
///
/// Pick a DWG import instance, extract polylines
/// from it visible in the current view and create
/// filled regions from them.
/// summary>
public void ProcessVisible( UIDocument uidoc )
{
Document doc = uidoc.Document;
View active_view = doc.ActiveView;
List visible_dwg_geo
= new List();
// Pick Import Instance
Reference r = uidoc.Selection.PickObject(
ObjectType.Element,
new JtElementsOfClassSelectionFilter() );
var import = doc.GetElement( r ) as ImportInstance;
// Get Geometry
var ge = import.get_Geometry( new Options() );
foreach( var go in ge )
{
if( go is GeometryInstance )
{
var gi = go as GeometryInstance;
var ge2 = gi.GetInstanceGeometry();
if( ge2 != null )
{
foreach( var obj in ge2 )
{
// Only work on PolyLines
if( obj is PolyLine )
{
// Use the GraphicsStyle to get the
// DWG layer linked to the Category
// for visibility.
var gStyle = doc.GetElement(
obj.GraphicsStyleId ) as GraphicsStyle;
// Check if the layer is visible in the view.
if( !active_view.GetCategoryHidden(
gStyle.GraphicsStyleCategory.Id ) )
{
visible_dwg_geo.Add( obj );
}
}
}
}
}
}
// Do something with the info
if( visible_dwg_geo.Count > 0 )
{
// Retrieve first filled region type
var filledType = new FilteredElementCollector( doc )
.WhereElementIsElementType()
.OfClass( typeof( FilledRegionType ) )
.OfType()
.First();
using( var t = new Transaction( doc ) )
{
t.Start( "ProcessDWG" );
foreach( var obj in visible_dwg_geo )
{
var poly = obj as PolyLine;
// Draw a filled region for each polyline
if( null != poly )
{
// Create loops for detail region
var curveLoop = new CurveLoop();
var points = poly.GetCoordinates();
for( int i = 0; i < points.Count - 1; ++i )
{
curveLoop.Append( Line.CreateBound(
points[i], points[i + 1] ) );
}
FilledRegion.Create( doc,
filledType.Id, active_view.Id,
new List() { curveLoop } );
}
}
}
}
}
```
![Visible DWG geometry](img/dwg_visible_geo.png)
Many thanks to Ryan for raising, solving and sharing this!
I copied the macro code to
[The Building Coder samples](https://github.com/jeremytammik/the_building_coder_samples)
[release 2019.0.139.2](https://github.com/jeremytammik/the_building_coder_samples/releases/tag/2019.0.139.2)
in
order to format and preserve it for posterity in
the [module CmdProcessVisibleDwg.cs](https://github.com/jeremytammik/the_building_coder_samples/blob/master/BuildingCoder/BuildingCoder/CmdProcessVisibleDwg.cs).
#### Import Image Using Foreground Option
Another import question, this time on images:
\*\*Question:\*\* When I insert a raster image into Revit, by default, it is displayed as 'background'.
Is there any way I can do it using the 'Foreground' option?
It would be a great help to know the property that I can set using the C# Revit API.
I am working on some automation projects, and all the images must be displayed as 'Foreground'.
\*\*Answer:\*\* I first took a look at the Revit API documentation on
the [Import method with `ImageImportOptions`](http://www.revitapidocs.com/2018.1/493d203e-ef4c-b447-f979-52b22725b2ad.htm),
the [ImageImportOptions class](http://www.revitapidocs.com/2018.1/0d92888c-aa24-5d7e-c7f9-a7bdf24e5581.htm) and
[its members](http://www.revitapidocs.com/2018.1/b45a2657-8cdf-6471-4929-a758d1675b17.htm).
It does not answer your question off-hand.
Next, I asked the development team, who explained:
> Not at the time of import.
> However, you can set the built-in parameter `IMPORT_BACKGROUND` to `0` on the `ImageInstance` afterwards.
> The element id of the image instance is returned from the `Document.Import` overloaded method for importing images.
\*\*Response:\*\* As you suggested, I set the parameter `IMPORT_BACKGROUND` to `0`.
It worked.
Thanks a lot for your help.