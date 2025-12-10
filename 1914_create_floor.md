---
post_number: "1914"
title: "Create Floor"
slug: "create_floor"
author: "Jeremy Tammik"
tags: ['elements', 'family', 'geometry', 'levels', 'parameters', 'revit-api', 'sheets', 'views', 'windows']
source_file: "1914_create_floor.md"
original_url: "https://thebuildingcoder.typepad.com/blog/1914_create_floor.html"
---

### Triangle Count, New Floor and Slab Creation
I am back from my break, which I mostly spent chilling with friends and hiking in the Jura and Ticino hills and mountains in the west and south of Switzerland.
Let's start getting back into the flow again with these news bites:
- [Model polygon or triangle count](#2)
- [Floor creation API clarification](#3)
- [Dynamo Studio EOL](#4)
- [My solar power project](#5)
#### Total Modal Polygon or Triangle Count
A recent [Revit API discussion forum](http://forums.autodesk.com/t5/revit-api-forum/bd-p/160) thread
asked [how to get polygon count of the project](https://forums.autodesk.com/t5/revit-api-forum/how-to-get-polygon-count-of-the-project/m-p/10530975):
\*\*Question:\*\* I am exporting the model into OBJ format.
Is it possible to the polygon count or the number of triangles in the model before export?
Does Revit provide any API for that?
Can I count the polygons for each element and then add them all?
\*\*Answer:\*\* No, no such API is provided ready-built.
As a non-programmer, you could export your Revit project (.rvt) or family (.rfa) file to FBX format and use an external viewer to view and analyse that.
For instance, in Revit, go to File > Export > FBX.
Open the FBX file in [Microsoft 3D Viewer](https://www.microsoft.com/en-us/p/3d-viewer/9nblggh42ths).
Depending on the file size, it may take a little while to open.
A spinning 3D box icon will indicate loading is in progress.
In the viewer, go to Tools and click on "Stats & Shading".
It will list the number of triangles and vertices.
The 3D Viewer is a free download if you don't already have it in Windows 10.
As a programmer, you can do as you suggest, retrieve all the element geometry, tessellate it, and sum up the total number of triangles.
You might find it easiest to do so using
a [custom exporter](http://thebuildingcoder.typepad.com/blog/about-the-author.html#5.1).
The result should be the same as in the FBX export.
[@techXMKH9](https://forums.autodesk.com/t5/user/viewprofilepage/user-id/5770785) very
kindly shared a nice and minimal sample custom exporter implementation that I integrated into a new external command `CmdTriangleCount`
in [release 2022.0.151.0](https://github.com/jeremytammik/the_building_coder_samples/releases/tag/2022.0.151.0)
of [The Building Coder samples](https://github.com/jeremytammik/the_building_coder_samples):
```csharp
class TriangleCounterContext : IExportContext
{
private Document document;
private Func isCanceled;
///
/// Callback at end with total count of model
/// geometry triangles and material ids
/// summary>
private Action callback;
private long numTriangles;
private List materialIds;
private bool includeMaterials = true;
public TriangleCounterContext(
Document document,
Func isCanceled,
Action callback )
{
this.isCanceled = isCanceled;
this.callback = callback;
this.document = document;
this.materialIds = new List();
}
public void OnPolymesh( PolymeshTopology polymesh )
{
this.numTriangles += (long) polymesh.NumberOfFacets;
}
public void Finish()
{
this.callback( this.numTriangles, this.materialIds.Count );
}
public bool IsCanceled()
{
return false;
}
public bool Start()
{
this.materialIds = new List();
return true;
}
public void OnRPC( RPCNode node )
{
}
public void OnLight( LightNode node )
{
}
public RenderNodeAction OnViewBegin( ViewNode node )
{
node.LevelOfDetail = 8;
return 0;
}
public void OnViewEnd( ElementId elementId )
{
}
public RenderNodeAction OnFaceBegin( FaceNode node )
{
return 0;
}
public void OnFaceEnd( FaceNode node )
{
}
public RenderNodeAction OnElementBegin( ElementId elementId )
{
return 0;
}
public void OnElementEnd( ElementId elementId )
{
}
public RenderNodeAction OnInstanceBegin( InstanceNode node )
{
return 0;
}
public void OnInstanceEnd( InstanceNode node )
{
}
public RenderNodeAction OnLinkBegin( LinkNode node )
{
return 0;
}
public void OnLinkEnd( LinkNode node )
{
}
public void OnMaterial( MaterialNode node )
{
if( this.includeMaterials )
{
if( node.MaterialId == ElementId.InvalidElementId )
{
return;
}
if( this.materialIds.Contains( node.MaterialId ) )
{
return;
}
this.materialIds.Add( node.MaterialId );
}
}
}
void TriangleCountReport( long nTriangles, int nMaterials )
{
string s = string.Format(
"Total number of model triangles and materials: "
+ " {0} triangle{1}, {2} material{3}",
nTriangles, Util.PluralSuffix( nTriangles ),
nMaterials, Util.PluralSuffix( nMaterials ) );
Debug.Print( s );
TaskDialog.Show( "Triangle Count", s );
}
public Result Execute(
ExternalCommandData commandData,
ref string message,
ElementSet elements )
{
var app = commandData.Application;
var uidoc = app.ActiveUIDocument;
var doc = uidoc.Document;
TriangleCounterContext context
= new TriangleCounterContext(
doc, null, TriangleCountReport );
CustomExporter exporter
= new CustomExporter(
doc, context );
exporter.Export( doc.ActiveView );
return Result.Succeeded;
}
```
Here is the result of running it in a minimal sample model:
![Triangle count](img/tbc_samples_triangle_count.png "Triangle count")
#### Floor Creation API Clarification
The development team provide some clarification on how to user
the [new floor creation API](https://thebuildingcoder.typepad.com/blog/2021/04/whats-new-in-the-revit-2022-api.html#4.1.4.1) introduced
in Revit 2022.
Some old APIs were obsoleted, and the new methods work a bit differently, so some instructions on how to migrate from the old API to the new may come in handy, especially a sample code snippet like this:
```csharp
/// The example below shows how to use Floor.Create
/// method to create a new Floor with a specified
/// elevation on a level using a geometry profile
/// and a floor type.
/// It shows how to adapt your old code using the
/// NewFloor and NewSlab methods, which became
/// obsolete with Revit 2022.
/// In this sample, the geometry profile is a
/// CurveLoop of lines; you can also use arcs,
/// ellipses and splines.
Floor CreateFloorAtElevation(
Document document,
double elevation )
{
// Get a floor type for floor creation
// You must provide a valid floor type (unlike the
// obsolete NewFloor and NewSlab methods).
ElementId floorTypeId = Floor.GetDefaultFloorType(
document, false );
// Get a level
// You must provide a valid level (unlike the
// obsolete NewFloor and NewSlab methods).
double offset;
ElementId levelId = Level.GetNearestLevelId(
document, elevation, out offset );
// Build a floor profile for the floor creation
XYZ first = new XYZ( 0, 0, 0 );
XYZ second = new XYZ( 20, 0, 0 );
XYZ third = new XYZ( 20, 15, 0 );
XYZ fourth = new XYZ( 0, 15, 0 );
CurveLoop profile = new CurveLoop();
profile.Append( Line.CreateBound( first, second ) );
profile.Append( Line.CreateBound( second, third ) );
profile.Append( Line.CreateBound( third, fourth ) );
profile.Append( Line.CreateBound( fourth, first ) );
// The elevation of the curve loops is not taken
// into account (unlike the obsolete NewFloor and
// NewSlab methods).
// If the default elevation is not what you want,
// you need to set it explicitly.
var floor = Floor.Create( document, new List {
profile }, floorTypeId, levelId );
Parameter param = floor.get_Parameter(
BuiltInParameter.FLOOR_HEIGHTABOVELEVEL_PARAM );
param.Set( offset );
return floor;
}
```
Sorry for the late information, and I hope it still helps with your migration.
It definitely helps for me, since I still have exactly two remaining warnings when compiling The Building Coder samples:
- Warning CS0618 `Document.NewFloor(CurveArray, bool)` is obsolete; this method is deprecated in Revit 2022 and may be removed in the future version of Revit. To create new instance of Floor, call Floor.Create() – in CmdEditFloor.cs line 119
- Warning CS0618 `Document.NewSlab(CurveArray, Level, Line, double, bool)` is obsolete; this method is deprecated in Revit 2022 and may be removed in the future version of Revit. To create new instance of Floor, call Floor.Create() – in CmdCreateSlopedSlab.cs line 88
I can make use of the sample snippet to fix these two now :-)
Many thanks to Oleg Sheydvasser for providing this!
#### Dynamo Studio EOL
Dynamo Studio is nearing its end of life.
That does not affect the rest of the Dynamo project in any way, though, nor its many other incarnations in various shapes and forms.
Please refer to
the [Dynamo Studio Frequently Asked Questions](https://knowledge.autodesk.com/support/dynamo-studio/learn-explore/caas/simplecontent/content/dynamo-studio-faq.html) for
detailed information on the current state and future plans for Dynamo.
#### Solar Power Project
I started experimenting with solar panels and an off-grid system requiring a charger, battery and inverter to generate standard 230 V AC power.
I installed four 100 W peak panels on a small rather steep south facing balcony roof.
Actually, it has a 33 degree twist towards the west, so the direction is SSW.
They did not provide much power before almost midday in summertime, so I later added four more facing east, or rather ESE.
I ran into numerous challenges working with hardware rather than software, nicely described in the article
on [learning from the real world: a hardware hobby project](https://stackoverflow.blog/2021/07/12/the-difference-between-software-and-hardware-projects).
One such challenge was implementing a performance monitor for the charger, including the required serial connection cables, etc.
I published a little piece of associated software in
the [jtracer GitHub repository](https://github.com/jeremytammik/jtracer)
I was initially working with a 12 V battery and am now in the process of upgrading to 24 V.
Once that is up and running, I may start on something bigger, trying to provide enough electrical power for several more apartments.
I hope I don't end up with something like this:
![Solar panels](img/many_solar_panels.jpg "Solar panels")