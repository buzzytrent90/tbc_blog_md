---
post_number: "1730"
title: "Dependent Elem"
slug: "dependent_elem"
author: "Jeremy Tammik"
tags: ['doors', 'elements', 'filtering', 'levels', 'references', 'revit-api', 'selection', 'sheets', 'transactions', 'views', 'walls', 'windows']
source_file: "1730_dependent_elem.md"
original_url: "https://thebuildingcoder.typepad.com/blog/1730_dependent_elem.html"
---

### Retrieving and Snooping Dependent Elements
Håvard Leding of [Symetri](https://www.symetri.com) already made some exciting contributions here,
adding [RevitLookup commands to snoops edges, faces and links](https://thebuildingcoder.typepad.com/blog/2019/01/new-revitlookup-snoops-edge-face-link.html)
and [exploring stable reference relationships](https://thebuildingcoder.typepad.com/blog/2019/02/stable-reference-relationships.html).
Now he raises another exciting topic on using the the `GetDependentElements` method to retrieve and snoop dependent elements, e.g., the sketch of a floor and the model lines defining the floor boundary in that sketch.
Some previous related discussions include use of the temporary transaction trick to
the [change the boundary of floor slabs](https://adndevblog.typepad.com/aec/2013/10/change-the-boundary-of-floorsslabs.html)
and [editing a floor profile](https://thebuildingcoder.typepad.com/blog/2008/11/editing-a-floor-profile.html).
In Håvard's own words:
- [The `GetDependentElements` method](#2)
- [Snoop dependent elements](#3)
- [`CmdSnoopModScopeDependents`](#4)
- [RevitLookup update](#5)
#### The GetDependentElements Method
We were talking about getting the boundaries of a floor through the Sketch element.
The Revit 2019 API provides a new helpful method for this:
The `Element` class has a method `GetDependentElements` taking an `ElementFilter` argument that returns 'elements which will be deleted along with this element'.
It is listed
in [What's New in the Revit 2019 API](https://thebuildingcoder.typepad.com/blog/2018/04/whats-new-in-the-revit-2019-api.html)
under [Find Element Dependencies](https://thebuildingcoder.typepad.com/blog/2018/04/whats-new-in-the-revit-2019-api.html#4.2.3):
> The new method `Element.GetDependentElements` returns a list of ids of elements which are 'children' of this element; that is, those elements which will be deleted along with this element. The method optionally takes an `ElementFilter` which can be used to reduce the output list to the collection of elements matching specific criteria.
That includes the `Sketch` element with the profile containing the boundary lines of a floor.
So, to get the boundary lines of a floor, wall or anything else that you can sketch, you can do this
(it might also apply to things like Crop Boundaries – not sure):
```csharp
ElementClassFilter filter = new ElementClassFilter(
typeof( Sketch ) );
IList dependentIds = e.GetDependentElements(
filter );
foreach( ElementId id in dependentIds )
{
Element depElem = doc.GetElement( id );
if( depElem is Sketch )
{
return depElem as Sketch;
}
}
```
Then you can get the boundaries in a `CurveArrArray` from the `Sketch.Profile`.
A well as references to each `CurveElement`; for a floor, those would be model lines.
Taking separate openings and cuts into account, these boundaries may not be the visible shape of the element, like doors or windows, if the element is a wall.
This goes a long way.
#### Snoop Dependent Elements
This could be a new Revit Lookup command :-)
Passing null as ElementFilter to get all ids.
![Snoop dependent elements](img/snoop_dependent_elements_1.png)
Here is the result after picking a floor:
![Snoop dependent elements](img/snoop_dependent_elements_2.png)
I realised one thing.
As you can see, there are two `Sketch` elements listed.
That is because the floor has an opening, which in turn has its own sketch.
So, it will take some elimination to get the sketch for the floor itself, but very doable.
#### CmdSnoopModScopeDependents
Here is a RevitLookup command implementation set up to process a preselected element:
```csharp
///
/// Snoop dependent elements using
/// Element.GetDependentElements
/// summary>
[Transaction( TransactionMode.Manual )]
public class CmdSnoopModScopeDependents : IExternalCommand
{
public Result Execute(
ExternalCommandData cmdData,
ref string msg,
ElementSet elems )
{
Result result = Result.Failed;
try
{
Snoop.CollectorExts.CollectorExt.m_app = cmdData.Application;
UIDocument uidoc = cmdData.Application.ActiveUIDocument;
ICollection idPickfirst = uidoc.Selection.GetElementIds();
Document doc = uidoc.Document;
ICollection elemSet = new List(
idPickfirst.Select(
id => doc.GetElement( id ) ) );
ICollection ids = elemSet.SelectMany(
t => t.GetDependentElements( null ) ).ToList();
Snoop.Forms.Objects form = new Snoop.Forms.Objects( doc, ids );
ActiveDoc.UIApp = cmdData.Application;
form.ShowDialog();
result = Result.Succeeded;
}
catch( System.Exception e )
{
msg = e.Message;
}
return result;
}
}
```
Implementing the preselection enables a view to be selected in the project browser and snooping that.
For instance, here is the result of snooping a level:
![Snoop dependent elements from level](img/snoop_dependent_level.png)
Ever so many thanks to Håvard for this great suggestion and implementation!
#### RevitLookup Update
I integrated Håvard's enhancement
into [RevitLookup release 2019.0.0.9](https://github.com/jeremytammik/RevitLookup/releases/tag/2019.0.0.9).
Here are two screen snapshots from my sample run, selecting this floor with two holes, one of them elliptical:
![Sample floor element](img/snoop_dependent_floor_1.png)
`CmdSnoopModScopeDependents` lists the following dependent elements:
![Snooping floor dependent elements](img/snoop_dependent_floor_2.png)
I hope you find this useful and wish you lots of fun and success experimenting and enhancing further.