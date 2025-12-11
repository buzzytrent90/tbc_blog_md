---
ai_optimized: true
author: Jeremy Tammik
complexity_score: 2.4
content_type: tutorial
optimization_date: '2025-12-11T11:44:14.285471'
original_url: https://thebuildingcoder.typepad.com/blog/0623_training_prep.html
post_number: '0623'
reading_time_minutes: 4
series: general
slug: training_prep
source_file: 0623_training_prep.htm
tags:
- revit-api
- views
title: Revit API Training Preparation
word_count: 706
---

### Revit API Training Preparation

Yesterday I pointed out that the
[Revit 2012 API training material](http://thebuildingcoder.typepad.com/blog/2011/07/revit-api-training-material.html) is
now available online from the
[Revit Developer Center](http://www.autodesk.com/developrevit) and
provided a more or less detailed overview of its content.

In addition to downloading and working through that, there are a couple of other preparations that I recommend my training participants to work through up front:

- Install the Revit SDK.- Install RevitLookup.- Install RvtSamples.- Work through the developer guide [hands-on walk-through tutorials](http://wikihelp.autodesk.com/Revit/enu/2012/Help/API_Dev_Guide/0001-Introduc1/0012-Getting_12).- Work through the [DevTV classes](http://thebuildingcoder.typepad.com/blog/2010/10/revit-2011-devtv.html).- Work through the [My First Revit Plugin](http://thebuildingcoder.typepad.com/blog/2011/05/my-first-revit-plug-in.html) tutorial.

The SDK, DevTV classes, and first plugin tutorial are all available from the
[Revit Developer Center](http://www.autodesk.com/developrevit).

RevitLookup, RvtSamples, and the developer guide are part of the SDK, and the developer guide is also
[available online](http://wikihelp.autodesk.com/Revit/enu/2012/Help/API_Dev_Guide).

Whether you work through the developer guide hands-on walk-throughs, the DevTV classes or the 'first plugin' material first is up to you.
The content covered overlaps to some extent.

If you like, you can postpone the installation of the SDK until after you have worked through these.
Working through them first will deepen your understanding of the installation of Revit add-ins, which will be useful for setting up RevitLookup and RvtSamples.

Once you have done all or most of this, you will be in a good shape to work through the
[Revit 2012 API training material](http://thebuildingcoder.typepad.com/blog/2011/07/revit-api-training-material.html),
which provides a pretty thorough introduction to all the basics of the Revit API.

Everything beyond goes into more depth in specific areas of your interest, either related to the particular Revit flavour appropriate to your domain or specialised advanced areas within the Revit API.

Here are a few more notes on the installation of the SDK, RevitLookup and RvtSamples:

Installing the SDK is trivial; it is an executable file, so you run it and specify a location of your choice to place the contents into.

To install
[RevitLookup](http://thebuildingcoder.typepad.com/blog/2010/05/revitlookup-update.html) (formerly
RvtMgdDbg, described
[here](http://thebuildingcoder.typepad.com/blog/2009/02/rvtmgddbg.html) in detail),
you open the Visual Studio project in the RevitLookup subfolder of the SDK installation folder, compile it, edit the add-in manifest file to point to the correct assembly location specifying the path of your add-in DLL, and copy that to the
[appropriate](http://thebuildingcoder.typepad.com/blog/2010/04/addin-manifest-and-guidize.html#1)
[location](http://wikihelp.autodesk.com/Revit/enu/2012/Help/API_Dev_Guide/0001-Introduc1/0018-Add-In_I18/0022-Add-in_R22) (replace '2011' by '2012') for Revit to pick up.

RvtSamples provides an interface to load and run all of the hundred-plus other SDK samples, instead of installing them individually.
Therefore, it requires you to compile all the other SDK samples as well, as described in the following posts:

- [Managing SDK samples](http://thebuildingcoder.typepad.com/blog/2008/08/managing-sdk-sa.html).- [Loading SDK samples](http://thebuildingcoder.typepad.com/blog/2008/09/loading-sdk-sam.html).- [Loading The Building Coder samples](http://thebuildingcoder.typepad.com/blog/2008/11/loading-the-building-coder-samples.html).- [Compiling and loading all the SDK samples](http://thebuildingcoder.typepad.com/blog/2009/04/installing-the-revit-2010-sdk.html).- [RvtSamples Conversion from 2009 to 2010](http://thebuildingcoder.typepad.com/blog/2009/05/porting-the-building-coder-samples.html).- [Migrating the Building Coder Samples to Revit 2012](http://thebuildingcoder.typepad.com/blog/2011/04/migrating-the-building-coder-samples-to-revit-2012.html).- [Updating my SDK installation for Revit 2012 Update Release 1](http://thebuildingcoder.typepad.com/blog/2011/06/updated-sdk-for-revit-2012-update-release-1.html).

I hope that this information is complete enough to get anybody interested in the Revit API happily started.
Please let me know if you discover anything missing.