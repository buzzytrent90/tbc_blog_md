---
ai_optimized: true
author: Jeremy Tammik
complexity_score: 1.3
content_type: qa
optimization_date: '2025-12-11T11:44:13.633064'
original_url: https://thebuildingcoder.typepad.com/blog/0257_category_name_change.html
post_number: '0257'
reading_time_minutes: 1
series: transactions
slug: category_name_change
source_file: 0257_category_name_change.htm
tags:
- elements
- revit-api
- selection
- transactions
title: Handle Category Name Change
word_count: 252
---

### Handle Category Name Change

Here is a little note I wrote sitting in Washington Dulles airport and did not get around to posting until now, to point out the importance of using language independent category comparisons:

**Question:** Apparently at least one category name changed in the German version of Revit from 2009 to 2010:

- Revit 2009: "Tragende Stützen"- Revit 2010: "Tragwerksstützen"

We rely on category names for mapping purposes.
In this case we discovered the change that was causing problems for us, but I am worried that there may be other similar changes that I also need to be aware of.
Is there any way to use a language independent identifier that will never change?
I very much hope to find a way to identify categories in a language independent fashion.

**Answer:** You can use the built-in category enumeration to make your code language independent when working with categories.
We looked at this and related topics in the discussions on
[category comparison](http://thebuildingcoder.typepad.com/blog/2009/01/category-comparison.html) and
later explored it further for the
[model element selection](http://thebuildingcoder.typepad.com/blog/2009/06/category-comparison-and-model-element-selection-revisited.html),
which we recently enhanced to
[handle a set of multiple categories](http://thebuildingcoder.typepad.com/blog/2009/11/select-model-elements-2.html) using
a list of built-in category enumeration values stored in \_bics\_to\_skip and the helper method SkipThisBic to compare them all with a given value from a candidate element.