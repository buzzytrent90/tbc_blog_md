---
ai_optimized: true
author: Jeremy Tammik
complexity_score: 4.2
content_type: qa
optimization_date: '2025-12-11T11:44:16.886083'
original_url: https://thebuildingcoder.typepad.com/blog/1849_change_elem_color.html
post_number: '1849'
reading_time_minutes: 6
series: transactions
slug: change_elem_color
source_file: 1849_change_elem_color.md
tags:
- elements
- filtering
- parameters
- references
- revit-api
- selection
- sheets
- transactions
- views
- walls
title: Change Elem Color
word_count: 1208
---

### Changing Element Colour and Material
The question of how to change the colour and material of individual elements has come up repeatedly over time:
- [Change element colour in a view](#2)
- [Assign new material to an element](#3)
- [Replace a material in a wall or floor](#4)
#### Change Element Colour in a View
We discussed
how to [change element colour](https://thebuildingcoder.typepad.com/blog/2011/03/change-element-colour.html) way
back in 2011.
The principle remains unchanged, but some API details have changed a bit since then.
Various solutions to change colour have been provided in
the [Revit API discussion forum](http://forums.autodesk.com/t5/revit-api-forum/bd-p/160),
e.g., [how to change the colour of an element](https://forums.autodesk.com/t5/revit-api-forum/how-change-the-color-a-element/m-p/5651177)
and [changing colour by element id + colour palette](https://forums.autodesk.com/t5/revit-api-forum/change-color-by-element-id-color-palette/m-p/4768209),
but most of them are also out of date.
So, to pick this up once again, I added a new sample external command `CmdChangeElementColor`
to [The Building Coder samples](https://github.com/jeremytammik/the_building_coder_samples).
\*\*Question:\*\* How can I change the element display colour in a given view?
\*\*Answer:\*\* You can use
the [OverrideGraphicSettings class](https://www.revitapidocs.com/2020/eb2bd6b6-b7b2-5452-2070-2dbadb9e068a.htm) and
its [SetProjectionLineColor method](https://www.revitapidocs.com/2020/6b780d28-87fb-2ba6-04fa-f973d85ca552.htm) to
change the colour of a selected element in the current view like this:
```csharp
void ChangeElementColor( Document doc, ElementId id )
{
Color color = new Color(
(byte) 200, (byte) 100, (byte) 100 );
OverrideGraphicSettings ogs = new OverrideGraphicSettings();
ogs.SetProjectionLineColor( color );
using( Transaction tx = new Transaction( doc ) )
{
tx.Start( "Change Element Color" );
doc.ActiveView.SetElementOverrides( id, ogs );
tx.Commit();
}
}
public Result Execute(
ExternalCommandData commandData,
ref string message,
ElementSet elements )
{
UIApplication uiapp = commandData.Application;
UIDocument uidoc = uiapp.ActiveUIDocument;
Document doc = uidoc.Document;
View view = doc.ActiveView;
ElementId id;
try
{
Selection sel = uidoc.Selection;
Reference r = sel.PickObject(
ObjectType.Element,
"Pick element to change its colour" );
id = r.ElementId;
}
catch( Autodesk.Revit.Exceptions.OperationCanceledException )
{
return Result.Cancelled;
}
ChangeElementColor( doc, id );
return Result.Succeeded;
}
}
```
This code now lives in the
new [sample command `CmdChangeElementColor`](https://github.com/jeremytammik/the_building_coder_samples/blob/master/BuildingCoder/BuildingCoder/CmdChangeElementColor.cs)
in [The Building Coder samples](https://github.com/jeremytammik/the_building_coder_samples).
#### Assign new Material to an Element
A slightly trickier question is how to assign a new material:
\*\*Question:\*\* I would like to assign a material to a selected element. The material assigned will come from spreadsheet data. I already know how to create a material object from the data, and now I would like to assign it to selected elements in Revit. How can this be achieved?
\*\*Answer:\*\* Regarding the assignment of a material to a BIM element:
Please be aware that Revit is a BIM software. It produces a realistic building model.
In reality, you cannot simply change the material of an existing building element such as a wall or a floor.
If your floor is made of concrete, it stays concrete.
If you prefer a wooden floor, you have to remove the concrete one first.
Hence, simply swapping the material of a wall or floor is not as straightforward as one might assume.
Furthermore, the material is not determined by the wall or floor itself, but by its type.
Maybe you can simply swap the type.
Higher up in the controlling hierarchy comes the category.
The category does in fact provide
a [`Material` property](https://www.revitapidocs.com/2020/00aa768a-fca2-172f-e5d4-a4d787803983.htm) that
can be read and written.
So, one way to control the material of a wall is to set its category's material.
However, this will affect other walls sharing the same category.
You may be better served manipulating a sub-category instead.
In any case, you need to proceed with care to avoid wrecking your model.
Here is a code snippet that demonstrates changing an element category's material to a randomly chosen different material:
```csharp
void ChangeElementMaterial( Document doc, ElementId id )
{
Element e = doc.GetElement( id );
if( null != e.Category )
{
int im = e.Category.Material.Id.IntegerValue;
List materials = new List(
new FilteredElementCollector( doc )
.WhereElementIsNotElementType()
.OfClass( typeof( Material ) )
.ToElements()
.Where( m
=> m.Id.IntegerValue != im )
.Cast() );
Random r = new Random();
int i = r.Next( materials.Count );
using( Transaction tx = new Transaction( doc ) )
{
tx.Start( "Change Element Material" );
e.Category.Material = materials[ i ];
tx.Commit();
}
}
}
```
I added a call to this method as well in `CmdChangeElementColor`.
This command is available
in [The Building Coder samples](https://github.com/jeremytammik/the_building_coder_samples)
new [release 2021.0.149.1](https://github.com/jeremytammik/the_building_coder_samples/releases/tag/2021.0.149.1).
![Materials](img/concrete_steel_wood_bamboo.jpg "Materials")
#### Addendum – Replace a Material in a Wall or Floor
Please refer to
the [Revit API discussion forum](http://forums.autodesk.com/t5/revit-api-forum/bd-p/160) thread
on how to [replace material](https://forums.autodesk.com/t5/revit-api-forum/replace-material/m-p/9570625) for
a more realistic and professional discussion on replacing materials in walls and floor in real-life BIMs.
I'll skip all the trivial technical details and jump right to the end, to the real-world experience
of Richard [RPThomas108](https://forums.autodesk.com/t5/user/viewprofilepage/user-id/1035859) Thomas
and [Lukas Kohout](https://forums.autodesk.com/t5/user/viewprofilepage/user-id/767846):
> The reality is that if you are an AEC consultant then the wall types you use are based on the company specification for such you have developed over numerous projects. It would be unusual or inefficient not to have all the wall types you commonly use already defined in a Revit file for import (made up of the materials your specification covers).
> On the other hand, say you’ve developed those types and due to project development (contractor/employer proposals) etc. the materials need to change (for acoustic/sustainability etc.). Do you manually remake the wall types with the new material, or do you create new types from existing types changing the one material that has changed? Probably you don't want to make new types due to the instance assignment of type-based sweeps/reveals etc. So instead should swap the material in the existing types. In theory you could redefine the material itself, but it may be used in elements where the change isn’t required. I'm not surprised people would turn to the API for this kind of task.
> I would also add, from my experience that sometimes you want to create new wall types and materials and you need API to do that.
> In my previous job, we developed a Revit add-in that was connected to building material manufacturers product database. From that data, it created new wall, floor and ceiling types in project. New types had different compound structures, materials assigned and parameters filled. Doing that manually is really tedious, never mind that you would have to find the data from product lists and put them into project yourself.
> At that time, you had to duplicate an existing type in project and then add the new compound structure to it.
Maybe that changed now.