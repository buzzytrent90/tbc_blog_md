---
post_number: "0661"
title: "Getting Started with the Revit 2012 API"
slug: "getting_started_2012"
author: "Jeremy Tammik"
tags: ['csharp', 'elements', 'family', 'geometry', 'revit-api', 'schedules', 'views']
source_file: "0661_getting_started_2012.htm"
original_url: "https://thebuildingcoder.typepad.com/blog/0661_getting_started_2012.html"
---

### Getting Started with the Revit 2012 API

We already published 'Getting Started with the Revit API' guides for
[Revit 2009](http://thebuildingcoder.typepad.com/blog/2009/04/getting-started-with-the-revit-2009-api.html) and
[2010](http://thebuildingcoder.typepad.com/blog/2009/09/getting-started-with-the-revit-2010-api.html).
Here is a completely revamped and much expanded version for Revit 2012.
Actually, before even reading any further on this page, go to the
[Revit Developer Center](http://www.autodesk.com/developrevit)
and see what material is available there.
That is always the most up-to-date place to look.
Still, this overview may help your orientation as well:

1. [Setting up the Environment](#1)- [Where to Start](#2)- [For More Information](#3)- [.NET and C# Tutorials and Books](#4)

### Getting Started with the Revit 2012 API

#### 1. Setting up the Environment

The first step is to obtain and set up a development environment.
This includes the Revit product itself, the Revit API, the Revit SDK (Software Developer Kit), and the development environment.

**Revit Product:** You can download the latest version of released products from the Autodesk product page.
Go to [www.autodesk.com](http://www.autodesk.com), look under the 'Products' menu, 'Alphabetically.'
From each Revit product page, look under Product Download.
Following are examples of the links at the time of writing (Note: exact link may change):

- [Revit Architecture](http://usa.autodesk.com/adsk/servlet/index?siteID=123112&id=9262388)- [Revit Structure](http://usa.autodesk.com/adsk/servlet/index?siteID=123112&id=9339705)- [Revit MEP](http://usa.autodesk.com/adsk/servlet/index?siteID=123112&id=9262907)

Without an activation code, you can use Revit in viewing-only mode.

**Revit API:** The Revit API is provided by the .NET assemblies RevitAPI.dll and RevitAPIUI.dll, which are present in every Revit installation under

```
<Revit install>\Program
```

**Revit SDK:** The Revit SDK is included in every Revit installation.

The SDK installation can be launched from the 'Install Tools and Utilities' menu on the main page of the Revit installer.
There is another menu called 'Utilities' you see right after the product installation, which contains only the link 'Content Batch Utilities'.
Click on the 'Back to First Page' button to move back to the main page of the installer to install the SDK.

Alternatively, you can also find the SDK in the extraction folder, under:

```
<extraction folder>\support\SDK\RevitSDK.exe
```

If you have accepted the default location, it typically looks like this:

```
C:\Program Files\Autodesk\Revit XXX 2012\support\SDK\RevitSDK.exe
```

Note that the SDK may be updated by intermediate releases.
Make sure you check for the latest version either on the members-only ADN site or the public
[Revit Developer Center](http://www.autodesk.com/developrevit).

The SDK includes Revit API documentation.
The most important items are the Revit API help file RevitAPI.chm and the developer guide "Revit 2012 API Developer Guide.pdf".
It also includes many samples.

**Development Environment:** Microsoft Visual Studio (MSVS) 2010 for Revit 2012.
You may also use VS 2008 + SP1, but debugging a Revit 2012 add-in does not work in that environment.
All the SDK samples are provided for VS 2010.
The Visual Studio Express editions can also be used.

#### 2. Where to Start

(1) Watch the training video – The best way to start is to watch our DevTV recording, posted towards the bottom on the
[Revit Developer Center](http://www.autodesk.com/developrevit).

If you are new to programming in general as well, take a look at the
[My First Revit Plug-in](http://www.autodesk.com/myfirstrevitplugin) training material.

You can also have a look at last year's
[Revit 2011 API webcast recording](http://download.autodesk.com/media/adn/RevitAPI_20May2010.zip).

It covers the basics of the Revit API since 2011.
Note that the Revit API underwent a major renovation in 2011.
The webcast materials include the recording itself, the Powerpoint presentation, and training labs code.
You can also find it on the ADN web site, under the Revit knowledgebase section, or from the Developer Center
[Training Course Schedule](http://www.adskconsulting.com/adn/cs/api_course_sched.php) and
[Webcast Archive](http://www.adskconsulting.com/adn/cs/api_course_webcast_archive.php).

The latest training labs for Revit 2012 are posted on the
[Revit Developer Center](http://www.autodesk.com/developrevit).

This year's [Revit 2012 API webcast](http://download.autodesk.com/media/adn/Autodesk_Revit_2012_API_Update.zip) covers several major API enhancements.

If you are interested in Family API, there is a
[Revit Family API webcast recording](http://download.autodesk.com/media/adn/Revit_2010_Family_API_Webcast-July2009.zip), too.

(2) Work through the latest hands-on training labs, which are available from the
[Revit Developer Center](http://www.autodesk.com/developrevit)
(scroll down toward the bottom).

(3) Try implementing and running the HelloRevit sample by yourself, for example following the steps described in Chapter 2, Walkthroughs, in the developer guide.
Go through the Labs code, and look at some Revit SDK samples.
The following SDK samples will be a good starting point:

- RevitCommands- Fire Rating- Viewers/ElementViewer- CreateBeamsColumnsBraces

#### 3. For More Information

- [The Building Coder](http://thebuildingcoder.typepad.com/blog) Revit API blog, also accessible through
  [blogs.autodesk.com/thebuildingcoder](http://blogs.autodesk.com/thebuildingcoder), and especially its
  [Getting Started](http://thebuildingcoder.typepad.com/blog/getting_started/) category.- [AEC DevCamp 2010](http://download.autodesk.com/media/adn/Autodesk_AEC_DevCamp_2010.zip):
    AEC DevCamp is a bi-annual event. This link includes the comprehensive set of materials up to the Revit 2011 API and is of interest to both beginners and advanced developers.
    Look under Revit Track 1 (Beginner) and Track 2 (Advanced), contained in the folders starting with 1.x or 2.x.- [Autodesk University](http://au.autodesk.com): AU 2007/2008/2009/2010 –
      go to the 'Online Classes' tab.
      Search for 'Revit API' and/or under the Developer and Customization tracks.
      You should be able to sign up and have access to those materials.
      For example, here are a few classes from AU2010 presented by Revit API engineers and the Developer Technical Services team whose materials give you additional, more in-depth information:

- [CP319-1](http://au.autodesk.com/?nd=class&session_id=6692)
  Analyzing Building Geometry using the Revit API – Scott Conover- [CP327-1](http://au.autodesk.com/?nd=class&session_id=6843)
    Using Dynamic Model Update in the Revit API – Scott Conover- [CP234-2](http://au.autodesk.com/?nd=class&session_id=7246)
      Revit 2011 Programming Optimization – Jeremy Tammik- [CP228-2](http://au.autodesk.com/?nd=class&session_id=6943)
        Optimal Use of Revit 2011 Programming Features – Jeremy Tammik- [CP316-1](http://au.autodesk.com/?nd=class&session_id=6905)
          Presenting Analysis Data in Revit 2011 – Harry Mattison

There is much more.
Please search the AU site.- [Autodesk Revit API Discussion Group](http://forums.autodesk.com/t5/Autodesk-Revit-API/bd-p/160):
  go to [forums.autodesk.com](http://forums.autodesk.com) and select Revit > Revit API.- [ADN members-only web site](http://adn.autodesk.com): provided by the Autodesk Developer Network.

- Search DevNotes, whitepapers, and Q&A pages- Submit questions through DevHelp Online

If you are currently not an ADN member and interested in joining, please refer to
[becoming an ADN member](http://www.autodesk.com/joinadn).

#### 4. .NET and C# Tutorials and Books

In a followup to the note on
[removing an add-in registered by the wizard](http://thebuildingcoder.typepad.com/blog/2011/10/product-and-add-in-wizard-updates.html#4),
Bryan Thatcher asks:

**Question:** I have programming experience starting with Qbasic, VBA, VB5, some C and C++, but I'm new to C# and the added complexity of the Revit API has been a challenge.
I have learned a lot from your blog, but can you recommend a comprehensive resource for the Revit API as it relates to C#?
All of the material I've found so far has been fragmented.

**Answer:** I can suggest very little regarding books on this topic.
The only books I read myself are novels :-)

I mentioned this
[.NET learning resource](http://thebuildingcoder.typepad.com/blog/2009/07/a-net-language-learning-resource.html) back in 2009, and some
[free .NET development and architecture e-books](http://thebuildingcoder.typepad.com/blog/2010/10/cairo-and-free-net-books.html) in 2010.

I would not worry much about the fragmentation, though.
.NET and C# is one thing, the Revit API another.

I would suggest learning C# independently of Revit using any free online tutorial you like, working through the Revit getting started materials bit by bit at the same time, and Bob's your uncle.
The DevTV tutorials are extremely easy to follow, require only minimal .NET and programming knowledge, and familiarise you with it step by step as you go along.

Look at the material listed above, and for more, there is the whole
[getting started](http://thebuildingcoder.typepad.com/blog/getting_started) category of this blog.