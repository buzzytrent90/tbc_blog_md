---
ai_optimized: true
author: Jeremy Tammik
complexity_score: 1.7
content_type: qa
optimization_date: '2025-12-11T11:44:14.084183'
original_url: https://thebuildingcoder.typepad.com/blog/0514_ui_automation_events.html
post_number: '0514'
reading_time_minutes: 2
series: general
slug: ui_automation_events
source_file: 0514_ui_automation_events.htm
tags:
- family
- revit-api
- rooms
- walls
- windows
title: Subscribing to UI Automation Events
word_count: 390
---

### Subscribing to UI Automation Events

Rudolf Honke of
[acadGraph CADstudio GmbH](http://www.acadgraph.de) recently
presented his results of exploring the Revit ribbon internals using UISpy and
[driving Revit using UI Automation](http://thebuildingcoder.typepad.com/blog/2011/01/ribbon-spying-and-ui-automation.html).

He sent an additional note to point out that this rapidly leads to many other very interesting possibilities, such as the following:

[Subscribing to UI Automation events](http://msdn.microsoft.com/en-us/library/ms752286.aspx) could
easily enable an application to be notified whenever a user...

- Creates a new Wall, Room, etc.- Picks into the property list- Hits the ribbon bar- Is about to open or save a Revit file (okay, this could possibly be achieved better in another way...)- Resizes the Revit app window- And so on ...

The events could be handled using a stand-alone application or a Revit add-in, registering the listener in the Revit external application OnStartup method...

Thinking about using it with the OnStartup method, I remember the fact that some parts of the Revit UI may not be generated completely at this time.
In this method, you usually create your buttons, so the ribbon bar may be in rebuilding process.
Also, the ribbon bar could be invisible or disabled if there is no opened project file.

Some time ago, I developed some functions to check the ribbon bar state before pressing any tab header or button.
One of these functions maximizes the ribbon bar if it is minimized.

By the way, if you just want to be notified if the app window is resized or moved, you could also use a BackgroundWorker which checks the Window.Size ten times a second.
Using old (but fast) Windows API methods via P/Invoke, this "polling" can be done with up to 50 Hz or more – ups, another theme for posting...

Hey, you could combine the notifyMeIfRoomIsToBeCreated listener with an OnDocumentOpened event, so the 'Create Room' button must be accessible (but you would have to check whether it is a family document)...

Anyway, just think of the possibilities this can offer!

A number of these features have actually been requested in the past, and nobody provided the UI Automation expertise to answer them ... until now.

Once again, many thanks to Rudolf for his many ideas and these valuable pointers!