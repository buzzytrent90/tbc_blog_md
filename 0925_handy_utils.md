---
ai_optimized: true
author: Jeremy Tammik
complexity_score: 2.1
content_type: documentation
optimization_date: '2025-12-11T11:44:14.907530'
original_url: https://thebuildingcoder.typepad.com/blog/0925_handy_utils.html
post_number: 0925
reading_time_minutes: 2
series: general
slug: handy_utils
source_file: 0925_handy_utils.htm
tags:
- elements
- filtering
- geometry
- parameters
- references
- revit-api
- walls
title: Handy Utility Classes
word_count: 374
---

### Handy Utility Classes

Rudolf Honke of
[Mensch und Maschine acadGraph GmbH](http://www.acadgraph.de) has
repeatedly encouraged me to raise awareness of the numerous utility classes available in the Revit API and now provided the following starting point for a discussion of them.

One way to find a number of utility classes is to search the Revit API help file RevitAPI.chm for the string "utils":

![Revit API utility classes](img/util_classes_1.png)

In general, these classes provide static methods that can be called from any valid context with no need for an object instance.

One of the better-known examples is the
[LabelUtils class](http://thebuildingcoder.typepad.com/blog/2011/08/built-in-parameter-name-and-labelutils.html) that
returns localised display strings for built-in parameters and unit types:

![LabelUtils methods](img/util_classes_2.png)

By the way, Rudolf misses a method for built-in categories in this class...

If might be possible to implement some of these methods yourself, but using the utility methods obviously saves effort and duplication of code.

Another utility class that has been mentioned here in the past is
[WallUtils](http://thebuildingcoder.typepad.com/blog/2011/08/wall-joins-and-geometry.html).

It is important to be aware of their existence, or at least know where to look for them.

They are mostly quite well described in the help file, and yet many developers fail to notice them.
As said, sometimes you can get around using them, albeit with more effort on your own part.

For example, you can retrieve the element id of a referenced document using an appropriate element filter, or, much more simply, via the ExternalFileUtils GetAllExternalFileReferences method.

On the other hand, some things cannot be achieved except by using these methods.

For instance, after placing a couple of detail instances, their display order and visibility can be modified using the DetailElementOrderUtils class methods BringToFront, BringForward, SendBackward oder SendToBack.

Here is an occurrence count of the string "utils" in the different versions of the help file, showing the growth of this group of methods:

- 2011: 43
- 2012: 265
- 2013: 405
- 2014: 455

I hope this whets your appetite and look forward to hearing about more examples of unexpected and powerful uses of these methods.