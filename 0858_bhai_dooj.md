---
ai_optimized: true
author: Jeremy Tammik
complexity_score: 2.3
content_type: qa
optimization_date: '2025-12-11T11:44:14.747150'
original_url: https://thebuildingcoder.typepad.com/blog/0858_bhai_dooj.html
post_number: 0858
reading_time_minutes: 2
series: general
slug: bhai_dooj
source_file: 0858_bhai_dooj.htm
tags:
- elements
- revit-api
- views
title: Happy Bhai Dooj!
word_count: 472
---

### Happy Bhai Dooj!

Happy
[Bhai Dooj](http://en.wikipedia.org/wiki/Bhau-beej) to you!

![Happy Bhai Dooj!](img/happy-bhai-dooj.jpg)

This greeting comes from my colleague Sandeep Kumar in
[Bangalore](http://en.wikipedia.org/wiki/Bangalore), who explains:

Bhai Dooj is a festival of prayers from sister to brother, brother's protection for her sister.
May we all celebrate this Bhai Dooj with even more love and protection for our sisters and brothers.
Best wishes on this Bhai Dooj.

Thus fortified, let us turn to a Revit API issue, based on this excerpt from a useful little chat I had yesterday that might be of general use:

#### Implementing a Single Command for Multiple Buttons

**Question:** My external application displays a large number of all kinds of ribbon items.
Now I thought I might implement one single handler class for all of the external commands these items can trigger, instead of creating a separate one for each.
I would like to create only one CommandHandler class derived from IExternalCommand, and decide which function is required in its Execute method.
The problem I have is that the ExternalCommandData instance passed in does not provide any information about the source of the event, so there is no way of knowing which RibbonItem was clicked.
Is there a way to get that information, e.g. retrieve the name of the button clicked or something?

**Answer:** A good idea. It makes perfect sense. Unfortunately, no, this is not possible.

At least, the Revit API does not give you this access.
You might possibly have more luck using .NET
[UI Automation](http://thebuildingcoder.typepad.com/blog/automation).
It probably provides
[UI Automation events](http://thebuildingcoder.typepad.com/blog/2011/01/subscribing-to-ui-automation-events.html) that
can let you find out what button was clicked.

Using the standard Revit API functionality, you really do have to implement a separate command for each button.
You can easily funnel them all into the same handler yourself afterwards, if you like.
That would achieve almost what you want.
Implement a separate command for each button, determine which button was clicked, and then funnel all calls into the same command handler method.
The handler method can check the source of the event and branch out again appropriately.

**Addendum:** In his comment below, Rudolf Honke points to the very interesting discussion thread on
[which pushbutton caused the ExternalCommand](http://forums.autodesk.com/t5/Autodesk-Revit-API/Which-pushbutton-caused-the-ExternalCommand/td-p/3698908) where
[Revitalizer](http://forums.autodesk.com/t5/user/viewprofilepage/user-id/1103138) and
[ollikat](http://forums.autodesk.com/t5/user/viewprofilepage/user-id/774564) expound
on various alternative solutions, e.g. subscribing to the UIElementActivated event, creating commands at runtime and implementing one single base class inheriting IExternalCommand and other command classes derived from it.