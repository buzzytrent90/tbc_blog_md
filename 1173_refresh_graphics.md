---
post_number: "1173"
title: "Refresh Element Graphics Display"
slug: "refresh_graphics"
author: "Jeremy Tammik"
tags: ['elements', 'revit-api', 'schedules', 'sheets', 'transactions', 'views', 'windows']
source_file: "1173_refresh_graphics.htm"
original_url: "https://thebuildingcoder.typepad.com/blog/1173_refresh_graphics.html"
---

### Refresh Element Graphics Display

I talked about how to
[refresh element graphics display](http://thebuildingcoder.typepad.com/blog/2011/07/refresh-element-graphics-display.html) back
in 2011, and it seems like the time has come to revisit that topic.

After that, I also want to mention an approach to
[determine the height of a schedule in a sheet](#3) and
how to
[avoid running a web server](#4).

#### Refresh Element Graphics Display

Afshin just submitted a
[comment](http://thebuildingcoder.typepad.com/blog/2013/12/replacing-an-idling-event-handler-by-an-external-event.html?cid=6a00e553e16897883301a3fd2391f5970b#comment-6a00e553e16897883301a3fd2391f5970b) about this on
[replacing an Idling event handler by an external event](http://thebuildingcoder.typepad.com/blog/2013/12/replacing-an-idling-event-handler-by-an-external-event.html),
and another developer asked something similar today, so here goes:

I am aware of several different possible ways to trigger a graphical refresh, e.g.:

- Call UIDocument.RefreshActiveView
- Call Document.Regenerate
- Commit a sub-transaction
- Commit a transaction
- Commit a transaction group

I would expect the various approaches to become more expensive in the order listed above.

You will have to try out what works best for you in your specific context.

If you have a specific element that you want refreshed and nothing else helps, I used the following neat
[temporary translation approach](http://thebuildingcoder.typepad.com/blog/2011/07/refresh-element-graphics-display.html) successfully
in various CAD systems:

- Move an element by a zero length vector
- Move an element by a non-zero distance, and then back again

I can think off-hand of two very nice samples that demonstrate refreshing the graphics screen programmatically, the DisplacementElementAnimation SDK sample and the kinetic facade sample.

The
[DisplacementElementAnimation sample](http://thebuildingcoder.typepad.com/blog/2013/08/animation-and-the-displacementelement-class.html) makes
use of the DisplacementElement class to create an automated real-time animation of exploding a building.

It is pretty radical, since it works with the Idling event to modify the model quite significantly to displace elements in each step, and achieves the graphics update by committing a transact5ion for each animation step:

The
[kinetic facade sample](http://thebuildingcoder.typepad.com/blog/2012/04/devcamp-and-refresh-display-for-a-kinetic-facade.html) is
a bit simpler.
It does not use Idling, performs a fixed number of animation steps, and uses sub-transactions to refresh:

**Addendum:** Arnošt Löbel addded some important clarifications and corrections to the statements above:

Palette update will probably not work in the middle of a command without adding new internal Revit support for it.
It is a modeless window reacting to messages sent to the Revit's message queue, and processing of messages from the queue is probably suspended until an external command finishes. Therefore, if you need the palette to refresh, you have to refactor the application around some modeless concept and give the palette a chance to update regularly.

Regarding the 'possible several ways' to trigger graphical refresh listed above:

- Committing a sub-transaction and transaction group does absolutely nothing in that regard.
- Regeneration might, but probably does not trigger anything in most cases, since regeneration itself is a pure DB operation.
- Committing a transaction mostly does trigger UI update, but probably not all of all possible UI items and controls.
- RefreshActiveView should trigger necessary UI refresh, naturally.

The 'zero move' trick that apparently works in some other CAD systems is not recommended.
If committing a transaction does not work, this would not help.

The properties palette does not update to the current situation if an external command is running.

In summary, the approach illustrated by the DisplacementElementAnimation SDK sample is probably best.

#### Determine the Height of a Schedule in a Sheet

**Question:** How can I determine the height of a schedule, i.e. a ScheduleSheetInstance element, placed on a sheet?

I need to find the height of several schedules in order to place them on the sheet without overlapping each other.

The sum of the row heights returned by the TableSectionData.GetRowHeight method does not match the total schedule height.

Is it possible to find their height from the ViewSchedule elements before placing them in a sheet?

**Answer:** Yes, the TableSectionData.GetRowHeight method returns the row height in the schedule/template view.

When a schedule is inserted into a sheet, the sizing is dynamically recalculated based on the section row height, text size, schema type, frozen control, split control, etc.

The final values of the row heights are not stored or exposed.

Therefore, the only way to find the total height is by using the ScheduleSheetInstance boundary box.

#### Avoid Running a Server

**Question:** Do I have to run a web server to open files on the local file system in JavaScript?

**Answer:** You do not have to, but some people do.

Here's why:

The browser has to be very careful about what is allowed to open files and where.

Everybody is scared of the consequences of a remote app running or opening files on your local machine.

Therefore, in general, the [same-origin policy](http://en.wikipedia.org/wiki/Same_origin_policy) is enforced.

Some restrictions can be worked around using CORS, [cross-origin resource sharing](http://en.wikipedia.org/wiki/Cross-origin_resource_sharing).

Basically, a client-side HTML file is really limited as to what it can access.

So many people do run web servers locally in order to open images etc.

Of course, that adds complexity.

On a Windows machine using Chrome, it can be avoided by adding a command line argument when starting up the browser, like this:

- "C:\Program Files (x86)\Google\Chrome\Application\chrome.exe" --allow-file-access-from-files

With this argument specified, you can do just about anything with JavaScript that you can on a server.

Thanks to Theo Armour and Mr.doob for showing
[how to run things locally](https://github.com/mrdoob/three.js/wiki/How-to-run-things-locally).