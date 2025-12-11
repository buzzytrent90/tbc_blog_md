---
ai_optimized: true
author: Jeremy Tammik
complexity_score: 2.3
content_type: tutorial
optimization_date: '2025-12-11T11:44:15.153945'
original_url: https://thebuildingcoder.typepad.com/blog/1028_newfamilyinstance.html
post_number: '1028'
reading_time_minutes: 2
series: elements
slug: newfamilyinstance
source_file: 1028_newfamilyinstance.htm
tags:
- family
- revit-api
- views
- elements
title: Family Instance Placement
word_count: 465
---

### Family Instance Placement

One issue that keeps cropping up is how to determine which
[NewFamilyInstance overload](http://thebuildingcoder.typepad.com/blog/2011/01/newfamilyinstance-overloads.html) to
use to place an instance of a given family.

As always, you need to first ensure that the desired workflow can be achieved manually through the user interface.

If anything fails in the interactively driven steps, it certainly will not work better when driven programmatically, and the user interface will provide much richer information on possible failure reasons.

The first place to look for information on driving this programmatically this is the Revit API Developer Guide Wikihelp section on
[FamilyInstances](http://wikihelp.autodesk.com/Revit/enu/2014/Help/3661-Developers/0074-Revit_Ge74/0083-Family_I83/0086-FamilyIn86) and
[creating FamilyInstance objects](http://wikihelp.autodesk.com/Revit/enu/2014/Help/3661-Developers/0074-Revit_Ge74/0083-Family_I83/0086-FamilyIn86#GUID-5163F12D-96E2-42A3-8B10-FBBCE24A6A12).

We looked at a large number of different examples of programmatic family instance placement and how to determine the correct NewFamilyInstance overload to use in the past, as you can see by simply searching globally for
[revit api newfamilyinstance overload](http://lmgtfy.com/?q=revit+api+newfamilyinstance+overload).

Here are two important testing tools for this that were discussed back in 2010:

1. [PlaceInstancesOnViews](http://thebuildingcoder.typepad.com/blog/2010/11/place-site-component.html#1):
   This method tests placing a specific family instance in all views, to ensure that an instance can indeed be placed in a view using a specific NewFamilyInstance overload.
2. [TestAllOverloads](http://thebuildingcoder.typepad.com/blog/2010/11/place-site-component.html#2):
   This method calls all possible overloads of NewFamilyInstance in order to find one that works.

Since then, a couple of enhancements have been added to the Revit API to simplify this task.

Here two important pieces of functionality to help clarify the family instance placement type up front, cited from the What's New section of the Revit 2013 API help file RevitAPI.chm:

#### NewFamilyInstance validation

Some validation has been added to overloads of NewFamilyInstance.
This validation is intended to prevent use of the incorrect overload for a given family symbol input.
For specific details on the exceptional conditions that will be validated, consult the documentation.

#### Family.PlacementType

This new property provides information about the placement type for instances of the given family. PlacementType roughly maps to the overloads of NewFamilyInstance.

The first enhancement automatically notifies you if you try to place an instance with some invalid input and can cause issues where previously *apparently* working code now raises an exception.

The second one is something you have to be aware of yourself, though.

I hope you make a note of these possibilities and find them helpful when you next run into any programmatic family instance placement issues.