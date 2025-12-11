---
ai_optimized: true
author: Jeremy Tammik
complexity_score: 1.9
content_type: qa
optimization_date: '2025-12-11T11:44:14.478701'
original_url: https://thebuildingcoder.typepad.com/blog/0732_split_duct_pipe.html
post_number: '0732'
reading_time_minutes: 1
series: mep
slug: split_duct_pipe
source_file: 0732_split_duct_pipe.htm
tags:
- elements
- revit-api
- walls
- mep
title: Split a Duct or Pipe
word_count: 295
---

### Split a Duct or Pipe

Here is another short and sweet answer to a question that keeps popping up regularly:

**Question:** How can I change the length of a pipe using the Revit MEP API?

I am actually interested in programmatically splitting the pipe, but I cannot see any support for this in the API.

I tried to change the length of the pipe to indirectly cause a split, but the LocationCurve.Length property is read-only, so I cannot simply modify it.

**Answer:** I touched briefly on the topic of
[splitting a duct or pipe](http://thebuildingcoder.typepad.com/blog/2009/04/revit-api-cases-1.html#2) a
long time ago, back in 2009.

As stated there, there is no built-in ready-made method to split a pipe or duct.

You can however create as many new ducts and pipes as you like using the NewDuct and NewPipe methods.

Also, although the LocationCurve.Length property is read-only, you can still modify it by simply assigning a completely new curve to the element Location property itself.
We already discussed samples of doing so for a
[beam](http://thebuildingcoder.typepad.com/blog/2011/09/bevelled-steel-beams.html) and a
[wall](http://thebuildingcoder.typepad.com/blog/2010/08/edit-wall-length.html).
The same approach can be used on a pipe as well, and obviously replacing the location curve can also change its length.

Finally, the whole operation of splitting pipes and ducts both by changing their lengths and by creating new elements is demonstrated by the Revit SDK samples, especially the AutoRoute and AvoidObstruction ones:

- AutoRoute: Route a set of ducts and fittings between a base air supply equipment and two terminals.- AvoidObstruction: Detect and resolve obstructions between ducts, pipes, and beams.