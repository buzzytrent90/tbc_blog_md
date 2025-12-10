---
post_number: "0120"
title: "Getting Started with the Revit 2009 API"
slug: "getting_started_2009"
author: "Jeremy Tammik"
tags: ['elements', 'revit-api', 'schedules', 'views']
source_file: "0120_getting_started_2009.htm"
original_url: "https://thebuildingcoder.typepad.com/blog/0120_getting_started_2009.html"
---

### Getting Started with the Revit 2009 API

Here is a succinct overview of some resources available for getting started using the Revit API, compiled by Mikako Harada.
The individual items have been mentioned in previous posts, and links to those are included.
This document provides a useful and up-to-date overview for the current state, still focused on Revit 2009.
You can expect an update of this material later in a couple of month's time, when the training material has been enhanced for Revit 2010.

1. [Setting up the Environment](#1)
2. [Where to Start](#2)
3. [For more information](#3)
4. [Looking forward for Revit 2010](#4)

### 1. Setting up the Environment

**Revit Product** you can download the latest version of the released products from the
[Autodesk product page](http://www.autodesk.com)
> Products > Alphabetically.
From each Revit product page, look under Product Download.
Here are the current links, but note that the exact locations may change:

- [Revit Architecture](http://usa.autodesk.com/adsk/servlet/index?siteID=123112&id=9262388)
- [Revit MEP](http://usa.autodesk.com/adsk/servlet/index?siteID=123112&id=9262907)
- [Revit Structure](http://usa.autodesk.com/adsk/servlet/index?siteID=123112&id=9339705)

Without an activation code, you can only use Revit in view-only mode.

**Revit SDK** the Revit SDK is included in every Revit install.

- DVD version: \Utilities\Common\Software Development Kit\Revit 2009 SDK.zip- Web version: \Download\Utilities\Common\Software Development Kit\Revit 2009 SDK.zip

**RevitAPI.dll** is part of every Revit installation and is located in the folder \Program.

**Development Environment** Microsoft Visual Studio 2005 for Revit 2009, and Visual Studio 2008 + SP1 for Revit 2010.
The Express edition can also be used.

**Revit API Documentation** The Revit SDK includes a Developer Guide, help file, and many samples.
More detailed information on the SDK contents and how to manage, build and load the samples was discussed in some early blog posts:

- [Getting Started with the Revit API](http://thebuildingcoder.typepad.com/blog/2008/08/getting-started.html)
- [The Revit SDK Contents](http://thebuildingcoder.typepad.com/blog/2008/08/the-revit-sdk-c.html)
- [Managing SDK Samples](http://thebuildingcoder.typepad.com/blog/2008/08/managing-sdk-sa.html)
- [The SDK Samples Solution SDKSamples2009.sln](http://thebuildingcoder.typepad.com/blog/2008/09/the-sdk-samples.html)
- [Loading SDK Samples](http://thebuildingcoder.typepad.com/blog/2008/09/loading-sdk-sam.html)
- [Debugging a Revit Add-In](http://thebuildingcoder.typepad.com/blog/2008/09/debugging-a-rev.html)

Yet more information for beginners is included in the
[Getting Started](http://thebuildingcoder.typepad.com/blog/getting_started/)
category.

### 2. Where to Start

**a.** Watch the training video - The best way to start will be to watch our
[webcast material](http://download.autodesk.com/media/adn/Revit2009APIWebcast_06May2008.zip).
This includes a recording of the webcast, the Powerpoint presentation, and training labs source code.
This material is also available from the
[Developer Center training schedule page](http://www.adskconsulting.com/adn/cs/api_course_sched.php).
There is a
[Revit Architecture and Revit Structure section](http://usa.autodesk.com/adsk/servlet/index?siteID=123112&id=2484975)
in the Developer Center which provides other material including a polished version of the recording called DevTV, which is available for
[download](http://download.autodesk.com/media/adn/DevTV_Introduction_to_Revit_Programming.zip)
or
[online viewing](http://download.autodesk.com/media/adn/DevTV_Introduction_to_Revit_Programming).
It is slightly out of date, being based on Revit 2008.
But you can still get the basic idea.
You may want to see which one is suited for you.

**b.** Try running the HelloRevit sample yourself, go through the Labs code, and then look at some further samples in the SDK.
The following samples provide good starting points:

- RevitCommands- Fire Rating- Viewers/ElementViewer- CreateBeamsColumnsBraces

**c.** A must-have tool is
[RvtMgdDbg](http://thebuildingcoder.typepad.com/blog/2009/02/rvtmgddbg.html),
which we discussed in a
[separate post](http://thebuildingcoder.typepad.com/blog/2009/02/rvtmgddbg.html).

### 3. For more information

- [Autodesk University](http://au.autodesk.com): search for 'Revit' in the Developer or Customization tracks.
  Here are a few classes held by us at last years AU with material providing additional information:
  - DE101-3 The Revit SDK Sample Smrgsbord
  - DE111-3 A Closer Look at the Revit Database with the Revit API
  - DE205-3 Enhancing Your Revit Add-InYou should be able to sign up and have access to this material.
- [Autodesk Discussion Groups](http://discussion.autodesk.com)
  > Revit Architecture > Revit API.
- [Autodesk Developer Network](http://adn.autodesk.com), the ADN web site, accessible to ADN members only, where you can:
  - Search DevNotes, which are Q & A pages.
  - Submit questions through DevHelp Online.

### 4. Looking forward to Revit 2010

We will be migrating and updating our training material for the Revit 2010 API in the coming months.
The first intermediate result will be the
[webcast training](http://www.adskconsulting.com/adn/cs/api_course_sched.php)
on April 29th.
If you are interested in learning more about what is coming for Revit 2010, please take a look at the recording of the recent
[Developer Days Online AEC webcast](http://adn.autodesk.com/adn/servlet/item?siteID=4814862&id=12554549&linkID=4901650).