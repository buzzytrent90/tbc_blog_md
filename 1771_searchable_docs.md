---
ai_optimized: true
author: Jeremy Tammik
complexity_score: 1.8
content_type: documentation
optimization_date: '2025-12-11T11:44:16.710963'
original_url: https://thebuildingcoder.typepad.com/blog/1771_searchable_docs.html
post_number: '1771'
reading_time_minutes: 2
series: filtering
slug: searchable_docs
source_file: 1771_searchable_docs.md
tags:
- elements
- revit-api
- sheets
- views
- filtering
title: Searchable Docs
word_count: 332
---

### 11 Years and Revit API Docs Full Text Search
Happy Birthday to The Building Coder!
The online Revit API documentation now supports full text search.
Dynamo implements a new `ViewCone` functionality:
- [Happy Birthday to The Building Coder](#2)
- [Revit API Docs Full Text Search](#3)
- [Dynamo `ViewCone`](#4)
![Birthday cake with candles](img/saikat_birthday_cake_cropped_550.jpg)
#### Happy Birthday to The Building Coder
Today is The Building Coder's eleventh birthday!
The Building Coder published its first article this day eleven years ago, August 22, 2008.
Many happy returns of the day!
#### Revit API Docs Full Text Search
Gui [@gtalarico](https://twitter.com/gtalarico) Talarico, the author of
the [online Revit API documentation revitapidocs.com](https://www.revitapidocs.com),
announced [new invaluable search functionality](https://twitter.com/gtalarico/status/1163345696038109184):
> Who doesn't love \*Searchable\* Docs?
> Only took me 3 years to catch this
> Who doesn't love \*Searchbable\* Docs?
> Only took me 3 years to catch this
> Thanks guys 🔎🤦‍♂️ [pic.twitter.com/4wXK3Re2Mw](https://t.co/4wXK3Re2Mw)
>
> — Gui Talarico (@gtalarico) [August 19, 2019](https://twitter.com/gtalarico/status/1163345696038109184?ref_src=twsrc%5Etfw)

Here is an example of a sample query:
![Revitapidocs full text search](img/revitapidocs_search.png)
#### Dynamo ViewCone
Kean Walmsley pointed out a new piece of Dynamo functionality that may come in useful for field-of-view analysis, in his article
on [using a view cone for targeted visibility in Dynamo with Space Analysis](https://www.keanw.com/2019/08/using-a-view-cone-for-targeted-visibility-in-dynamo-with-space-analysis.html):
> This post is about a new visibility-related capability: the ViewCone.
This is an object that allows you to perform a more limited visibility analysis within a particular cone of view (i.e., from a point towards a direction with a specified field of view).
This is very handy when analysing the view someone has from their desk, for instance.
Unfortunately for us here, this is probably only available in Dynamo, not in the pure Revit API underlying it.