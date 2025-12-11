---
ai_optimized: true
author: Jeremy Tammik
complexity_score: 2.6
content_type: qa
optimization_date: '2025-12-11T11:44:15.102285'
original_url: https://thebuildingcoder.typepad.com/blog/1005_get_line_styles.html
post_number: '1005'
reading_time_minutes: 2
series: general
slug: get_line_styles
source_file: 1005_get_line_styles.htm
tags:
- csharp
- elements
- filtering
- revit-api
- rooms
- selection
- transactions
title: Retrieving All Available Line Styles
word_count: 363
---

### Retrieving All Available Line Styles

I
[mentioned](http://thebuildingcoder.typepad.com/blog/2012/09/divideparts-in-f.html#5) an
[AEC DevBlog](http://adndevblog.typepad.com/aec) post
providing sample code showing how to
[retrieve all line styles](http://adndevblog.typepad.com/aec/2012/09/retrieving-a-list-of-line-styles.html) through
the CurveElement GetLineStyleIds property and mentioned that you can create a temporary curve element to invoke the property on in a separate transaction which is rolled back afterwards.

Actually, there is no need at all for all this rigmarole, as the following answer to a developer query shows:

**Question:** I need to retrieve all the available line styles in a project.

I did some research and found the abovementioned blog post requiring a dummy line to achieve that.

This seems to be inconsistent with obtaining other project data like Material and FillPattern instances.

You can easily make selections of all kinds, like these:

```csharp
  ElementCollectionHelper.GetAllProjectElements(doc)
    .Where( c => c.GetType() == typeof(Material) )
    .ToList());

  ElementCollectionHelper.GetAllProjectElements(doc)
    .Where( c => c.GetType() == typeof(FillPatternElement) )
    .ToList());
```

I find it rather strange that you need a dummy line to obtain line styles.

Is there a better way, please?

**Answer:** While the Revit API does not provide a true 'Line style' element, the line styles are actually subcategories of the Lines category.
Therefore, the FilteredElementCollector cannot easily be used for this in a single statement, like in your examples above.

It should be possible to retrieve the line styles without a line instance, though.

Here’s a macro that lists all subcategories of the Lines category:

```csharp
  public void GetListOfLinestyles( Document doc )
  {
    Category c = doc.Settings.Categories.get\_Item(
      BuiltInCategory.OST\_Lines );

    CategoryNameMap subcats = c.SubCategories;

    foreach( Category lineStyle in subcats )
    {
      TaskDialog.Show( "Line style", string.Format(
        "Linestyle {0} id {1}", lineStyle.Name,
        lineStyle.Id.ToString() ) );
    }
  }
```

Note that some line styles like 'Room Boundary' cannot actually be assigned to arbitrary lines in the UI, but this should be good enough to find a usable one.

Once you have a collection of the line style subcategories of interest, you can create a filtered element collector retrieving all ElementType elements belonging to any one of them.