---
ai_optimized: true
author: Jeremy Tammik
complexity_score: 1.7
content_type: qa
optimization_date: '2025-12-11T11:44:14.866979'
original_url: https://thebuildingcoder.typepad.com/blog/0909_units_revitlookup.html
post_number: 0909
reading_time_minutes: 3
series: general
slug: units_revitlookup
source_file: 0909_units_revitlookup.htm
tags:
- elements
- parameters
- revit-api
- views
title: Units and RevitLookup
word_count: 538
---

### Units and RevitLookup

Here is a typical question from a Revit API newbie that prompts me to reiterate some important basic beginner information on units and the RevitLookup tool for exploring the Revit database:

**Question:** How can I retrieve the length and mass units from a Revit document?

**Answer:** The internal Revit database length unit is always imperial feet.

That cannot be changed, so there is you answer for that.

Similarly, the internal database unit for weight is kg, as far as I know.

#### Revit Database Units

In fact, most internal Revit database units except length are
[SI units](http://en.wikipedia.org/wiki/International_System_of_Units),
and
[length is handled differently](http://thebuildingcoder.typepad.com/blog/2011/03/internal-imperial-units.html).

The user interface can display other units, set up just as the user prefers.
They can also be modified programmatically, as demonstrated by the
[ProjectUnit SDK sample](http://thebuildingcoder.typepad.com/blog/2009/10/unit-suffix-and-the-projectunit-sdk-sample.html).

You can determine any internal database unit by setting the value of a given parameter using it on some Revit element to a specific number, e.g. 1.0, in the user interface.

You can then use [RevitLookup](#3) to examine the parameter's resulting underlying database value.

The quotient between the user display and the underlying database values provides a factor that you can use to convert between user display and internal database units and enables you to determine what the internal database unit actually is.

This topic has been discussed exhaustively on The Building Coder blog.
In fact, one of the very first posts ever is still perfectly valid.
It deals with and covers all the
[fundamental facts of units](http://thebuildingcoder.typepad.com/blog/2008/09/units.html).

We presented a
[unit conversion tool](http://thebuildingcoder.typepad.com/blog/2009/08/unit-conversion.html) and
updated it for Revit
[2011](http://thebuildingcoder.typepad.com/blog/2011/01/unit-conversion-and-new-blogs.html) and
[2013](http://thebuildingcoder.typepad.com/blog/2013/01/revit-2013-unit-conversion-utility.html).

Many other aspects of this topic are collected in The Building Coder
[units category](http://thebuildingcoder.typepad.com/blog/units).

#### RevitLookup

I mentioned RevitLookup above.
This is one of the most important tools, both for developers and end users, that you absolutely must be aware of.

It was formerly named RvtMgdDbg.
This
[detailed description](http://thebuildingcoder.typepad.com/blog/2009/02/rvtmgddbg.html) still
calls it so.

Here are
[step-by-step installation instructions](http://whatrevitwants.blogspot.de/2010/06/how-to-view-revit-database-2011-secret.html) for
Revit 2011 that pretty much still apply.
There is even a
[video demonstrating the installation for Revit 2013](http://www.youtube.com/watch?v=mkkWdrnSwCw).

RevitLookup is included in the Revit SDK available from the
[Revit Developer Center](http://www.autodesk.com/developrevit),
which also includes all the other important material for
[getting started with the Revit API](http://thebuildingcoder.typepad.com/blog/getting_started).
Here is also an updated version with a
[corrected add-in manifest file](http://adndevblog.typepad.com/aec/2012/09/revitlookupaddin-manifest-file-.html)
to simplify the installation.

I hope that fully answers your question and helps you get started successfully.