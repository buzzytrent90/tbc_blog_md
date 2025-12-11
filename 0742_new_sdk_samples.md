---
ai_optimized: true
author: Jeremy Tammik
complexity_score: 2.8
content_type: tutorial
optimization_date: '2025-12-11T11:44:14.500944'
original_url: https://thebuildingcoder.typepad.com/blog/0742_new_sdk_samples.html
post_number: '0742'
reading_time_minutes: 3
series: general
slug: new_sdk_samples
source_file: 0742_new_sdk_samples.htm
tags:
- references
- revit-api
- transactions
- views
title: New Revit 2013 SDK Samples
word_count: 609
---

### New Revit 2013 SDK Samples

Revit 2013 has been released, and I presented an overview of the
[Revit 2013 API](http://thebuildingcoder.typepad.com/blog/2012/03/revit-2013-and-its-api.html) two
days ago.

As always, the use of most of the API features is demonstrated by various SDK samples.
For a better understanding of the use and contents of the Revit SDK in general, please refer to the
[getting started](http://thebuildingcoder.typepad.com/blog/2011/10/getting-started-with-the-revit-2012-api.html) and
[self-preparation](http://thebuildingcoder.typepad.com/blog/2012/01/preparing-for-a-hands-on-revit-api-training.html) guides.

Here is an overview of the new samples:

- [ModelessForm\_ExternalEvent and ModelessForm\_IdlingEvent (ModelessDialog)](#1)- [ProgressNotifier (Events)](#2)- [RoutingPreferenceTools](#3)- [UIAPI](#4)- [WorkThread (MultiThreading)](#5)

These samples are all related to the new
[add-in integration](http://thebuildingcoder.typepad.com/blog/2012/03/revit-2013-and-its-api.html#2) functionality,
except for the RoutingPreferenceTools which obviously demonstrate some of the
[MEP related enhancements](http://thebuildingcoder.typepad.com/blog/2012/03/revit-2013-and-its-api.html#3).

By the way, the 'VSTA Samples' folder was renamed to 'Macro Samples' since
[VSTA](http://en.wikipedia.org/wiki/Visual_Studio_Tools_for_Applications) was
replaced by the open source
[SharpDevelop](http://en.wikipedia.org/wiki/SharpDevelop) IDE.

#### ModelessForm\_ExternalEvent and ModelessForm\_IdlingEvent

Both of these display and show how to interact with a modeless form.
One way to do this is to use the
[Idling event](http://thebuildingcoder.typepad.com/blog/idling) initially provided in Revit 2012,
which we have discussed so much and in such depth in the past.

The ModelessForm\_IdlingEvent sample should clarify many of the issues we dealt with, and is also related the material presented by Arnošt Löbel in his Autodesk University 2011 class
[CP5381](http://au.autodesk.com/?nd=event_class&session_id=9879&jid=1763185) on
asynchronous interactions and managing modeless UI.

The ModelessForm\_ExternalEvent sample demonstrates an easier way to implement this interaction using the new external event interface.

#### ProgressNotifier

The ProgressNotifier sample displays progress information for an action in a stack data structure for easier analysis. It demonstrates how to subscribe to the ProgressNotify related events, access properties in the event handler arguments, and organize the subtransaction progress information into a stack.

#### RoutingPreferenceTools

The RoutingPreferenceTools sample provides a number of MEP pipe routing preference tools.

This sample contains three commands, one for analysis and reporting purposes, two for importing and exporting routing preferences to XML:

- Routing Preference Analysis: Analyze the routing preferences of a given pipe type to check for common problems, using the routing preferences API to look at all rules and criteria for a given PipeType.- Routing Preference Builder with its two commands CommandReadPreferences and CommandWritePreferences: Set pipe type, fitting, and routing preferences in a project from data in an XML file and export these preferences to XML for archival, documentation, and collaboration purposes, allowing a user to work with routing preference data in a shareable XML format suitable for reuse in a wide variety of BIM management environments.

#### UIAPI

The UIAPI sample demonstrates a number of the new
[add-in integration API features](http://thebuildingcoder.typepad.com/blog/2012/03/revit-2013-and-its-api.html#2) that
I already listed, including embedding a Revit view as WPF control inside its own dialogue, the new drag and drop API, and the Options dialogue support for custom extensions using arbitrary WPF components.
This sample was also shown at the DevDays 2011 conferences.

#### WorkThread

The WorkThread sample demonstrates utilizing the Idling event in a multi-threaded application to communicate with the Revit API from an external work thread.