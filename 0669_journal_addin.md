---
ai_optimized: true
author: Jeremy Tammik
complexity_score: 1.2
content_type: qa
optimization_date: '2025-12-11T11:44:14.364544'
original_url: https://thebuildingcoder.typepad.com/blog/0669_journal_addin.html
post_number: 0669
reading_time_minutes: 2
series: general
slug: journal_addin
source_file: 0669_journal_addin.htm
tags:
- revit-api
title: Loading an Add-in With a Journal File
word_count: 324
---

### Loading an Add-in With a Journal File

I touched upon journal files a couple of times in the past, for example to
[load an external application](http://thebuildingcoder.typepad.com/blog/2009/07/journal-file-replay.html) and
automatically perform an action in the DocumentOpened event handler, or to
[implement a script automating IFC import](http://thebuildingcoder.typepad.com/blog/2010/07/ifc-import-and-conversion-journal-script.html).

The Revit SDK also includes the Journaling sample which demonstrates how an add-in can define what information goes into the journal file while it is being recorded, and how to use that data to repeat the add-in's external command functionality during replay.

However, several people had problems getting the Journaling sample to run in Revit 2012.
This is due to changes in the add-in loading behaviour introduced in the recent releases of Revit in connection with the shift from INI files to the add-in manifest.

Here is the quick summary of the solution, which is utterly simple but pretty hard to discover on your own:

To run this sample, make sure that the journal file, the add-in manifest (.addin) and the add-in assembly (.dll) are all in the same folder, otherwise it definitely will not work.
This is by design, to reduce the impact during regression testing: in this way, only the required add-ins in the journal folder will be registered and loaded by Revit.

This ties in very well with my current tendency to copy all add-in assemblies into the Revit add-in folder, where the add-in manifests have to be placed in order for Revit to find them.
As I mentioned discussing the
[add-in wizard updates](http://thebuildingcoder.typepad.com/blog/2011/10/product-and-add-in-wizard-updates.html#3),
this has the significant advantage of obviating the need to specify the full assembly path in the add-in manifest.