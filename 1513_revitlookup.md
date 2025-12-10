---
post_number: "1513"
title: "Revitlookup"
slug: "revitlookup"
author: "Jeremy Tammik"
tags: ['revit-api', 'selection', 'sheets']
source_file: "1513_revitlookup.md"
original_url: "https://thebuildingcoder.typepad.com/blog/1513_revitlookup.html"
---

### RevitLookup Supports Spot Dimension and Escape
[RevitLookup](https://github.com/jeremytammik/RevitLookup) sports
two new useful enhancements
since [adding support for the NuGet Revit API package](http://thebuildingcoder.typepad.com/blog/2016/12/nuget-revit-api-package.html#3) in mid-December:
- [Close all forms using the `Escape` key](#2)
- [Support for spot dimension](#3)
#### Close all Forms Using the Escape Key
Paul [@luftbanana](https://github.com/luftbanana) van Nassau added support to close all the nested RevitLookup forms using the `Escape` key by assigning them the appropriate cancel button behaviour
in [pull request #20](https://github.com/jeremytammik/RevitLookup/pull/20):
> I assigned the CancelButton field (in each form that didn't have a CancelButton field assigned yet) to the proper button available.
> This provides the function of closing each form with the keyboard ESC key (which can be faster than pointing and clicking your mouse).
I integrated Paul significant time-saving enhancement
into [RevitLookup release 2017.0.0.10](https://github.com/jeremytammik/RevitLookup/releases/tag/2017.0.0.10).
Many thanks to Paul for implementing this!
Here is [the diff showing what he added](https://github.com/jeremytammik/RevitLookup/compare/2017.0.0.9...2017.0.0.10).
#### Support for Spot Dimension
LeeJaeYoung reported an issue snooping spot dimensions in
a [comment](http://thebuildingcoder.typepad.com/blog/2016/04/revit-2017-revitlookup-and-sdk-samples.html#comment-3077972791) on
the discussion of [Revit 2017, RevitLookup and SDK Samples](http://thebuildingcoder.typepad.com/blog/2016/04/revit-2017-revitlookup-and-sdk-samples.html), saying:
> Spot dimension position and text position causes an error in RevitLookup 2017.0.0.8, but works okay in Revit 2015.
I think the leader and text position properties of spot dimension are causing this.
Snooping a `SpotDimension` displays the message:

```
  Can't set/get leader position for one of these cases:

  1. SpotElevation.
  2. When using equality formula.
  3. When dimension style is ordinate.
```

![RevitLookup spot dimension error](img/revitlookup_spot_dimension_error.png)
Here are the [steps to reproduce](http://thebuildingcoder.typepad.com/blog/2016/04/revit-2017-revitlookup-and-sdk-samples.html#comment-3083808853):
- Create a spot dimension on a slab.
- Select it.
- Pick RevitLookup > Snoop Current Selection.
- The error occurs.
Here is a [47-second YouTube recording](https://www.youtube.com/watch?v=CFCHpmyYDa4) demonstrating the problem:

Many thanks to LeeJaeYoung for reporting and documenting this problem!
I fixed it
in [RevitLookup release 2017.0.0.11](https://github.com/jeremytammik/RevitLookup/releases/tag/2017.0.0.11).
Here is [the diff showing what I changed](https://github.com/jeremytammik/RevitLookup/compare/2017.0.0.10...2017.0.0.11).
As always, you can grab the most up-to-date version from
the [RevitLookup repository `master` branch](https://github.com/jeremytammik/RevitLookup).