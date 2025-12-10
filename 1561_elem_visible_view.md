---
post_number: "1561"
title: "Elem Visible View"
slug: "elem_visible_view"
author: "Jeremy Tammik"
tags: ['elements', 'family', 'filtering', 'levels', 'parameters', 'revit-api', 'sheets', 'views']
source_file: "1561_elem_visible_view.md"
original_url: "https://thebuildingcoder.typepad.com/blog/1561_elem_visible_view.html"
---

### Retrieving Elements Visible in View
Song 'Minho' [@moshpit](https://forums.autodesk.com/t5/user/viewprofilepage/user-id/1967259) Min re-opened
the [Revit API discussion forum](http://forums.autodesk.com/t5/revit-api-forum/bd-p/160) thread
on [getting the `ElementId` of all visible entities in a viewport](http://forums.autodesk.com/t5/revit-api/get-elementid-of-all-visible-entities-in-a-viewport/m-p/5879077),
providing a good opportunity to mention Colin Stark's answer to the StackOverflow thread
on [determining whether a `FamilyInstance` is visible in a View](http://stackoverflow.com/questions/44012630/determine-is-a-familyinstance-is-visible-in-a-view):
#### Question
I am looking for code to get the ElementIds of all entities inside a viewport.
A viewport is a region of a big view plan.
How to retrieve the `ElementId` of all visible entities in a viewport?
#### Answer
Use the `FilteredElementCollector` constructor overload taking a view id argument:

```
  public FilteredElementCollector(
    Document document,
    ElementId viewId )
```

It returns the host document elements visible in this view.
There is a limitation for elements in linked documents, as explained in the thread
on [elements from linked document](http://forums.autodesk.com/t5/revit-api/elements-from-linked-document/m-p/5867049).
In order to include linked elements, you can use
a [custom exporter](http://thebuildingcoder.typepad.com/blog/2013/07/graphics-pipeline-custom-exporter.html).
For more info on that, please refer
to [The Building Coder custom exporter topic group](http://thebuildingcoder.typepad.com/blog/about-the-author.html#5.1).
The solution for your specific case is to find the `ViewPlan` associated with the viewport and check the `BoundingBox` of all elements on the level of this `ViewPlan` to find ones that fit inside the `ViewPlan.CropBox`.
#### Question 2
I am learning Revit API and a beginner. I am so desperate to learn what you have mentioned. Could you provide a code of how the method you have mentioned can be fleshed out?
#### Answer 2
Welcome to Revit API programming!
I suggest that you first of all take a look at
the [Revit API getting started material](http://thebuildingcoder.typepad.com/blog/about-the-author.html#2) and
work through the step-by-step instructions provided by the DevTV and My First Revit Plugin video tutorials.
That will show you what other important material is available that you MUST be aware of, answer this question of yours, and many, many more besides.
Once you understand the basics of using filtered element collectors, the answers provided above will become self-explanatory.
They already answer your question in full.
The Building Coder discussed the related topic of determining all views displaying a specific element in depth, implementing an external command named `CmdViewsShowingElements` to try out some approaches:
- [Determine views displaying given element](http://thebuildingcoder.typepad.com/blog/2014/05/views-displaying-given-element-svg-and-nosql.html)
- [Determining views showing an element](http://thebuildingcoder.typepad.com/blog/2016/12/determining-views-showing-an-element.html)
A similar question recently also arose in the StackOverflow thread on how
to [determine whether a `FamilyInstance` is visible in a `View`](http://stackoverflow.com/questions/44012630/determine-is-a-familyinstance-is-visible-in-a-view).
Colin Stark answered that succinctly, saying:
> I have found that the most reliable way of knowing whether an element is visible in a view is to use a FilteredElementCollector specific to that view. There are so many different ways of controlling the visibility of an element that it would be impractical to try to determine this any other way.
> Below is the utility function I use to achieve this. Note this works for any element, and not just for family instances.
> The category filter is used to eliminate any element not of the desired category before using the slower parameter filter to find the desired element. It is probably possible to speed this up further with clever usage of filters, but I have found that it is plenty fast enough for me in practice.
I added Colin's code to [The Building Coder samples](https://github.com/jeremytammik/the_building_coder_samples)
module [CmdViewsShowingElement](https://github.com/jeremytammik/the_building_coder_samples/blob/master/BuildingCoder/BuildingCoder/CmdViewsShowingElements.cs).
```csharp
///
/// Determine whether an element is visible in a view,
/// by Colin Stark, described in
/// http://stackoverflow.com/questions/44012630/determine-is-a-familyinstance-is-visible-in-a-view
/// summary>
public static bool IsElementVisibleInView(
this View view,
Element el )
{
if( view == null )
{
throw new ArgumentNullException( nameof( view ) );
}
if( el == null )
{
throw new ArgumentNullException( nameof( el ) );
}
// Obtain the element's document.
Document doc = el.Document;
ElementId elId = el.Id;
// Create a FilterRule that searches
// for an element matching the given Id.
FilterRule idRule = ParameterFilterRuleFactory
.CreateEqualsRule(
new ElementId( BuiltInParameter.ID_PARAM ),
elId );
var idFilter = new ElementParameterFilter( idRule );
// Use an ElementCategoryFilter to speed up the
// search, as ElementParameterFilter is a slow filter.
Category cat = el.Category;
var catFilter = new ElementCategoryFilter( cat.Id );
// Use the constructor of FilteredElementCollector
// that accepts a view id as a parameter to only
// search that view.
// Also use the WhereElementIsNotElementType filter
// to eliminate element types.
FilteredElementCollector collector =
new FilteredElementCollector( doc, view.Id )
.WhereElementIsNotElementType()
.WherePasses( catFilter )
.WherePasses( idFilter );
// If the collector contains any items, then
// we know that the element is visible in the
// given view.
return collector.Any();
}
}
```
![Eyes](img/eyes26.gif)