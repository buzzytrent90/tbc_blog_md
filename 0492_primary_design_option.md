---
ai_optimized: true
author: Jeremy Tammik
complexity_score: 2.0
content_type: tutorial
optimization_date: '2025-12-11T11:44:14.047264'
original_url: https://thebuildingcoder.typepad.com/blog/0492_primary_design_option.html
post_number: 0492
reading_time_minutes: 1
series: general
slug: primary_design_option
source_file: 0492_primary_design_option.htm
tags:
- csharp
- elements
- revit-api
title: Primary Design Option
word_count: 153
---

### Primary Design Option

Here is a neat little
[idea by Benson](http://thebuildingcoder.typepad.com/blog/2009/11/visible-elements.html?cid=6a00e553e168978833013488fe0388970c#comment-6a00e553e168978833013488fe0388970c) on
how to retrieve the primary design option in a project that has been rattling around in my to-do list for a while now and seems suitable for a Tel Aviv Saturday morning post:

The method DesignOption.GetActiveDesignOptionId exists in the Revit 2011 API, and this method can return the 'active' design option id.
However, it is not the primary design option id.

The DesignOption class property IsPrimary indicates whether a design option is primary.

So, we can iterate though all design options and use that property to determine the primary one.

The Revit 2010 API does not provides the method 'GetActiveDesignOptionId' and property 'IsPrimary', so it seems impossible to get the active design option and primary option in that version.

Thanks to Benson for this hint!