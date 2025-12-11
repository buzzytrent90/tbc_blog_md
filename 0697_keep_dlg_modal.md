---
ai_optimized: true
author: Jeremy Tammik
complexity_score: 1.6
content_type: qa
optimization_date: '2025-12-11T11:44:14.417000'
original_url: https://thebuildingcoder.typepad.com/blog/0697_keep_dlg_modal.html
post_number: 0697
reading_time_minutes: 3
series: general
slug: keep_dlg_modal
source_file: 0697_keep_dlg_modal.htm
tags:
- elements
- python
- revit-api
- selection
title: Keep Dialogues Modal!
word_count: 619
---

### Keep Dialogues Modal!

Today is the
[winter](http://en.wikipedia.org/wiki/Winter_solstice)
[solstice](http://en.wikipedia.org/wiki/Solstice),
the shortest day of the year, at least in the parts of the world I normally frequent.
Just as the sun reached the end of its descent, we finally reached the end of our conference tour, here in the UK.
I am exhausted and very much looking forward to returning home.
There will be only rather a short time left to prepare for Christmas and my children's visit when I arrive.

Meanwhile, here are a couple of items that showed up in my in-box:

- [Village BIM](#1)- [Dynamically served WebGL content using the FBX Python SDK](#2)- [Keeping dialogues modal](#3)

#### Bienvenu a Village BIM!

A new BIM and thus Revit related blog in French language has been launched by group of French AEC technical specialists, Alexandre Mihalache, Emmanuel Di Giacomo, Julien Drouet, Olivier Bayle and Stephane Balmain:
[Village BIM](http://villagebim.typepad.com/).

#### Dynamically Served WebGL Content Using the FBX Python SDK

Are you interested in FBX?
Want to be impressed by the power of Python?
How about dynamically serving WebGL content over the web from an FBX file?
This can be easily achieved using the FBX Python SDK, as demonstrated by this
[description including a short video](http://area.autodesk.com/blogs/chris/serving_webgl_content_using_the_fbx_python_sdk).

#### Keep Dialogues Modal

Here is a piece of very important advice on the topic of modeless versus modal and asynchronous communication with Revit by Mario Guttman, AIA, LEED AP, Senior Associate and Design Applications Research Leader at
[Perkins+Will](http://www.perkinswill.com/):

I want to share something with you which you may find interesting.
A lot of my work entails presenting the user with a dialog box and then taking action based on their choices on the dialog box.
Recently I have been using this pattern to drive actions that use the on-screen selection methods 'Autodesk.Revit.UI.Selection.PickObjects' and 'Autodesk.Revit.UI.Selection.PickElementsByRectangle'.
The problem is that I want to leave my dialog box during the pick process and then, optionally, return to it when the user has finished making selections.

For a while I was using a non-modal dialog but got some odd behaviour and then learned at AU that this was not a good idea at all unless I was going to really manage the threads with an Idling event.
I did some research around this issue and even went to Arnošt Löbel's class
**[CP5381](http://au.autodesk.com/?nd=class&session_id=9879)** on
Asynchronous Interactions and Managing Modeless UI on this topic.
It's all pretty complicated for us mere mortals.

What I realized, however, is that all of the discussion I was hearing was about ways of keeping a non-modal dialog visible while the user was working on the screen. I don't really need that, in fact I'd rather not have the dialog in the way. I could close the dialog before taking action but then I would have to remember all the settings and code the action in a separate file. The solution that I found is to just keep the dialog form alive but just not open. This means that all of the settings, commands, etc. can be kept in the code file associated with the dialog, even though I am calling it from outside. This also makes it easy to reopen the dialog if needed after the user is done interacting with the model.

Here is
[InterfaceStudy.zip](zip/2011-12-14_InterfaceStudy.zip) containing a simple example that illustrates this idea.
Jeremy adds: as said, this is really relevant advice, so please pay heed!