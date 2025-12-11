---
ai_optimized: true
author: Jeremy Tammik
complexity_score: 3.8
content_type: qa
optimization_date: '2025-12-11T11:44:16.383241'
original_url: https://thebuildingcoder.typepad.com/blog/1608_create_mass_floor.html
post_number: '1608'
reading_time_minutes: 3
series: general
slug: create_mass_floor
source_file: 1608_create_mass_floor.md
tags:
- csharp
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
title: Create Mass Floor
word_count: 680
---

### Creating Face Wall and Mass Floor
I went on my first ski tour this season, on and around
the [Hochgescheid, in the Black Forest](https://en.wikipedia.org/wiki/List_of_mountains_and_hills_of_the_Black_Forest),
for a change:
![Hochgescheid](img/020_michael_jeremy_1000x404.jpg)
On the Revit API side of things, besides lots of interesting issues in
the [Revit API discussion forum](http://forums.autodesk.com/t5/revit-api-forum/bd-p/160),
a Japanese ADN case came up on programmatically generating a mass floor, enabling us to mention yet another hitherto unmentioned Revit API usility class, `MassInstanceUtils`:
\*\*Question:\*\* マス床をAPIで生成する方法があれば教えてください。 – Please tell me if a method exists to generate a mass floor using the API.
\*\*Answer:\*\* First of all, here is the description
on [creating a floor from a mass floor in the user interface](http://help.autodesk.com/view/RVT/2018/ENU/?guid=GUID-03EABD3A-4736-4762-97F8-F473FEE18162).
Please be aware that the Revit API does not support the creation of in-place mass elements.
As suggested by
the [Revit API discussion forum](http://forums.autodesk.com/t5/revit-api-forum/bd-p/160) thread
on [creating components in mode 'Model In-Place'](https://forums.autodesk.com/t5/revit-api-forum/creation-components-in-mode-model-in-place/m-p/3822639),
the alternative is to create a [direct shape](http://thebuildingcoder.typepad.com/blog/about-the-author.html#5.50) instead.
Unfortunately, you will still not end up with a mass.
However, if you have already created a mass element by some other means, you can use it to generate face walls and mass floors programmatically.
This really old thread asking the same question
on [creating mass floors and or scope boxes](https://forums.autodesk.com/t5/revit-api-forum/creating-mass-floors-and-or-scope-boxes/m-p/2388509) still
remains open today.
I'll add this answer to that as soon as I complete publishing it on The Building Coder.
Nowadays, the answer to this question is provided by
the [MassInstanceUtils.AddMassLevelDataToMassInstance](http://www.revitapidocs.com/2018.1/fe3b251b-2677-094d-7e72-77fea0f49f24.htm) method.
It generates floors defined by levels based on a given mass geometry.
Its use is demonstrated and very clearly explained
by Harry Mattison of [Boost your BIM](https://boostyourbim.wordpress.com) in his post
on [automating the Building Maker workflow](https://boostyourbim.wordpress.com/2014/02/11/automating-the-building-maker-workflow).
It uses the `FaceWall` `Create` and `MassInstanceUtils` `AddMassLevelDataToMassInstance` methods to generate walls and floors based on a selected mass element's geometry, faces and levels.
He demonstrates the add-in running live in his three-and-a-half-minute video
on [Face Wall and Mass Floor creation with the Revit API](https://youtu.be/nHWen2_lN6U):

The API calls are driven by Harry's C# .NET Revit API macro `CreateFaceWallsAndMassFloors`.
Here is the code copied from Harry's post and added
to [The Building Coder samples](https://github.com/jeremytammik/the_building_coder_samples)
in [lines 414-473 of CmdFaceWall.cs](https://github.com/jeremytammik/the_building_coder_samples/blob/master/BuildingCoder/BuildingCoder/CmdFaceWall.cs#L414-L473):
```csharp
#region CreateFaceWallsAndMassFloors
// By Harry Mattison, Boost Your BIM,
// Automating the Building Maker workflow
// https://boostyourbim.wordpress.com/2014/02/11/automating-the-building-maker-workflow/
// Face Wall and Mass Floor creation with the Revit API
// https://youtu.be/nHWen2_lN6U
///
/// Create face walls and mass floors on and in selected mass element
/// summary>
public void CreateFaceWallsAndMassFloors( UIDocument uidoc )
{
Document doc = uidoc.Document;
FamilyInstance fi = doc.GetElement(
uidoc.Selection.PickObject(
ObjectType.Element ) )
as FamilyInstance;
WallType wType = new FilteredElementCollector( doc )
.OfClass( typeof( WallType ) )
.Cast().FirstOrDefault( q
=> q.Name == "Generic - 6\" Masonry" );
Options opt = new Options();
opt.ComputeReferences = true;
using( Transaction t = new Transaction( doc ) )
{
t.Start( "Create Face Walls & Mass Floors" );
foreach( Solid solid in fi.get_Geometry( opt )
.Where( q => q is Solid ).Cast() )
{
foreach( Face f in solid.Faces )
{
if( FaceWall.IsValidFaceReferenceForFaceWall(
doc, f.Reference ) )
{
FaceWall.Create( doc, wType.Id,
WallLocationLine.CoreExterior,
f.Reference );
}
}
}
FilteredElementCollector levels
= new FilteredElementCollector( doc )
.OfClass( typeof( Level ) );
foreach( Level level in levels )
{
MassInstanceUtils.AddMassLevelDataToMassInstance(
doc, fi.Id, level.Id );
}
t.Commit();
}
}
#endregion // CreateFaceWallsAndMassFloors
```
Many thanks to Harry for documenting and implementing this!