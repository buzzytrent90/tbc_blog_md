---
ai_optimized: true
author: Jeremy Tammik
complexity_score: 2.6
content_type: qa
optimization_date: '2025-12-11T11:44:15.513650'
original_url: https://thebuildingcoder.typepad.com/blog/1209_namingutils.html
post_number: '1209'
reading_time_minutes: 2
series: general
slug: namingutils
source_file: 1209_namingutils.htm
tags:
- csharp
- elements
- filtering
- parameters
- revit-api
- transactions
title: Unique Names and the NamingUtils Class
word_count: 326
---

### Unique Names and the NamingUtils Class

The Revit API is still full of surprises.

Here is another one that leads us to look at a utility class that you may not have noticed:

**Question:** I am encountering a strange problem with the name checking functionality when creating ParameterFilterElements.

The following code snippet creates two ParameterFilterElements with all identical settings except for a slight naming difference:

```csharp
  public void ParameterFilterElementError(
    Document doc )
  {
    ICollection<ElementId> ceilingCategory
      = new List<ElementId>();

    ceilingCategory.Add(
      doc.Settings.Categories.get\_Item(
        BuiltInCategory.OST\_Ceilings ).Id );

    using( Transaction tr = new Transaction( doc ) )
    {
      tr.Start( "Test" );

      ParameterFilterElement.Create( doc,
        "HygienKlass 2", ceilingCategory );

      ParameterFilterElement.Create( doc,
        "Hygienklass 2", ceilingCategory );

      tr.Commit();
    }
  }
```

One is called "Hygienklass 2" and the other "HygienKlass 2". This throws an exception saying that the names are equal.

This happens both in Revit 2014 and Revit 2015.

Is there any fix for this?

**Answer:** The API is consistent with the Revit UI in this case.

The user interface displays the following message when you use two names that differ only by case:

![](img/duplicate_filter_name.png)

This restriction around Revit unique naming is currently not highlighted anywhere in the Revit API help file, even though it applies to more than just filters.

We may enhance the error message to clarify this in future.

The Revit rules around what constitutes a unique name and how names are sorted in lists and trees can definitely cause surprises for some folk.

For this reason, the Revit API actually exposes the NamingUtils.CompareNames method, and also because some non-standard comparisons are used, especially around breaking up sections of numeric and non-numeric sequences.

Please take a look at the NamingUtils class, which provides a collection of utilities related to element naming.

Currently, it implements the following two static member methods:

- CompareNames – compares two object name strings using Revit's comparison rules.
- IsValidName – identifies if the input string is valid for use as an object name in Revit.