---
ai_optimized: true
author: Jeremy Tammik
complexity_score: 2.2
content_type: qa
optimization_date: '2025-12-11T11:44:13.388191'
original_url: https://thebuildingcoder.typepad.com/blog/0115_filter_for_family.html
post_number: '0115'
reading_time_minutes: 2
series: filtering
slug: filter_for_family
source_file: 0115_filter_for_family.htm
tags:
- csharp
- elements
- family
- filtering
- revit-api
- views
title: Filter for a Family
word_count: 400
---

### Filter for a Family

We already discussed the topic of
[family category and filtering](http://thebuildingcoder.typepad.com/blog/2009/01/family-category-and-filtering.html)
previously, but this question keeps cropping up anyway, so here is a repetition from a slightly different point of view by Saikat Bhattacharya.

**Question:**
I am trying to get the loaded families with a LightingFixtures category.
However, I am not getting the results I expected.
I used the following code:

```csharp
Autodesk.Revit.Creation.Filter cf
  = app.Create.Filter;

Filter f1 = cf.NewTypeFilter(
  typeof( Family ) );

Filter f2 = cf.NewCategoryFilter(
  BuiltInCategory.OST\_LightingFixtures );

LogicAndFilter f3 = cf.NewLogicAndFilter(
  f1, f2 );

List<Element> a = new List<Element>();
doc.get\_Elements( f3, a );
foreach( Family f in a )
{
  Util.InfoMsg( f.Name );
}
```

No elements are returned from the Document using this filter.
So, I retrieved all the families instead and checked if the category name of the family is "Lighting Fixtures".
This is language dependant and consumes long time.
How can I achieve this using filters?

**Answer:**
As mentioned in the discussion on
[family category and filtering](http://thebuildingcoder.typepad.com/blog/2009/01/family-category-and-filtering.html),
the Family class does not implement the Category property for all family files.
Therefore, you cannot use a category filter to identify family files.

If you retrieve all family files and check their category using some other means, like in you the workaround you describe, you can enhance performance and avoid the language dependence by comparing the category element id instead of its name string.

By the way, you can always manually test and interactively simulate the result of any API filter by using the Revit SDK sample ElementsFilter.
For a quick test, load a lighting fixture family into a new Revit document and run this sample with a family filter selecting the lighting fixture family.
The following results show the Category as blank for the Family element:

![ElementsFilter results filtering for a family](img/filter_for_family.gif)

Instead of creating a filter using Family and Category, you can also change it to FamilySymbol and Category to obtain the symbols instead of the family elements.
The Category property is always defined for family symbols.
If you need the families, they are accessible from the symbols as well.
If you simply replace typeof( Family ) by typeof( FamilySymbol ) in the definition of the filter f1 in the code above, it works fine.