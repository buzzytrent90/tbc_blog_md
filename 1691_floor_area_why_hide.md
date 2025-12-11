---
ai_optimized: true
author: Jeremy Tammik
complexity_score: 5.2
content_type: qa
optimization_date: '2025-12-11T11:44:16.530823'
original_url: https://thebuildingcoder.typepad.com/blog/1691_floor_area_why_hide.html
post_number: '1691'
reading_time_minutes: 7
series: general
slug: floor_area_why_hide
source_file: 1691_floor_area_why_hide.md
tags:
- elements
- family
- filtering
- geometry
- parameters
- revit-api
- rooms
- sheets
- transactions
- views
- walls
title: Floor Area Why Hide
word_count: 1388
---

### Floor Area Above Room and Mysterious Hide
Today, let's take a break from the last two
day's [Forge](https://autodesk-forge.github.io)
[Design Automation API](https://forge.autodesk.com/en/docs/design-automation/v2/overview) for Revit explorations
on [swallowing warnings](http://thebuildingcoder.typepad.com/blog/2018/09/swallowing-stairsautomation-warnings.html)
and [auto-running an add-in](http://thebuildingcoder.typepad.com/blog/2018/09/auto-run-an-add-in-for-design-automation.html) and
focus on two pure programming questions from
the [Revit API discussion forum](http://forums.autodesk.com/t5/revit-api-forum/bd-p/160) thread instead:
- [Area of an exterior floor above a room](#2)
- [Mysterious element hiding activity](#3)
#### Area of an Exterior Floor Above a Room
Sicheng Zhu asked how
to [get the floor area above a room](https://forums.autodesk.com/t5/revit-api-forum/get-the-floor-area-above-a-room/m-p/8295372):
\*\*Question:\*\* I want to get the exterior floor area above a room, assuming that the floors are modelled separately and I already know which floor is an exterior floor:
![Find exterior floor area above room](img/findexteriorfloorarea0.png)
I want a common solution. Sometimes the floor area is the same as the room area. Sometimes its projection is within the room boundaries. Sometimes they have intersections:
![Find exterior floor area above room](img/findexteriorfloorarea1.png)
Perhaps we can use the class `SolidCurveIntersection` to get the intersected curves and discuss in situations?
Perhaps you can figure out some better solutions?
\*\*Answer:\*\* Thank you for your interesting query.
For a completely generic solution, it seems to me that you will require a 2D Boolean operation.
The Revit geometry API does not provide that built-in, but you can
easily [integrate an external 2D Boolean operations library](http://thebuildingcoder.typepad.com/blog/about-the-author.html#5.2).
Once you have the 2D Boolean operations in place, you can easily create two overlapping horizontal polygons:
- Retrieving the room boundary curves and creating a contiguous curve loop from them.
- Projecting the floor boundary curves, projecting them onto the room top surface, and creating a contiguous curve loop from them.
Here is an example of [projecting and transforming a polygon in Revit](http://thebuildingcoder.typepad.com/blog/2008/12/polygon-transformation.html).
This assumes your wall segments are straight, and not curved.
If they are curved, I would suggest using the curve tessellation provided by the Revit geometry API to convert them to straight segments.
With two overlapping 2D polygons and a 2D Boolean operation in place, I hope your problem is perfectly solved in a totally 'common' way.
#### Mysterious Element Hiding Activity
I would also like to highlight one more
of Frank [@Fair59](https://forums.autodesk.com/t5/user/viewprofilepage/user-id/2083518) Aarssen's very illuminating answers.
This one explores the mysterious element hiding activity performed by Revit in a newly created 3D view, what is causing it, how to resolve it:
- [Made a new 3DView. It's empty. Why?](https://forums.autodesk.com/t5/revit-api-forum/made-a-new-3dview-it-s-empty-why/m-p/8285271)
It also demonstrates the kind of problems a newbie to Revit commonly runs into, in spite of being experienced in other CAD systems:
> I've coded in CATIA, Rhino, AutoCAD, and Maya for years without any issues, but I've never had such trouble than when I want to do the simplest things in Revit.
Fair59 explains in with care and loving, detailed, didactic depth...
\*\*Question:\*\* I created a new 3D view.
But it doesn't anything.
Why?
Please help.
Note: When I iterate through all the objects found using a `FilteredElementCollector` on that specific 3d View, it returns over 9000 elements with their respective element ids. That means there is geometry in that view, but for some reason it's not visible in 3D.
I tried various ViewOrientations and nothing appears in my view.
Nothing is working.
This seems really dumb and simple. Does anyone have a simple portion of Revit API code that makes a 3D view, and has objects in it? What is the proper way to make a 3D View in these API's?
I've coded in CATIA, Rhino, AutoCAD, and Maya for years without any issues, but I've never had such trouble than when I want to do the simplest things in Revit.
\*\*Answer:\*\* Yes, indeed, it can help a lot if you can manage to forget what you know about programming other CAD systems before diving into the Revit API... [Revit and its API is different](http://thebuildingcoder.typepad.com/blog/about-the-author.html#5.41).
First of all, you might want to make a list of the error elements to see the Revit classes to which they belong:
```csharp
List errorIds;
StringBuilder sb = new StringBuilder();
foreach( ElementId id in errorIds )
{
Element e = doc.GetElement( id );
sb.AppendLine( string.Format( "<{0}> {1} [{2}]",
id, e.Name, e.GetType() ) );
}
TaskDialog.Show( "debug", sb.ToString() );
```
I think you should make your own ViewFamilyType, without ViewTemplate so all elements are Visible on creation.
You can then hide all elements (that can be hidden) except the error elements.
```csharp
List errorIds;
ViewFamilyType viewFamilyType = ( from v in
new FilteredElementCollector( doc )
.OfClass( typeof( ViewFamilyType ) )
.Cast()
where v.ViewFamily == ViewFamily.ThreeDimensional
select v ).First();
View3D view;
using( TransactionGroup tg = new TransactionGroup( doc ) )
{
tg.Start( "create Error View" );
using( Transaction t = new Transaction( doc ) )
{
t.Start( "Create Issues View" );
// create new ViewFamilyType without ViewTemplate
ViewFamilyType my_type = viewFamilyType
.Duplicate( "Error_View" ) as ViewFamilyType;
my_type.get_Parameter( BuiltInParameter
.DEFAULT_VIEW_TEMPLATE ).Set(
ElementId.InvalidElementId );
view = View3D.CreateIsometric( doc, my_type.Id );
// Hide DWG, Analytical Categories, Annotaion
view.AreImportCategoriesHidden = true;
view.AreAnalyticalModelCategoriesHidden = true;
view.AreCoordinationModelHandlesHidden = true;
view.AreAnnotationCategoriesHidden = true;
// Hide RVT Links
view.SetCategoryHidden( new ElementId(
BuiltInCategory.OST_RvtLinks ), true );
// show Parts
view.get_Parameter( BuiltInParameter
.VIEW_PARTS_VISIBILITY ).Set(
2 );
t.Commit();
}
IEnumerable elemsInView
= new FilteredElementCollector( doc, view.Id )
.WhereElementIsNotElementType()
.Excluding( errorIds );
List elemsToHide = new List();
foreach( Element e in elemsInView )
{
if( e.CanBeHidden( view ) ) elemsToHide.Add( e.Id );
}
using( Transaction t = new Transaction( doc ) )
{
t.Start( "hide elements" );
view.HideElements( elemsToHide.ToList() );
t.Commit();
}
tg.Assimilate();
}
this.ActiveUIDocument.ActiveView = view;
this.ActiveUIDocument.ShowElements( errorIds );
```
Last thing to check (if needed) visibility of worksets; turn on all worksets, e.g., using `view.SetWorksetVisibility`.
\*\*Response:\*\* Thank you for your detailed answer. I like your approach.
I have a few questions.
- It looks like we are duplicating the first view found via the filter criteria, so the file is basically taking an existing view and duplicating it.
Is that better than the `Create` method?
- Then you are getting a parameter and setting it to an ElementId enumeration of invalid element? Why? Is ElementId.InvalidElementId not an enumeration?
- What is this line setting the `VIEW_PARTS_VISIBILITY` parameter doing exactly? Why `2`?
```csharp
view.get_Parameter( BuiltInParameter
.VIEW_PARTS_VISIBILITY ).Set(
2 );
```
All of this is really good. I'll try it out. I have seen most of what you are providing, but never employed it in that way. Much smarter looping and filtering in your suggestion than my original code, but I'm still negotiating how Revit works.
\*\*Answer:\*\* \*Regarding 'Why duplicate?'\*
It is not duplicating an existing view, but duplicating an existing `ViewFamilyType`.
Because I don't know which elements are on the `errorId` list, I want to make sure that all elements are visible. A view has a `ViewFamilyType`. The `ViewFamilyType` can have a `ViewTemplate` (which regulates visibility / non-visibility of categories) assigned to it, that will be applied to newly created views. As I want all elements visible, I want to use a `ViewFamilyType` without a `ViewTemplate`. To avoid messing with the model, I duplicate the first `ViewFamilyType` found, and remove the `ViewTemplate` from it.
```csharp
my_type.get_Parameter(
BuiltInParameter.DEFAULT_VIEW_TEMPLATE )
.Set( ElementId.InvalidElementId );
```
`ElementId.InvalidElementId` is the equivalent to null for an ElementId value, meaning, do not assign a `ViewTemplate`.
Then I use this `ViewFamilyType` to create the 3D view.
\*Regarding `BuiltInParameter.VIEW_PARTS_VISIBILITY`\*:
Revit has the possibility to [divide (system) families into `Parts`](htttp://help.autodesk.com/view/RVT/2018/ENU/?guid=GUID-DA150C6B-996C-4C70-9E8C-3C536C232851).
A view can show either the original element, the parts, or both. By setting the `BuiltInParameter` to `2`, I set the view to show the original element and the parts.
Once again, very many thanks indeed to Fair59 for your continued invaluable support!