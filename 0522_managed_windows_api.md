---
ai_optimized: true
author: Jeremy Tammik
complexity_score: 2.0
content_type: qa
optimization_date: '2025-12-11T11:44:14.098113'
original_url: https://thebuildingcoder.typepad.com/blog/0522_managed_windows_api.html
post_number: '0522'
reading_time_minutes: 1
series: general
slug: managed_windows_api
source_file: 0522_managed_windows_api.htm
tags:
- csharp
- revit-api
- vbnet
- windows
title: Managed Windows API
word_count: 183
---

### Managed Windows API

This useful library was pointed out by Augusto Gonçalves to Kean and by Kean on his
[blog](http://through-the-interface.typepad.com/through_the_interface/2011/01/technology-advances-the-managed-windows-api.html),
and seems too good to be missed, so I'll reproduce the pointer here as well:

The
[Managed Windows API](http://mwinapi.sourceforge.net) library:

Here are excerpts from this site regarding the problem:

If you want use Windows functionality in a .NET application which is not covered by the .NET framework (there is no "managed API" available for it), you usually have to use PInvoke, an interface that allows to invoke raw API functions from C# and VB.NET.

And the solution:

[Managed Windows API](http://mwinapi.sourceforge.net) is a collection of C# components that wrap Windows API functionality. It contains those features the author needed for his C# development, but if you have components yourself you want to share, please submit them so that this project can grow.

I hope you find this useful too. Thanks to Kean and Augusto for pointing it out!