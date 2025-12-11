---
ai_optimized: true
author: Jeremy Tammik
complexity_score: 2.0
content_type: news
optimization_date: '2025-12-11T11:44:15.000442'
original_url: https://thebuildingcoder.typepad.com/blog/0962_bc_migr_2014_deprec.html
post_number: 0962
reading_time_minutes: 2
series: general
slug: bc_migr_2014_deprec
source_file: 0962_bc_migr_2014_deprec.htm
tags:
- elements
- filtering
- references
- revit-api
title: Removing Deprecated API Compilation Warnings
word_count: 311
---

### Removing Deprecated API Compilation Warnings

I completed
[The Building Coder sample migration to Revit 2014](http://thebuildingcoder.typepad.com/blog/2013/04/migrating-the-building-coder-samples-to-revit-2014.html) back
in April, but it was still generating
[111 warnings](zip/bc_migr_2014_d.txt) due
to use of obsolete API calls.

After
[migrating the ADN training lab material](http://thebuildingcoder.typepad.com/blog/2013/06/migrating-the-adn-training-labs-to-revit-2014.html) last
week, I thought I could now turn my attention back to resolving these warnings.

I replaced 27 calls to the get\_EndPoint method, which actually refers to the EndPoint property, by the new GetEndPoint method.

That reduced the count to
[84 warnings](zip/bc_migr_2014_e.txt).Replacing the obsolete creation application NewLineBound and NewLineUnbound calls and adding an element id to the argument to all calls to Document.Delete
reduced the error list to
[48 warnings](zip/bc_migr_2014_f.txt).

Replacing the calls to NewSketchPlane by SketchPlane.Create reduces it to
[24 warnings](zip/bc_migr_2014_g.txt).

Addressing almost all the rest produced the final result for today with just
[5 warnings](zip/bc_migr_2014_h.txt),
three of which are the infamous 'mismatch between the processor architecture of the project being built "MSIL" and the processor architecture' of the referenced Revit API assemblies that I cannot fix myself.

One of the other two refers to a call to the obsolete Document.TitleBlocks, which is intentionally left in to compare with the new approach using the filtered element collector instead.

The second warns about use of the obsolete FindReferencesWithContextByDirection method.
It should be converted to use the ReferenceIntersector class instead, which we postpone to some other time.

Here is the result,
[version 2014.0.100.3](zip/bc_14_100_3.zip) of
The Building Coder samples.

Many of the changes have been marked with a trailing comment stating '2013' and '2014', respectively.

Enjoy!