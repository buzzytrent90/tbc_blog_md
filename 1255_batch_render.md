---
post_number: "1255"
title: "AU Ends and Batch Rendering Across Several Projects"
slug: "batch_render"
author: "Jeremy Tammik"
tags: ['doors', 'revit-api', 'selection', 'views']
source_file: "1255_batch_render.htm"
original_url: "https://thebuildingcoder.typepad.com/blog/1255_batch_render.html"
---

### AU Ends and Batch Rendering Across Several Projects

Let me share a couple of thoughts and a batch of links to discussions of topics related to batch processing that came up answering a case today in between all the hectic activity at Autodesk University.

For my part, AU is over now, and I am already sitting in the airport waiting for the flight back to Europe and the series of West European DevDays conferences starting in Paris on Monday.

The batch-processing topic was raised by this query:

**Question:** Can I use the Revit API to batch render views in several different individual Revit projects?

I have lot of Revit files, each with a set of views named in a structured, standardized manner.

I would like to batch render these overnight or during the weekend.

Is that achievable using the Revit API?

If not, do you know of any other way to do it, e.g. through a combination of Revit API and maybe external user interface scripting or something?

**Answer:** Yes, this is definitely possible.

There are a number of different possible approaches.

They may or may not involve some or all of the following techniques:

- Use of the journal file.
- Use of an add-in performing required tasks automatically triggered by the DocumentOpened event.
- If the DocumentOpened event does not allow you to achieve everything you need, you may want to implement a cascading event triggering a one-off Idling event.

Here is a bunch of discussions on The Building Coder on these and related topics:

- [Getting the Journal File Path](http://thebuildingcoder.typepad.com/blog/2009/02/getting-the-journal-file-path.html)
- [Journal File Replay](http://thebuildingcoder.typepad.com/blog/2009/07/journal-file-replay.html)
- [IFC Import and Conversion Journal Script](http://thebuildingcoder.typepad.com/blog/2010/07/ifc-import-and-conversion-journal-script.html)
- [Modeless Door Lister Flaws](http://thebuildingcoder.typepad.com/blog/2011/01/modeless-door-lister-flaws.html)
- [Subscribing to UI Automation Events](http://thebuildingcoder.typepad.com/blog/2011/01/subscribing-to-ui-automation-events.html)
- [Loading an Add-in With a Journal File](http://thebuildingcoder.typepad.com/blog/2011/11/loading-an-add-in-with-a-journal-file.html)
- [Invoke External Command on Start-up](http://thebuildingcoder.typepad.com/blog/2012/07/elbow-fitting-selection-and-dimensioning.html#6)
- [The Revit Server REST API](http://thebuildingcoder.typepad.com/blog/2013/08/the-revit-server-rest-api.html)

Of these, the IFC import and conversion example is the most complete and probably the best place to start, while the others cover various additional aspects.

Actually, the more I think about this, the more different topics might be involved:

You might want to
[drive Revit from outside](http://thebuildingcoder.typepad.com/blog/about-the-author.html#5.28),
or you could launch the whole batch process from within Revit, like the
[external command lister](http://thebuildingcoder.typepad.com/blog/2013/05/external-command-lister-and-adding-ribbon-commands.html).
Please be aware that you might run out of memory if you open and close a large number of files in one single Revit session.
Revit was built with end user interaction in mind, and an end user is normally not expected to open more than a handful or a few dozen files in one session.
Therefore, a serious batch processor will either have to monitor and log its progress, regularly check that Revit is still alive and well from time and time, and shut down and automatically restart to continue processing where it broke off if not.

When running unattended, you may need to programmatically
[detect and handle messages and failures](http://thebuildingcoder.typepad.com/blog/about-the-author.html#5.32).

You may be able to choose between the journaling approach pointed to above, and using
[PostCommand](http://thebuildingcoder.typepad.com/blog/about-the-author.html#5.3) to
programmatically launch built-in and Revit add-in commands instead.
Customisation using the journal file is not officially supported, whereas the PostCommand approach is.

> Alice came to a fork in the road. 'Which road do I take?' she asked.
>
> 'Where do you want to go?' responded the Cheshire Cat.
>
> 'I don't know,' Alice answered.
>
> 'Then,' said the Cat, 'it doesn't matter.

![Alice and the Cheshire Cat](img/alice_and_the_cheshire_cat.jpg)

Good luck to you with this, and wish me luck now crossing the pond!

#### Addendum

**Question:** Thank you for replying in such a fast manner.

I’ve found several posts on The Building Coder on automating Revit using the Journal file, but as far as I understand this is not directly supported by AutoDesk and it’s not very stable across Revit versions (compared to a strict API approach).

You covered my question about workarounds and I appreciate that.

But would it be possible to achieve this ONLY using the Revit API, e.g., a batch rendering plugin that doesn’t use the journal file?

**Answer:** I partially covered that aspect as well above already.

I do indeed believe that it is possible to implement a solution that avoids all use of the journal file.

Maybe like this:

1. Implement the rendering functionality you require as an external command.
2. Rewrite it so that it can be called from the DocumentOpened event, or Idling, if necessary.
3. Launch Revit with your required files one by one in a batch file and let your add-in do its dirty deed.

With all the possible variations already hinted at, I am optimistic that you can solve this to your full satisfaction.

By the way, I have now landed safely in London.
Next leg is Paris.