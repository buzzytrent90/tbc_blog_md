---
ai_optimized: true
author: Jeremy Tammik
complexity_score: 2.3
content_type: qa
optimization_date: '2025-12-11T11:44:15.879018'
original_url: https://thebuildingcoder.typepad.com/blog/1373_worksharingutils.html
post_number: '1373'
reading_time_minutes: 3
series: general
slug: worksharingutils
source_file: 1373_worksharingutils.md
tags:
- csharp
- elements
- geometry
- python
- revit-api
- sheets
- views
title: Worksharingutils
word_count: 510
---

### WorksharingUtils
Rudolf [Revitalizer](http://forums.autodesk.com/t5/user/viewprofilepage/user-id/1103138) Honke
has frequently requested better documentation of the Revit API `\*Utils` classes, e.g., when he pointed out
the [handy utility classes](http://thebuildingcoder.typepad.com/blog/2013/04/handy-utility-classes.html) back in 2013.
This is still an interesting and somewhat poorly documented area in the Revit API, as we can see from the discussion below:
- [WorksharingUtils and the WorksharingTooltipInfo class](#2)
- [Ten interesting C# features](#3)
- [A new acronym – TL;DR](#4)
#### WorksharingUtils and the WorksharingTooltipInfo Class
The development team answered the following recent Revit API forum thread
on [API access to the workshared element user history](http://forums.autodesk.com/t5/revit-api/workshared-element-user-history/m-p/5896907):
\*\*Question:\*\* Is there any access at all to Workshared Element User History?
If I turn on one of the worksharing display modes and hover my mouse over any element, I get a popup like this advising me of who that element was originally created by and who last updated it in the central model:
![Worksharing tooltip](img/worksharing_tooltip.png)
Is it possible to programmatically access these fields?
\*\*Answer:\*\* I was unable to find any information about this, so I passed on the question to the Revit development team.
Their answer is short, succinct and to the point: this is in the `WorksharingTooltipInfo` class acquired from the `WorksharingUtils.GetWorksharingTooltipInfo` method.
So we are saved once again by our beloved `Utils` classes.
They and the important functionality they provide are apparently still hard to find.
Many thanks to Scott and Arnošt for pointing this out, and the importance of improving their visibility.
#### Ten Interesting C# Features
My colleague [Augusto Gonçalves](http://adndevblog.typepad.com/aec/augusto-goncalves.html) recently pointed out an article
on [10 features in C# that you really should learn (and use!)](http://www.codeaddiction.net/articles/15/10-features-in-c-that-you-really-should-learn-and-use).
While I do not totally agree with them all, similarly to some of the options voiced in
the comments section, it is still definitely an interesting and worthwhile quick read.
\*\*Addendum:\*\* In
his [comment](http://thebuildingcoder.typepad.com/blog/2015/11/worksharingutils.html#comment-2350844009) below,
Guy Robinson points out another interesting related link to the MSDN article
on [new features in C# 6](http://blogs.msdn.com/b/csharpfaq/archive/2014/11/20/new-features-in-c-6.aspx),
which is actually more structured and in-depth than the former.
#### A New Acronym – TL;DR
I really dislike acronyms that I do not know the meaning of.
I quite like them when I do, though, and I am certain that my communication partner knows them as well.
I just discovered this one that I like, since it expresses a sentiment I often share: TL;DR – too long; didn't read.
Of course, this leads back to the first topic again: the `\*Utils` classes are all documented in the Revit API help file RevitAPI.chm... all you have to do is read it :-)