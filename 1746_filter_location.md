---
ai_optimized: true
author: Jeremy Tammik
complexity_score: 7.1
content_type: qa
optimization_date: '2025-12-11T11:44:16.660484'
original_url: https://thebuildingcoder.typepad.com/blog/1746_filter_location.html
post_number: '1746'
reading_time_minutes: 7
series: filtering
slug: filter_location
source_file: 1746_filter_location.md
tags:
- elements
- family
- filtering
- geometry
- parameters
- revit-api
- rooms
- sheets
title: Filter Location
word_count: 1301
---

### Location Point and Filtering Hints
I am probably doomed to spend the rest of my life telling people not to unnecessarily apply `ToList` to a filtered element collector.
It happened several times again today answering questions in
the [Revit API discussion forum](http://forums.autodesk.com/t5/revit-api-forum/bd-p/160):
- [Don't trust the family instance location point](#2)
- [Searching by name for element type and text note type](#3)
- [Searching using a negated category filter](#4)
#### Don't Trust the Family Instance Location Point
The first issue is not directly related to filtering, however; Benoit explains why
the [element `Location` property value is far from the element bounding box](https://forums.autodesk.com/t5/revit-api-forum/element-location-property-value-is-far-from-the-element-bounding/m-p/8762611):
\*\*Question:\*\* I have a FamilyInstance element.
I'm trying to determine its location (X,Y,Z) coordinates.
The value that is returned is ca. (33, -86, 0).
The true location of the element is ca. (-153, -32, 58).
It doesn't seem to be related to the document Transform since it doesn't fix it, and also the bounding box doesn't even bound this element, neither before nor after applying the transform.
Not even close.
I even tried to look at the element geometry and it still doesn't match.
Is there something I'm missing?
The element location is a `LocationPoint`:
```csharp
Location L = e.Location;
LocationPoint lp = L as LocationPoint;
lp.Point;
```
\*\*Answer:\*\* The location point only depends on how the family is modelled.
If you don't like this, you can modify the family, save it and update it in the model.
That is why you should never use the `LocationPoint` to find an element.
You can't know how the family was modelled, or whether it was modified later...
Many thanks to Benoit Favre, CEO of [etudes & automates](http://www.etudesetautomates.com), for this answer!
#### Searching by Name for Element Type and Text Note Type
A question
on [creating a `TextNote` with a specific type, e.g., 1/10" Arial, 1/10" Monospace...](https://forums.autodesk.com/t5/revit-api-forum/creating-a-textnote-with-a-specific-type-i-e-1-10-quot-arial-1/m-p/8765648):
\*\*Question:\*\* I am trying to create a text note using `TextNote.Create`.
But I am struggling to use anything other than the default `TextNoteType` id.
I want to be able to create text using the '1/10" Monospace' family type that I already have loaded in:
![Text note type type properties](img/tnt_3_type_properties.png)
Can anyone please tell me how to easily do that? I think there used to be a `GetFamilyTypeIdByName` function which no longer exists.
Here are the creation options I set up:
![Text note options](img/tnt_1_textnoteoptions.png)
I use them like this:
![Text note create](img/tnt_2_textnote_create.png)
Thank you!
\*\*Answer:\*\* You can obtain the text note type from the document with a filtered element collector and LINQ.
Here is a code snippet showing how:
```csharp
TextNoteType textNoteType
= new FilteredElementCollector( doc )
.OfClass( typeof( TextNoteType ) )
.Cast()
.Where( q => q.Name == "2.5mm Arial" )
.First();
```
Then you can set the text note id in text note creation call or after it has been created.
```csharp
TypeId = textNoteType.Id;
```
The method you mention, `GetFamilyTypeIdByName`, was probably implemented very similarly.
However, pondering its name, \*FamilyType\* is really more suited for the family editor context.
In the project context, you have a base class for types, `ElementType`.
The `TextNoteType` class is derived from that, as is the `FamilySymbol` class.
Just from the name, I cannot infer whether `GetFamilyTypeIdByName` retrieved `ElementType` objects, `FamilySymbol` ones, or something else.
Anyway, for safety's sake, to cover all bases, I implemented and added three new methods to The Building Coder samples for you:
- GetElementTypeByName
- GetFamilySymbolByName
- GetTextNoteTypeByName
Their implementations are almost identical, except that they retrieve the first named object of the specific class, respectively.
Since there are more `ElementType` objects than `TextNoteType` ones in the project, the latter method is certainly faster.
Also, all three methods could be speeded up by using a (quick) parameter filter instead of the (slower than slow) LINQ post-processing accessed by the `First` method, as described in the recent discussion
on [slow, slower still and faster filtering](https://thebuildingcoder.typepad.com/blog/2019/04/slow-slower-still-and-faster-filtering.html#2).
You can see the methods I added in
this [diff to the previous version](https://github.com/jeremytammik/the_building_coder_samples/compare/2020.0.145.1...2020.0.145.2).
For the sake of completeness, I copied their code here as well:
```csharp
///
/// Return the first element type matching the given name.
/// This filter could be speeded up by using a (quick)
/// parameter filter instead of the (slower than slow)
/// LINQ post-processing.
/// summary>
public static ElementType GetElementTypeByName(
Document doc,
string name )
{
return new FilteredElementCollector( doc )
.OfClass( typeof( ElementType ) )
.First( q => q.Name.Equals( name ) )
as ElementType;
}
///
/// Return the first family symbol matching the given name.
/// Note that FamilySymbol is a subclass of ElementType,
/// so this method is more restrictive above all faster
/// than the previous one.
/// summary>
public static ElementType GetFamilySymbolByName(
Document doc,
string name )
{
return new FilteredElementCollector( doc )
.OfClass( typeof( FamilySymbol ) )
.First( q => q.Name.Equals( name ) )
as FamilySymbol;
}
///
/// Return the first text note type matching the given name.
/// Note that TextNoteType is a subclass of ElementType,
/// so this method is more restrictive above all faster
/// than Util.GetElementTypeByName.
/// summary>
TextNoteType GetTextNoteTypeByName(
Document doc,
string name )
{
return new FilteredElementCollector( doc )
.OfClass( typeof( TextNoteType ) )
.First( q => q.Name.Equals( name ) )
as TextNoteType;
}
```
#### Searching using a Negated Category Filter
Finally, we got a chance to make use of a negated category filter answering this question
on [deleting lines that are not assigned to the  subcategory](https://forums.autodesk.com/t5/revit-api-forum/deleting-lines-that-are-not-assigned-to-the-lt-room-separation/m-p/8765491):
\*\*Question:\*\* I have code that deletes all lines in the document.
I only want to delete the lines that are not on the  subcategory.
I probably need to add something to my `FilteredElementCollector`, but I can't figure out what.
```csharp
//Lines
var linIds = new FilteredElementCollector( doc, vw.Id )
.OfClass( typeof( CurveElement ) )
.ToElementIds();
foreach (var lin_id in linIds)
{
doc.Delete(lin_id);
linCount++;
}
```
\*\*Answer:\*\* You can use LINQ to query the elements.
With LINQ, you can check an element's category like this:
```csharp
// This filtered element collector collects all
// elements in the document except room separation lines
var lines = new FilteredElementCollector( doc )
.OfClass( typeof( CurveElement ) )
.Where( q => q.Category.Id != new ElementId(
BuiltInCategory.OST_RoomSeparationLines ) )
.ToList();
foreach (var line in lines)
{
doc.Delete(line.Id);
}
```
However, this initial suggestion can be improved upon significantly.
As I already very frequently pointed out, calling `ToList` at the end is a waste of time and space.
It requests a (totally unnecessary) copy of all the results from the filtered element collector.
You can iterate over the collector itself directly.
Furthermore, a built-in Revit filter will always be faster than LINQ post-processing.
In this case, you can use
a [negated `ElementCategoryFilter`](https://apidocs.co/apps/revit/2019/6b8f4e3a-1975-7388-3848-462cf305d523.htm) taking
a Boolean argument.
Yet further, you might gain some additional performance by deleting all the elements in one single call to `Delete`, rather than by stepping through them one by one.
For instance, like this:
```csharp
///
/// Delete all non-room-separating curve elements
/// summary>
void DeleteNonRoomSeparators( Document doc )
{
ElementCategoryFilter non_room_separator
= new ElementCategoryFilter(
BuiltInCategory.OST_RoomSeparationLines,
true );
FilteredElementCollector a
= new FilteredElementCollector( doc )
.OfClass( typeof( CurveElement ) )
.WherePasses( non_room_separator );
doc.Delete( a.ToElementIds() );
}
```
I added this method to The Building Coder samples for you, as you can see from
the [diff to the preceding version](https://github.com/jeremytammik/the_building_coder_samples/compare/2020.0.145.2...2020.0.145.3).