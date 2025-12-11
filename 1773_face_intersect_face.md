---
ai_optimized: true
author: Jeremy Tammik
complexity_score: 6.4
content_type: qa
optimization_date: '2025-12-11T11:44:16.715965'
original_url: https://thebuildingcoder.typepad.com/blog/1773_face_intersect_face.html
post_number: '1773'
reading_time_minutes: 7
series: general
slug: face_intersect_face
source_file: 1773_face_intersect_face.md
tags:
- csharp
- elements
- filtering
- geometry
- revit-api
- sheets
- views
- walls
- windows
title: Face Intersect Face
word_count: 1448
---

### Face Intersect Face is Unbounded
My work on setting up a new PC is nearing completion.
There is also a need to clarify the use of the `Face.Intersect(Face)` method:
- [The unbounded `Face.Intersect` method](#2)
- [Making use of the unbounded face intersection](#3)
- [Rectangular face intersection ideas](#4)
- [Copy as HTML update](#5)
- [Visual Studio Revit add-in wizard update](#6)
#### The Unbounded Face.Intersect Method
Several [Revit API discussion forum](http://forums.autodesk.com/t5/revit-api-forum/bd-p/160) threads are discussing the status of
the [Face.Intersect (Face) method](https://www.revitapidocs.com/2020/91f650a2-bb95-650b-7c00-d431fa613753.htm):
- [Surprising results from `Face.Intersect(Face)` method](https://forums.autodesk.com/t5/revit-api-forum/surprising-results-from-face-intersect-face-method/m-p/8992586)
- [Problems with `Intersect` method (`Face`)](https://forums.autodesk.com/t5/revit-api-forum/problems-with-intersect-method-face/m-p/8992566)
- [Get connection type and geometry between two elements from the model](https://forums.autodesk.com/t5/revit-api-forum/get-conection-type-and-geometry-between-two-elements-from-the/m-p/6465671)
- [`Face` class `Intersect` method problem](https://forums.autodesk.com/t5/revit-api-forum/face-class-intersect-method-problem/m-p/7460720)
Briefly, the main problem is this:
\*\*Question:\*\* This issue was mentioned in 2016, in the third thread listed above, but it is worth bringing up again, as the issue hasn't been resolved yet in 2018.
As far as I can tell, the `Face.Intersect(face)` method always returns `FaceIntersectionFaceResult.Intersecting`.
When I run the code below in a view with a single wall and single floor, each face to face test returns an intersection:
![Non-intersecting faces](img/face_intersect_face_1.png)
```csharp
public Result Execute(
ExternalCommandData commandData,
ref string message,
ElementSet elements )
{
var list = new FilteredElementCollector(
commandData.Application.ActiveUIDocument.Document,
commandData.View.Id )
.WhereElementIsNotElementType()
.Where( e => e is Wall || e is Floor );
foreach( var f1 in list.First()
.get_Geometry( new Options() )
.OfType().First()
.Faces.OfType() )
{
foreach( var f2 in list.Last()
.get_Geometry( new Options() )
.OfType().First()
.Faces.OfType() )
{
if( f1.Intersect( f2 )
== FaceIntersectionFaceResult.Intersecting )
{
if( System.Windows.Forms.MessageBox.Show(
"Intersects", "Continue",
System.Windows.Forms.MessageBoxButtons.OKCancel,
System.Windows.Forms.MessageBoxIcon.Exclamation )
== System.Windows.Forms.DialogResult.Cancel )
{
return Result.Succeeded;
}
}
}
}
return Result.Succeeded;
}
```
\*\*Answer:\*\* Thank you for your report and reproducible case.
I see the same behaviour in Revit 2019 as well.
I added some code to The Building Coder samples
in [CmdIntersectJunctionBox.cs](https://github.com/jeremytammik/the_building_coder_samples/blob/master/BuildingCoder/BuildingCoder/CmdIntersectJunctionBox.cs#L27-L116) to
test and report in more depth:
```csharp
///
/// Return all faces from first solid
/// of the given element.
/// summary>
IEnumerable GetFaces( Element e )
{
Options opt = new Options();
IEnumerable faces = e
.get_Geometry( opt )
.OfType()
.First()
.Faces
.OfType();
int n = faces.Count();
Debug.Print( "{0} has {1} face{2}.",
e.GetType().Name, n, Util.PluralSuffix( n ) );
return faces;
}
void TestIntersect( Document doc )
{
View view = doc.ActiveView;
var list = new FilteredElementCollector( doc, view.Id )
.WhereElementIsNotElementType()
.Where( e => e is Wall || e is Floor );
int n = list.Count();
Element floor = null;
Element wall = null;
if( 2 == n )
{
floor = list.First() as Floor;
if( null == floor )
{
floor = list.Last() as Floor;
wall = list.First() as Wall;
}
else
{
wall = list.Last() as Wall;
}
}
if( null == floor || null == wall )
{
Util.ErrorMsg( "Please run this command in a "
+ "document with just one floor and one wall "
+ "with no mutual intersection" );
}
else
{
Options opt = new Options();
IEnumerable floorFaces = GetFaces( floor );
IEnumerable wallFaces = GetFaces( wall );
n = 0;
foreach( var f1 in floorFaces )
{
foreach( var f2 in wallFaces )
{
if( f1.Intersect( f2 )
== FaceIntersectionFaceResult.Intersecting )
{
++n;
if( System.Windows.Forms.MessageBox.Show(
"Intersects", "Continue",
System.Windows.Forms.MessageBoxButtons.OKCancel,
System.Windows.Forms.MessageBoxIcon.Exclamation )
== System.Windows.Forms.DialogResult.Cancel )
{
return;
}
}
}
}
Debug.Print( "{0} face-face intersection{1}.",
n, Util.PluralSuffix( n ) );
}
}
```
Here is a simple test model with a disjunct floor and wall:
![Disjunct floor and wall](img/floor_wall_disjunct.png)
My sample code reports:

```
  Floor has 7 faces.
  Wall has 6 faces.
  38 face-face intersections.
```

So, in fact, not every face-to-face test reports an intersection, because 7\*6 equals 42.
Only the vast majority do :-)
I logged the issue \*REVIT-133627 [Face.Intersect returns false positives]\* with our development team to explore this further and provide an explanation.
This turned out to be a duplicate of an existing older issue, \*REVIT-58034 [API Face.Intersect(Face) returns true even if two faces don't intersect with each other]\* and was therefore closed again.
The development team replied:
> We are aware of this issue.
This function does indeed not do what one expects.
At most, it computes intersections between the underlying (unbounded) surfaces, not the (bounded) faces lying in the surfaces.
As a first step, the documentation will be updated to reflect this fact.
Then, we'll see whether resources can be found to fully implement the face intersection functionality, or remove the incomplete functionality.
As a result, \*REVIT-58034\* was renamed. Currently, the following related tickets have all been closed:
- REVIT-58034 [Improve documentation for API Face.Intersect(Face) to reflect what the function actually does]
- REVIT-133627 [Face.Intersect returns false positives]
- REVIT-133819 [Improve API Face.Intersect(Face) to actually intersect faces, not underlying surfaces]
#### Making Use of the Unbounded Face Intersection
Frank [@Fair59](https://forums.autodesk.com/t5/user/viewprofilepage/user-id/2083518) Aarssen adds:
I have previously plotted the intersections using the `ByRef` curve overload and found this to be the case, as explained in the thread
on the [`Face` class `Intersect` method problem](https://forums.autodesk.com/t5/revit-api-forum/face-class-intersect-method-problem/m-p/7460720):
> Apparently, the intersecting faces are considered infinite with therefore many possible intersections beyond the range of the element itself.
> In this image, the green lines are plotted using curves from the overload `Face.Intersect(ByVal Face, ByRef Curve)`:
![Unbounded face intersections between two walls](img/face_intersect_wall_intersects.png)
> Hard to understand at first why they all exist, but once you trace along parallel to the faces, you can see all are valid.
In actual fact, an API user may prefer to find this form of intersection, rather than be told no intersection exists (due to the bounds of the face preventing it).
You can always compare points on the original face to those on the curve intersect result to check if there is an actual intersection.
However, if no results were given, then that would not be useful at all.
Probably, there should be a bound versus unbound option, perhaps.
#### Rectangular Face Intersection Ideas
The sample images shown above all include rectangular wall faces.
If the potentially intersecting faces are rectangular, the problem can presumably be reduced to a pretty simple query.
One approach would be to retrieve the intersection curves of the unbounded faces and test whether they lie on the bounded faces, as suggested above by Frank.
Alternatively, a DIY solution may also be feasible and more efficient.
After all, a rectangular face can be seen as two coplanar triangles, and the intersection of 3D triangles is a pretty common requirement.
Here are some useful-looking results that showed up in a couple of quick Internet searches:
- [A Fast Triangle-Triangle Intersection Test](https://web.stanford.edu/class/cs277/resources/papers/Moller1997b.pdf)
- [Efficient AABB/triangle intersection in C#](https://stackoverflow.com/questions/17458562/efficient-aabb-triangle-intersection-in-c-sharp)
- [How do I find the Intersection of two 3D triangles?](https://math.stackexchange.com/questions/1220102/how-do-i-find-the-intersection-of-two-3d-triangles)
- [Algorithms for ray (or segment) and triangle intersection, triangle-plane and triangle-triangle intersection](http://geomalgorithms.com/a06-_intersect-2.html)
If anyone would like to share a reliable, robust, and efficient 3D triangle or rectangle intersection algorithm, please let us know.
Thank you!
#### Copy as HTML Update
I implemented and installed a couple of
different [source code colourizer tools](https://thebuildingcoder.typepad.com/blog/about-the-author.html#5.36) in
the past.
This spring, I set up
the [PowerTools 2015 Copy HTML Markup](https://thebuildingcoder.typepad.com/blog/2019/04/close-doc-and-zero-doc-rvtsamples.html#4) functionality.
On the new computer, I updated to Visual Studio 2017 instead of Visual Studio 2015.
Accordingly, I update the code colouriser
to [Productivity Power Tools 2017/2019](https://marketplace.visualstudio.com/items?itemName=VisualStudioPlatformTeam.ProductivityPowerPack2017).
#### Visual Studio Revit Add-In Wizard Update
In the same vein, I reinstalled
the [Visual Studio Revit Add-In Wizards](https://thebuildingcoder.typepad.com/blog/about-the-author.html#5.20),
again, slightly updated for Visual Studio 2017, captured
in [VisualStudioRevitAddinWizard release 2020.0.0.2)](https://github.com/jeremytammik/VisualStudioRevitAddinWizard/releases/tag/2020.0.0.2).