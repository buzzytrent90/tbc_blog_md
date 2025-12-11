---
ai_optimized: true
author: Jeremy Tammik
complexity_score: 1.4
content_type: documentation
optimization_date: '2025-12-11T11:44:14.089334'
original_url: https://thebuildingcoder.typepad.com/blog/0517_ui_auto_switch_view.html
post_number: '0517'
reading_time_minutes: 4
series: views
slug: ui_auto_switch_view
source_file: 0517_ui_auto_switch_view.htm
tags:
- elements
- geometry
- parameters
- revit-api
- views
- windows
title: Further Ideas for Using UI Automation and Galileo
word_count: 719
---

### Further Ideas for Using UI Automation and Galileo

Rudolf Honke of
[acadGraph CADstudio GmbH](http://www.acadgraph.de) recently
presented his results of exploring the Revit ribbon internals using UISpy,
[driving Revit using UI Automation](http://thebuildingcoder.typepad.com/blog/2011/01/ribbon-spying-and-ui-automation.html), and
[subscribing to UI Automation events](http://thebuildingcoder.typepad.com/blog/2011/01/subscribing-to-ui-automation-events.html).
Here is another of his ideas of what can be achieved using this technique, plus some thoughts on the
[Galileo project](#2) that I mentioned on Friday:

#### Automating Switch Windows

This screen snapshot shows the list of opened Revit files and views in the 'Switch Windows' dropdown list:

![Switch Windows dropdown list](img/switch_windows.png)

You could easily use UI Automation to set the active view or active project file by performing the following steps:

- Press the 'Switch Windows' button.- Wait some milliseconds.- Browse the popup list.- Select the entry matching your desired file and view name.- Invoke the list entry using the appropriate ControlPattern.

This is frequently requested wish list item.

Ah, and of course another point to be aware of:
Pressing buttons may cause warning, error or other functional dialogs to appear, so you need one of the known workarounds to suppress or auto-click them.

There is yet another point to add:
The user can make the file extensions disappear in Windows explorer; this will also affect the Revit application title.
I haven't tested whether the list of opened views is affected, too.
To be safe, one must take this into account when comparing the file name strings.

#### Thoughts on Galileo

When reading your last blog entry about
[Galileo](http://thebuildingcoder.typepad.com/blog/2011/01/bim-apps-project-galileo-and-piotrs-plug-in.html), I remembered this website
[www.procedural.com](http://www.procedural.com) about creating rule-based models on a city scale.
It includes a section about rebuilding Pompeii, which is a really good example to show the approach.
If you are able to reduce the "rules" of Roman architecture to some algorithms, the program will suggest a bunch of possible results.
It's like a holodeck simulation in Star Trek.
Think about playing with the parameters, e.g. "what would it look like if glass would have been much cheaper at this time", "how many people could have lived there if the 'insulae' would have had an additional floor", etc.
Also think about ways of layout optimization...

In fact, this is analogous to Revit in the sense that everything is parameterized.
In addition, this program also makes suggestions.
In the case of procedural.com, there are input parameters and rules; after calculation, there is a result (To be honest, I don't know this program, but perhaps there could be x results).
The resulting geometry is also parameterized, so you can modify façade patterns, move streets, etc. even after they've been created.

Both Revit and Galileo do not have the ability to suggest some geometry, they also do not offer geometric variations.
What the user can do is to assign DesignOptions, but he must do all the work on his own.

I think, Galileo could target a similar direction as Revit in the sense that a Revit project preserves the object relations between its elements, and Galileo seems just port this concept to a larger scale.

I guess that there is a mixture of intelligent and un-intelligent objects like in Revit because also Galileo seems to be able to import un-parameterized objects.
Loosing object intelligence is the price that must be paid for interoperability with other CAD products.

Creating procedural models leads to an "architectural description language" in general.
However, this differs from the Wikipedia definition of
[architecture description language](http://en.wikipedia.org/wiki/Architecture_description_language).
That is not what I mean.

If you think about pattern recognition in the next stage of handling point clouds, it will be necessary to "describe" architectural layouts.
Because this interpretation must be performed on each scale (city, quarter, block, building, façade, &), this "description language" should be developed in consideration of both Revit and Galileo.

"Interpreting point clouds" bears a likeness to "suggesting layouts", both of them work with probabilities.

Just some thoughts...

Once again, many thanks to Rudolf for his valuable thoughts and pointers!