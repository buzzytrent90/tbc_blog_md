---
post_number: "0537"
title: "Explode a DWG"
slug: "explode_dwg"
author: "Jeremy Tammik"
tags: ['csharp', 'elements', 'geometry', 'revit-api', 'transactions', 'views', 'windows']
source_file: "0537_explode_dwg.htm"
original_url: "https://thebuildingcoder.typepad.com/blog/0537_explode_dwg.html"
---

### Explode a DWG

After yesterday's very practical note on
[creating a pipe cap](http://thebuildingcoder.typepad.com/blog/2011/02/create-a-pipe-cap.html),
here is another slightly less practical note from Rudolf Honke of
[acadGraph CADstudio GmbH](http://www.acadgraph.de),
who picked up on Sami's
[comment](http://thebuildingcoder.typepad.com/blog/2011/01/subscribing-to-ui-automation-events.html?cid=6a00e553e1689788330148c78dc659970c#comment-6a00e553e1689788330148c78dc659970c) on
[subscribing to UI Automation events](http://thebuildingcoder.typepad.com/blog/2011/01/subscribing-to-ui-automation-events.html).
Sami asks:

**Question:** Is it possible to explode an imported DWG file using the API?

**Answer:** After showing the mechanism of
[invoking commands twice](http://thebuildingcoder.typepad.com/blog/2011/02/command-and-conquer-when-switching-views.html),
I do think this can be achieved.

Just like all other ribbon elements, also the 'Explode' buttons are accessible via UI Automation.

By the way, I have noticed that sometimes an ImportInstance does not contain all the geometry that is visible to the user, even using tiny DWGs consisting just of a few lines or curves.
If we explode it after import, we can access all the lines by subscribing to the DocumentChanged event and reading the event argument GetAddedElementIds data, because the explosion results are added to the database as individual new Revit elements.
This way, we can work around the issue of the missing elements and retrieve all the DWG data we need.

After collecting the geometrical data, we could roll back the transaction to undo the explosion.
If we are using UI Automation, though, we have already left our own command context, so we cannot manipulate the transaction programmatically.
Even in this case, though, we can still undo the operation by again using UI Automation to simulate a click on the Undo button.

#### UI Event Subscription Sample

Slightly less practical?

Who said anything about slightly less practical?

Here is a complete sample application demonstrating some aspects of event handling using UI Automation:

I am not the kind of person who says how easy it is to do something and then laughs maliciously in the background watching the others struggle.

In contrary, as an autodidact, I depend on easy-to-read information myself, and have no use for vague hints lacking concrete information, too.

To make my
[former UIAutomation hints](http://thebuildingcoder.typepad.com/blog/2011/01/automate-designoption-and-64-bit-add-in-templates.html) more
substantial, here is a Visual Studio project
[RevitUIAutoSamples.zip](zip/RevitUIAutoSamples2.zip)
containing two mechanisms:

- Subscribing to events such as ribbon bar clicks and Revit main window boundary changes.- Invoking a command twice + pressing the DWG explode button afterwards.

Please note that this is all very experimental.
For instance, the ribbon bar click subscription might hang if your ribbon item is a child of a SplitButton or a ListItem.

I am very glad if you find this useful!

Funnily enough, Kean is also talking about
[exploding AutoCAD objects using .NET](http://through-the-interface.typepad.com/through_the_interface/2011/02/exploding-autocad-objects-using-net.html) at
this very moment.