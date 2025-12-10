---
post_number: "1012"
title: "No Command Launching from Dockable Panel"
slug: "dockable_api_access"
author: "Jeremy Tammik"
tags: ['csharp', 'elements', 'parameters', 'revit-api', 'rooms', 'transactions', 'views', 'windows']
source_file: "1012_dockable_api_access.htm"
original_url: "https://thebuildingcoder.typepad.com/blog/1012_dockable_api_access.html"
---

### No Command Launching from Dockable Panel

Here is an issue that has come up a few times now and is worthwhile clarifying: how to operate on a Revit document from a dockable panel.

The dockable panel UI API framework was introduced in Revit 2014.
Its use is demonstrated by the DockableDialogs SDK sample, including modeless dialog design and external events, and also by the
[simpler dockable panel sample](http://thebuildingcoder.typepad.com/blog/2013/05/a-simpler-dockable-panel-sample.html).

The fact that a dockable panel lives in a modeless context is apparently confusing for some.
Since the panel is active at all times, you are not inside a Revit external command handler and therefore not in a valid Revit API context.

This leads to queries like this:

**Question 1:** I'm probably overlooking something, but how do you call an IExternalCommand from a dockable window.
I guess this is possible, right?

**Answer:** No, sorry.
See below.

**Question 2:** I create my own dockable dialog.
It loads a table from an external data source and displays that information in a tree view.
That works fine.

Now I want to add a button to that form to retrieve a selected Revit element and attach that to one of the tree nodes (cf. discussion group
[comment](http://thebuildingcoder.typepad.com/blog/2013/05/a-simpler-dockable-panel-sample.html?cid=6a00e553e16897883301910445b4b2970c#comment-6a00e553e16897883301910445b4b2970c)).
In fact, I just want to keep it simple and copy some tree node data to a Revit shared parameter.

I implemented the code to determine the right tree node.

However, how can I get this button to connect to the current active Revit document?
Am I doing something wrong here?
Is that possible with this at all?

I noticed an outdated AUGI discussion thread on
[accessing a Revit document from a Windows form](http://forums.augi.com/showthread.php?98991-Windows-Form-Pass-to-Revit-Command) that
seems to implicate that I need to pass the Application object to the XAML form.
However, the explanation in that answer is not enough for me to solve my problem.

Now I also discovered that starting a transaction from an external application running outside of a valid Revit API context is not allowed.

How can I solve this, please?

**Answer:** As mentioned in the introduction above, a dockable panel is similar to a modeless form, which is similar to an external Windows application, which is basically the same as any other process outside of Revit: it has not been registered with the Revit API, Revit is completely unaware of its existence, and you can make no use whatsoever of the Revit API from that process.

Modeless access to the Revit API boils down to accessing the Revit application from an arbitrary external application.

The Revit API does not provide any direct access to the Revit application, except in the valid Revit API context situations defined by the various Revit API events.
Basically, the complete Revit API is based entirely on call-back methods.
There is no API support for driving Revit from an external application, whether .NET, WPF, or anything else.

Being in a modeless context, you have no valid Revit API context.
You can subscribe to the Idling event to obtain one, and the available dockable dialogue samples demonstrate how to do so.

Workarounds exist, have been frequently described, both here and elsewhere, and numerous previous discussions address various aspects of this situation.

The nicest and most recent complete sample I know of is my
[cloud-based round-trip 2D Revit model editor](http://thebuildingcoder.typepad.com/blog/2013/04/room-editor-project-overview-and-couchdb-configuration.html) project.

Here are some other recent mentions of the topic:

- [Display real-time interactive element properties](http://thebuildingcoder.typepad.com/blog/2013/05/effortless-extensible-storage.html#7)
- [Behind the scenes of the NBS Revit add-in](http://thebuildingcoder.typepad.com/blog/2013/06/behind-the-scenes-of-the-nbs-revit-add-in.html)

- [Controlling Revit from modeless dialogues](http://thebuildingcoder.typepad.com/blog/2013/06/behind-the-scenes-of-the-nbs-revit-add-in.html#3)

- [Batch mode data extraction](http://thebuildingcoder.typepad.com/blog/2013/08/the-revit-server-rest-api.html)

For more background information, please look at the following blog categories and keyword search results:

- [Idling](http://thebuildingcoder.typepad.com/blog/idling)
- [External](http://thebuildingcoder.typepad.com/blog/external)
- [Driving Revit from outside](http://lmgtfy.com/?q=revit+api+driving+outside)
- [Modeless Revit API access](http://lmgtfy.com/?q=revit+api+modeless)

Happy exploring!