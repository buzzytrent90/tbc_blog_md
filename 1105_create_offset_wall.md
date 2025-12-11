---
ai_optimized: true
author: Jeremy Tammik
complexity_score: 1.5
content_type: qa
optimization_date: '2025-12-11T11:44:15.313461'
original_url: https://thebuildingcoder.typepad.com/blog/1105_create_offset_wall.html
post_number: '1105'
reading_time_minutes: 2
series: general
slug: create_offset_wall
source_file: 1105_create_offset_wall.htm
tags:
- parameters
- revit-api
- walls
title: Creating an Offset Wall Solution
word_count: 325
---

### Creating an Offset Wall Solution

Happy
[St. Valentine's Day](http://en.wikipedia.org/wiki/Valentine%27s_Day)!

![A heart of hands](img/linas_hand_heart.png)

As we all know and have known for a long time from the exploration of the
[wall compound layers](http://thebuildingcoder.typepad.com/blog/2008/11/wall-compound-layers.html),
the Revit API wall location line is at the centre of the wall.

There is no way to change that, and it is important to note that the API location line is completely separate from the one whose location can be controlled through the user interface, and also through the built-in parameter WALL\_KEY\_REF\_PARAM, but only after the wall has been created.

When creating a new wall, the location line you specify is always the wall centre line.

This has sometimes been an issue for people wishing to programmatically
[create an offset wall](http://thebuildingcoder.typepad.com/blog/2013/09/creating-an-offset-wall.html) along
an edge, e.g. on top of a given slab, so that the wall finish coincides with the slab edge.

Well, the best solutions are always the simplest, and now Simon Moreau seems to have come up with the long-awaited ultimate one for this situation in his
[comment](http://thebuildingcoder.typepad.com/blog/2013/09/creating-an-offset-wall.html?cid=6a00e553e16897883301a73d7387f4970d#comment-6a00e553e16897883301a73d7387f4970d) on
the topic pointing out how he solves this to
[model skirting boards](http://bim42.com/2014/02/09/modellingskirtingboards):

"I just found a workaround for creating walls with the location line set to 'Finish Face' (for example) on creation.
I just first create a wall two times thicker, change the location line position, and finally change my wall type to its final thickness."

![Wall creation workaround](img/wall_creation_baseline_workaround.png)

Many thanks to Simon for his clever simple idea and sharing this thought!

I bet some readers wish they had thought of this themselves when needing it...