---
post_number: "1180"
title: "Upgrading Family Files Silently"
slug: "silent_upgrade"
author: "Jeremy Tammik"
tags: ['family', 'revit-api']
source_file: "1180_silent_upgrade.htm"
original_url: "https://thebuildingcoder.typepad.com/blog/1180_silent_upgrade.html"
---

### Upgrading Family Files Silently

Here is a recent case that I like and can share with you just as it is:

**Question:** My add-in loads a number of predefined families into the current document.
However, if many of them were saved in a previous version of Revit, a popup message saying the family file is being upgraded is displayed for each one.

Is there a way to load the families silently, so that the upgrade is still performed but the prompt is either not shown, or only shown once for the whole batch of family files?

Alternatively, if that is not possible, is there a way to find out if a model requires upgrading before loading it?

**Answer:** Thank you for your very pertinent query.
I like it very much that you care about that :-)

I can think of two completely different approaches.

**1.** Google for
[revit file updater](http://lmgtfy.com/?q=revit+file+updater).

Numerous results are available.

The
[file upgrader on Autodesk labs](http://labs.blogs.com/its_alive_in_the_lab/2011/07/updated-file-upgrader-for-revit-now-available.html) was
written by us, the ADN DevTech team, and is provided with full source code included as a learning tool.

**2.** Handle the pop-up messages, so that the user never sees them, using one of the techniques suggested for
[handling dialogues and failures](http://thebuildingcoder.typepad.com/blog/about-the-author.html#5.32).

**3.** Yes, you can
[determine the Revit version](http://thebuildingcoder.typepad.com/blog/2013/01/basic-file-info-and-rvt-file-version.html) used
to save a model.

The post describes two different approaches:
you can either use the documented and supported Revit API functionality provided by the BasicFileInfo class, or the unsupported approach parsing the RVT file yourself and extracting the desired information from its OLE document Structured Storage data.

**Addendum:** Please don't miss the
[continuation of this discussion](http://thebuildingcoder.typepad.com/blog/2014/07/upgrading-family-files-silently-part-2.html).