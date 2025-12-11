---
ai_optimized: true
author: Jeremy Tammik
complexity_score: 3.8
content_type: qa
optimization_date: '2025-12-11T11:44:16.299960'
original_url: https://thebuildingcoder.typepad.com/blog/1565_dim_hatch.html
post_number: '1565'
reading_time_minutes: 6
series: general
slug: dim_hatch
source_file: 1565_dim_hatch.md
tags:
- elements
- geometry
- references
- revit-api
- rooms
- sheets
- transactions
- views
- walls
title: Dim Hatch
word_count: 1122
---

### Hatch Line Dimensioning Voodoo
Reporting on a very exciting topic from the Revit API discussion forum from the Forge accelerator in Barcelona:
- [Barcelona Forge accelerator](#2)
- [Retain the Add-In GUID](#3)
- [Dimension on hatch pattern](#4)
#### Barcelona Forge Accelerator
I am back in the Barcelona Autodesk office supporting the [Forge Accelerator](http://autodeskcloudaccelerator.com/forge-accelerator).
I was here [last year](http://thebuildingcoder.typepad.com/blog/2016/03/adn-becomes-forge-and-barcelona-accelerator.html#3)
[as well](http://thebuildingcoder.typepad.com/blog/2016/05/forge-accelerator-devcon-and-answer-day.html), where
I started working on
the [roomedit3d project](https://github.com/jeremytammik/roomedit3d), implementing all
its [Revit-independent functionality](http://thebuildingcoder.typepad.com/blog/2016/05/roomedit3d-console-test-and-rendering-assets.html#2).
Let's look at two Revit API issues before I dig in deeper into Forge:
#### Retain the Add-In GUID
\*\*Question:\*\* If I create a Revit add-in, should the `AddinId` be unique across different versions of Revit (2017/2018/2019)?
Or should I retain the same `AddinId`?
\*\*Answer:\*\* The different versions can have the same GUID.
Not only can, but they definitely should keep the same add-in id.
Some frameworks, e.g., updaters, external services and extensible storage, use the add-in id and serialise it with relevant data.
Therefore, you will want to retain the same add-in id in future versions, so that Revit doesn't treat the capability as unrecognized.
#### Dimension on Hatch Pattern
On to a much more complex question, adding a new entry to our collection
of [undocumented `ElementId` relationships](http://thebuildingcoder.typepad.com/blog/2011/11/undocumented-elementid-relationships.html).
It came up in
the [Revit API discussion forum](http://forums.autodesk.com/t5/revit-api-forum/bd-p/160) thread
on creating a [dimension on hatch pattern slab](https://forums.autodesk.com/t5/revit-api-forum/dimension-on-hatch-pattern-slab/td-p/7063302) and
was answered very creatively indeed by Fair59, who has already contributed numerous answers to other very tricky issues.
Fair59's solution reminds me of
Scott Wilson's [`Reference` stable representation magic voodoo](http://thebuildingcoder.typepad.com/blog/2016/04/stable-reference-string-magic-voodoo.html):
\*\*Question:\*\* I want to get dimension on hatch pattern slab like this:
![Dimension hatch lines](img/dim_hatch_01.png)
But I don't know how to retrieve hatch line.
\*\*Answer 1:\*\* First: Use [RevitLookup](https://github.com/jeremytammik/RevitLookup) to
find out what elements you are after:
Second: Retrieve their geometry, asking for `ComputeReferences = true`.
Third: Create the dimensioning using the references:
- [Dimension walls by iterating faces](http://thebuildingcoder.typepad.com/blog/2011/02/dimension-walls-by-iterating-faces.html)
- [Dimension walls using `FindReferencesByDirection`](http://thebuildingcoder.typepad.com/blog/2011/02/dimension-walls-using-findreferencesbydirection.html)
\*\*Response:\*\* That does not help.
1. I already know it's a floor and can get it.
2. I already retrieve the solid of floor.
The problem is, from the solid I cannot retrieve the geometry of the line pattern in the floor to create dimensions between them.
Can you explain more details, please?
Here my code in this project so far:
```csharp
// Pick floor element
IList elems = sel.PickElementsByRectangle();
Floor fl = null;
// Loop elems to get Floor
foreach( Element e in elems )
{
if( e is Floor )
{
fl = (Floor) e;
break;
}
}
Solid s = null;
Options o = new Options();
o.ComputeReferences = true;
GeometryElement geoElem = fl.get_Geometry( o );
// Loop the geometry of Floor to get solid
foreach( GeometryObject geoObj in geoElem )
{
if( s != null )
break;
s = (Solid) geoObj;
}
// Stop here,have solid of floor but don't know
// how to retrieve line in fill pattern of floor :(
```
\*\*Answer 2:\*\* The answer is: you cannot get the geometry objects of the hatch pattern, at all.
You even cannot calculate the hatch line positions manually, since there is no information about origin and direction in relation to the face.
Also, the user may have edited the hatch pattern origin by hand.
Additionally, there are two kinds of fill patterns: 'model' and 'drawing'. One of them depends on view's scale, the other not.
You can analySe the related FillPattern's FillGrids, but you don't have the logical connection to the Face's coordinates.
So, no, it cannot be done.
\*\*Answer 3:\*\* There is indeed no direct way to get to hatch lines.
However, we have just about enough information to construct them.
I had a hunch, that the reference to the surface and to the hatch lines are related.
A call to `Reference.ConvertToStableRepresentation` returns:
- Surface: 926c1621-1982-4ef6-bcbc-21fe350f0087-001f2d0b:1:SURFACE
- Hatch Line: 926c1621-1982-4ef6-bcbc-21fe350f0087-001f2d0b:1:SURFACE/6
So, given a `StableRepresentation` of a surface , you can construct a `Reference` to a hatch line like this, where `index` is an integer `>1`:
```csharp
Reference HatchRef
= Reference.ParseFromStableRepresentation( doc,
StableRepresentation of Surface + "/index" );
```
The indices are distributed over the different hatch lines as follows:
![Dimension hatch line indices](img/dim_hatch_line_index.png)
Take 2 references to a HatchLine and you can create a dimension.
Once you have the dimension, you can determine the direction and position of the 'unbound' lines.
Here is my code the creates that dimension:
```csharp
Floor _floor;
Reference top = HostObjectUtils.GetTopFaces( _floor )
.First();
PlanarFace topFace = _floor.GetGeometryObjectFromReference(
top ) as PlanarFace;
// Check for model surfacepattern
Material mat = doc.GetElement(
topFace.MaterialElementId ) as Material;
FillPatternElement patterntype = doc.GetElement(
mat.SurfacePatternId ) as FillPatternElement;
if( patterntype == null ) return Result.Failed;
FillPattern pattern = patterntype.GetFillPattern();
if( pattern.IsSolidFill
|| pattern.Target == FillPatternTarget.Drafting )
return Result.Failed;
// Get number of gridLines in pattern
int _gridCount = pattern.GridCount;
// Construct StableRepresentations and find the
// Reference to HatchLines
string StableRef = top.ConvertToStableRepresentation(
doc );
ReferenceArray _resArr = new ReferenceArray();
for( int ip = 0; ip < 2; ip++ )
{
int index = 1 + ( ip \* _gridCount \* 2 );
string StableHatchString = StableRef
+ string.Format( "/{0}", index );
Reference HatchRef = null;
try
{
HatchRef
= Reference.ParseFromStableRepresentation(
doc, StableHatchString );
}
catch
{ }
if( HatchRef == null ) continue;
_resArr.Append( HatchRef );
}
// 2 or more References => create dimension
if( _resArr.Size > 1 )
{
using( Transaction t = new Transaction( doc ) )
{
t.Start( "dimension Hatch" );
Dimension _dimension = doc.Create.NewDimension(
doc.ActiveView, Line.CreateBound(
XYZ.Zero, new XYZ( 10, 0, 0 ) ),
_resArr );
// Move dimension a tiny amount to orient the
// dimension perpendicular to the hatchlines
// I can't say why it works, but it does.
ElementTransformUtils.MoveElement( doc,
_dimension.Id, new XYZ( .1, 0, 0 ) );
t.Commit();
}
}
```
\*\*Response:\*\* I ended up avoiding the problem by using a `FilledRegion` previous set on the floor pattern. The `FilledRegion` dimension should be equal to one tile dimension.
So, I can locate one tile on Floor, then create a dimension from that to others.
Many thanks to Fair59 for this extremely creative approach and another gem in the collection of tips and tricks making use of undocumented Revit `ElementId` relationships!