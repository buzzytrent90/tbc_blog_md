---
post_number: "1716"
title: "Snoop Edge Face Link"
slug: "snoop_edge_face_link"
author: "Jeremy Tammik"
tags: ['elements', 'family', 'filtering', 'geometry', 'python', 'references', 'revit-api', 'selection', 'sheets']
source_file: "1716_snoop_edge_face_link.md"
original_url: "https://thebuildingcoder.typepad.com/blog/1716_snoop_edge_face_link.html"
---

### New RevitLookup Snoops Edge, Face, Link
Before leaving for the weekend, let me highlight some recent additions
to [RevitLookup](https://github.com/jeremytammik/RevitLookup) by
Håvard Leding of [Symetri](https://www.symetri.com):
- [Three new RevitLookup commands](#3)
- [About "Snoop Pick Face..."](#4)
- [About "Pick Linked Element..."](#5)
- [Running in a family document](#6)
I added and tested the new commands
in [RevitLookup release 2019.0.0.6](https://github.com/jeremytammik/RevitLookup/releases/tag/2019.0.0.6).
Below is the description and some additional background information in Håvard's own words.
Many thanks to Håvard for implementing and sharing this!
#### Three New RevitLookup Commands
This is perhaps something of interest to someone.
Three very simple additions:
![Three new commands](img/hl_revitlookup_01.jpg)
The first one really helped when debugging stable references on joined solid geometry in families.
Just the picked `Reference` passed into the `Object` form.
If I pass the `GeometryObject` (the face), it will not retrieve a reference, presumably because `GeometryObjectFromReference` doesn't calculate references.
If of any use, the [commands are attached here](zip/hl_revitlookup_commands.txt).
Here is the code to generate the additional ribbon entries:
```csharp
optionsBtn.AddPushButton( new PushButtonData( "Snoop Pick Face...", "Snoop Pick Face...", ExecutingAssemblyPath, "RevitLookup.CmdSnoopModScopePickSurface" ) );
optionsBtn.AddPushButton( new PushButtonData( "Snoop Pick Edge...", "Snoop Pick Edge...", ExecutingAssemblyPath, "RevitLookup.CmdSnoopModScopePickEdge" ) );
optionsBtn.AddPushButton( new PushButtonData( "Snoop Pick Linked Element...", "Snoop Linked Element...", ExecutingAssemblyPath, "RevitLookup.CmdSnoopModScopeLinkedElement" ) );
```
#### About "Snoop Pick Face..."
Using `Autodesk.Revit.UI.Selection.ObjectType.Face...` gets you a reference to a face.
But if you use this instead...
```csharp
Face face = cmdData.Application.ActiveUIDocument
.Document.GetElement( refElem )
.GetGeometryObjectFromReference( refElem ) as Face;
```
...then `Face.Reference` will be null.
Such a reference is needed when placing face-based families or dimensions, for example.
But it does get you a stable reference to the face:
```csharp
string stableRef = refElem
.ConvertToStableRepresentation( uidoc.Document );
```
Which you can use to find the same face using `Element.get_Geometry`.
Where `Options` calculate the references.
And that face (really the same face) will have a usable reference.
Using Lookup, I found this `stableRef` inside a `GeomCombination`.
And so, I knew I had to include `GeomCombination` in my `FilteredElementCollector`.
Perhaps this could be an improvement on `GetGeometryObjectFromReference`?
An overload to calcuate references if possible.
`GetGeometryObjectFromReference(` `Reference,` `bool` `CalculatedReference )`.
The new pick options helped guide me to my target face.
#### About "Pick Linked Element..."
"Snoop Pick Linked Element..." I haven't had use for yet.
I suspect I will use it quite a bit when debugging interaction with linked elements.
Seems to me you could have used "Pick Linked Element..." yourself in your latest discussion
on [retrieving linked `IfcZone` elements using Python](https://thebuildingcoder.typepad.com/blog/2019/01/retrieving-linked-ifczone-elements-using-python.html).
![Snooping linked elements](img/hl_revitlookup_04.jpg)
#### Running in a Family Document
In my case, the code is running in a family document, not a project.
I get all the solids I need first, knowing that somewhere inside, there is the face I first picked.
```csharp
List solidsInFamily = new List();
IList geomTypes = new List() { typeof( GenericForm ), typeof( GeomCombination ) };
ElementMulticlassFilter emcf = new ElementMulticlassFilter( geomTypes );
FilteredElementCollector colForms = new FilteredElementCollector( doc )
.WherePasses( emcf );
Options opt = new Options();
opt.ComputeReferences = true;
foreach( CombinableElement combinable in colForms )
{
if( combinable is GenericForm && !( combinable as GenericForm ).IsSolid )
continue;
GeometryElement geomElem = combinable.get_Geometry( opt );
List solids = Utils.GetElementSolids( geomElem );
solidsInFamily.AddRange( solids );
}
```
Then, using the stable reference from "Pick Face", iterate the solids until I find the face I'm looking for:
```csharp
foreach( Solid solid in solids )
{
foreach( Face face in solid.Faces )
{
string stable = face.Reference.ConvertToStableRepresentation( doc );
if( stable == stableRef )
{
return face as PlanarFace;
}
}
}
```
Perhaps there is another, simpler, way of getting (picking) a face which has a reference?
But if I do this, passing the picked face, not the reference:
![Snooping Face object](img/hl_revitlookup_02.jpg)
Then the `Face` has no reference:
![Snooping Face object](img/hl_revitlookup_03.jpg)
Which more or less prevents any interaction with it, such as placing dimensions, alignments or face-based families.
Enjoy!