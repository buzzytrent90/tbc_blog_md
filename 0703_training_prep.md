---
post_number: "0703"
title: "Preparing for a Hands-on Revit API Training"
slug: "training_prep"
author: "Jeremy Tammik"
tags: ['csharp', 'revit-api', 'views']
source_file: "0703_training_prep.htm"
original_url: "https://thebuildingcoder.typepad.com/blog/0703_training_prep.html"
---

### Preparing for a Hands-on Revit API Training

**Question:** I have registered for a hands-on Revit API Training, but I have little experience in programming, and almost none in .NET or C#.
How can I prepare for it to make best use of this opportunity?

**Answer:** This is actually all described in the discussion on
[getting started with the Revit 2012 API](http://thebuildingcoder.typepad.com/blog/2011/10/getting-started-with-the-revit-2012-api.html).
Still, let's highlight some of the points and put them in proper order for you.

The first of the following items is of interest for anyone just getting started with programming, .NET, or C#.
All the rest are for those with some prior .NET programming experience:

1. Work through some .NET programming tutorial.
   There are lots on free online .NET tutorials available.
   I mentioned a few in the
   [getting started overview](http://thebuildingcoder.typepad.com/blog/2011/10/getting-started-with-the-revit-2012-api.html#4).
   Some more advanced and interesting hints are listed in the
   [C# and .NET little wonders](http://thebuildingcoder.typepad.com/blog/2010/11/c-and-net-little-wonders.html).
   Even spending just a day working through such a tutorial will make an enormous difference.- Install the Revit SDK, available from the
     [Revit Developer Center](http://www.autodesk.com/developrevit),
     and
     [set up the environment](http://thebuildingcoder.typepad.com/blog/2011/10/getting-started-with-the-revit-2012-api.html#1).- Read the first chapter of the Developer Guide "Revit 2012 API Developer Guide.pdf",
       [Welcome to the Revit Platform API](http://wikihelp.autodesk.com/Revit/enu/2012/Help/API_Dev_Guide/0001-Introduc1/0002-Welcome_2), also available online.- Work through the
         [My First Revit Plugin](http://thebuildingcoder.typepad.com/blog/2011/05/my-first-revit-plug-in.html) tutorial,
         also available for
         [VB](http://thebuildingcoder.typepad.com/blog/2011/11/my-first-revit-plug-in-in-vb.html),
         both again from the
         [Revit Developer Center](http://www.autodesk.com/developrevit).
         The developer center also provides further introductory training videos, the
         [DevTV recordings](http://thebuildingcoder.typepad.com/blog/2010/10/revit-2011-devtv.html).
         Equivalently, work through the hands-on tutorials in the developer guide Chapter 2,
         [Getting Started](http://wikihelp.autodesk.com/Revit/enu/2012/Help/API_Dev_Guide/0001-Introduc1/0012-Getting_12).- Install the
           [RevitLookup](http://thebuildingcoder.typepad.com/blog/2010/05/revitlookup-update.html) add-in,
           included in the Revit SDK.
           This really is *the* most important development tool for interactively exploring the internals of the Revit database, and of great interest for advanced end users as well, actually.- Install the
             [RvtSamples](http://thebuildingcoder.typepad.com/blog/2008/09/loading-sdk-sam.html) external
             application add-in, which is also provided as part of the Revit SDK.
             It provides an interface to load and run all of the hundred-plus other SDK samples, instead of installing them individually, which makes the exploration and debugging of their functionality *much* more accessible.
             Here are some pointers to
             [more information on RvtSamples](http://thebuildingcoder.typepad.com/blog/2011/07/revit-api-training-preparation.html).

All of these steps are described in more detail in numerous other places.
Many of them are pointed to from the
[getting started guide](http://thebuildingcoder.typepad.com/blog/2011/10/getting-started-with-the-revit-2012-api.html).

The blog also defines a dedicated category providing a collection of all the
[getting started material](http://thebuildingcoder.typepad.com/blog/getting_started).

Some additional notes on these issues are provided by a previous take on this very issue of
[preparing for a hands-on training](http://thebuildingcoder.typepad.com/blog/2011/07/revit-api-training-preparation.html).
You might also be interested in this overview of the
[training material](http://thebuildingcoder.typepad.com/blog/2011/07/revit-api-training-preparation.html) that
we use in our own classes.

This should cover everything of importance up-front pretty thoroughly :-)

Good luck getting started, don't give up, and above all **have fun!**

#### Dances With Elephants

I want to point out this useful new developer related blog with the most awesome name of the year:
[Dances With Elephants](http://dances-with-elephants.typepad.com/blog) by
Jim Quanci, Director of the Autodesk Developer Network ADN, on how small companies can leverage big ones to build their business, e.g. by using the Autodesk Revit API to create and provide your add-in functionality to a large global audience.

Kean just published some
[background info and a short description](http://through-the-interface.typepad.com/through_the_interface/2012/01/a-new-autodesk-blog-on-partnering-dances-with-elephants.html) of
it in slightly greater depth.
Enjoy!