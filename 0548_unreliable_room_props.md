---
ai_optimized: true
author: Jeremy Tammik
complexity_score: 2.5
content_type: documentation
optimization_date: '2025-12-11T11:44:14.149999'
original_url: https://thebuildingcoder.typepad.com/blog/0548_unreliable_room_props.html
post_number: 0548
reading_time_minutes: 4
series: general
slug: unreliable_room_props
source_file: 0548_unreliable_room_props.htm
tags:
- doors
- elements
- family
- filtering
- levels
- parameters
- revit-api
- rooms
- walls
- windows
title: Unreliable Room Properties
word_count: 781
---

### Unreliable Room Properties

We often discussed the properties linking hosted elements to their host, such as a door or window hosted by a wall.
One of the first and most fundamental posts on this topic is the trusty old
[relationship inverter](http://thebuildingcoder.typepad.com/blog/2008/10/relationship-in.html).

Unfortunately, it is not always so trustworthy after all, not due to any flaw in the algorithm itself, but based on the
[GIGO](http://en.wikipedia.org/wiki/Garbage_In,_Garbage_Out) principle of
[garbage in, garbage out](http://en.wikipedia.org/wiki/Garbage_In,_Garbage_Out).

The underlying properties may simply not always be correctly set.

Rudolf Honke of
[acadGraph CADstudio GmbH](http://www.acadgraph.de) recently
pointed out that a family instance may be
[lacking its Level property](http://thebuildingcoder.typepad.com/blog/2011/01/family-instance-missing-level-property.html).

Here another observation by Rudolf on the Room, ToRoom and FromRoom properties of windows.

Note that we recently looked at a problem dealing with the
[phase dependent FromRoom and ToRoom](http://thebuildingcoder.typepad.com/blog/2011/02/phase-dependent-room-properties.html) properties
on family instances, and you need to take the phase into account when querying them, but that does apparently not help in this case.

Rudolf says:

Besides the fact that one must make sure that a property is not null before one use it to retrieve elements using a filtered element collector, there can occur other problems as well.

Here is a situation showing two rooms with a window of the same type and direction each:

![Two rooms with windows](img/rh_windows_by_room_1.png)

The left room's element id is 212140; the right one's is 212143.

If we look at the window properties, the left window has a valid 'FromRoom' property, but 'ToRoom' is null:

![Left window properties](img/rh_windows_by_room_2.png)

In contrast, the right window has a valid 'ToRoom' property, but 'FromRoom' is null:

![Right window properties](img/rh_windows_by_room_3.png)

Both the left and the right window are hosted in the same wall.
They differ in their 'Mirrored' property, even though they are correctly placed.
If I mirror one of them by pressing its control arrows, their 'Mirrored' property becomes equal, but one of them faces in the wrong direction, of course.
In this picture, the left window is now turned inside out:

![Flipped window facing the wrong way](img/rh_windows_by_room_4.png)

**So in this case, how can we reliably collect windows by room?**

We have to check both properties – if one is valid, we find the corresponding room, using a simple 'OR' condition.
This will still not work if there is a window between two rooms, because this middle window would belong to both rooms.

For test purposes, I put a window of another type between our two rooms.
As you can see, this window has both the 'ToRoom' and 'FromRoom' properties set, but it also has a valid value for its 'Room' property, in addition.
Some windows have this, others don't:

![Window properties between two rooms](img/rh_windows_by_room_5.png)

How can we handle all these situations?
Should we check all three properties and select the room that occurs most often?
We also could say that this window belongs to both rooms, but, be aware; we must make sure that it will not be counted twice if we sum the windows of all rooms.

To cut a long story short: you cannot always rely on properties, and need to be prepared for certain irregularities.

Rudolf did some further testing, including an analysis of the phases involved, and continues:

The issue with the
[unreliable Level property](http://thebuildingcoder.typepad.com/blog/2011/01/family-instance-missing-level-property.html) happened
in a different file, and I think that was a different problem, namely the invisibility of 'nulled' parameters in the UI, making it impossible for the user to correct them.

I tested accessing the properties using all phases.
The phases of windows and rooms are all identical.

I think that the theme of these two issues is 'don't rely on properties', in general.
And you see that even a corrupted project or family file may work pretty well and continue to be used and worked on.
What I want to express is that there might be thousands of RVTs and RFAs out there in the world that have lost part of their structural integrity.
People continue working with them and don't even notice that they are damaged because these damages are just limited locally.
So we developers and every application we produce must be prepared to handle these sort of files because we cannot give all of them to ADN support to fix...