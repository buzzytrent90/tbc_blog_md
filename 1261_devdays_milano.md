---
post_number: "1261"
title: "DevDays in Milano, Links, Textures, UR4 vs R2"
slug: "devdays_milano"
author: "Jeremy Tammik"
tags: ['elements', 'references', 'revit-api', 'views']
source_file: "1261_devdays_milano.htm"
original_url: "https://thebuildingcoder.typepad.com/blog/1261_devdays_milano.html"
---

### DevDays in Milano, Links, Textures, UR4 vs R2

Today we are holding the last Western European DevDays conference in Milano before the winter break.

The
[Milano meetups](http://thebuildingcoder.typepad.com/blog/2014/12/milano-meetups-and-my-new-nfc-business-card.html) last
night met with great interest and enthusiasm, so I will probably be returning here next year to present and conduct workshops on the
[Autodesk View and Data API](http://thebuildingcoder.typepad.com/blog/about-the-author.html#5.46) in
more depth.

Right now, we are in the middle of Jim's morning session:

![DevDays conference in Milano](img/2014-12-17_devdays_milano.jpeg)

Independently of that, here are a few other miscellaneous topics that cropped up in the past few days:

- [Loading and unloading Revit links](#2)
- [Currently no programmatic texture assignment](#3)
- [Revit 2015 R2 and UR API updates](#4)

#### Loading and Unloading Revit Links

**Question:** How can I 'reload from' or 'remove' a link?

All I can see is ReloadLatest.

**Answer:** I would suggest starting by looking at the Revit API help file RevitAPI.chm.
Have you done that?

I looked at the Document.ReloadLatest method that fetches changes from central (due to one or more synchronisations with central) and merges them into the current session.

That has to do with central files, not links.

Searching a bit further for 'link', 'load', etc., I discovered the RevitLinkType.Load method that loads or reloads the Revit link from its currently-stored location.

It also provides the RevitLinkType.Unload method that unloads the Revit link.

Could that be what you are looking for?

This functionality was introduced in the
[Revit 2014 API](http://thebuildingcoder.typepad.com/blog/2013/04/whats-new-in-the-revit-2014-api.html);
search for 'enhancements to interactions with links'.

Here is another aspect of
[removing RVT links](http://thebuildingcoder.typepad.com/blog/2014/06/technical-summit-day-1-and-removing-rvt-references.html#7).

You might also want to take a look at the
[link API additions](http://thebuildingcoder.typepad.com/blog/2014/04/whats-new-in-the-revit-2015-api.html#4.03) in the
[Revit API 2015 news](http://thebuildingcoder.typepad.com/blog/2014/04/whats-new-in-the-revit-2015-api.html).

#### Currently No Programmatic Texture Assignment

I recently discussed
[DirectShape texture assignment](http://thebuildingcoder.typepad.com/blog/2014/11/directshape-versus-families-category-and-texture.html#4),
leading to this obvious follow-up query:

**Question:** Thanks for the answer, it was very helpful!

I used the TessellatedFace.MaterialId method and it works well.

I now have a few follow-up questions: How can I rotate and scale a material (texture) on a face? Is there a way to map a texture to a face, like mapping texture coordinates [u,v] to face coordinates [x,y,z]? How does Revit map a material (texture) to a face?

**Answer:** In the UI, you can affect the texture mapping by selecting the material asset in the Materials dialogue, clicking the Appearance tab, clicking the pull-down to the right of the Image display, clicking Edit Image, and editing the properties of that image.

I am sorry to report that there is currently no API support for setting these properties, just as there is no API support for editing visual material properties in general.

Unfortunately, material manipulation is currently programmatically inaccessible.

By the way, talking about DirectShape elements, you absolutely have to check out
[DirectObjLoader](https://github.com/jeremytammik/DirectObjLoader),
my new
[OBJ to DirectShape import utility add-in](http://thebuildingcoder.typepad.com/blog/2014/12/devdays-github-stl-and-obj-model-import.html#3).

#### Revit 2015 R2 and UR API Updates

The Revit 2015 R2 subscription release included both new product functionality and some API enhancements, which led to the
[updated SDKs for Revit 2015 R2 and UR4](http://thebuildingcoder.typepad.com/blog/2014/10/updated-sdks-for-revit-2015-r2-and-ur4.html") and
the following query:

**Question:** How do I tell which SDK to choose for download?

**Answer:** Which SDK you choose is pretty irrelevant.
It only contains documentation and sample code.

The Revit API .NET assembly DLLs provide the actual API, and they are all included with the Revit installation.

There are only some minute enhancements in Revit 2015 R2 over Revit 2015, e.g. access to the Workflow.Create and modification methods, as discussed in the Revit API discussion forum threads on
[worksets](http://forums.autodesk.com/t5/revit-api/worksets/m-p/5360275) and
[R2 vs. UR4](http://forums.autodesk.com/t5/revit-api/r2-vs-ur4/m-p/5439929).

All the rest is identical.