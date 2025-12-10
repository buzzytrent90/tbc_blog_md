---
post_number: "0835"
title: "Element Intersection"
slug: "element_intersection"
author: "Jeremy Tammik"
tags: ['elements', 'geometry', 'levels', 'references', 'revit-api', 'rooms', 'walls']
source_file: "0835_element_intersection.htm"
original_url: "https://thebuildingcoder.typepad.com/blog/0835_element_intersection.html"
---

﻿

### Element Intersection

A common question when analysing a BIM is determining whether any of its elements intersect.

This was actually surprisingly difficult to answer programmatically in Revit a few releases back, as some of the interesting uses of the FindReferencesByDirection method show, for example the FindColumns SDK sample.
Later, that method was enhanced to lead to FindReferencesWithContextByDirection.

The FindColumns sample searches for intersections between walls and columns by shooting a ray along the wall centre line, raised above the bottom level by a foot or so, and detecting any columns hit by it.
One obvious problem with this approach is its inaccuracy – any column just partially intersecting the wall may easily be missed by the ray.

A few years ago, that was the best an add-in developer could hope for, and things looked even worse before the advent of these methods.

So what is the situation today?
In short:

**Question:** How can I programmatically detect whether two Revit elements intersect?

**Answer:** You can now perform a Boolean operation on Revit element solids and determine whether they intersect using the BooleanOperationsUtils class ExecuteBooleanOperation method.

Please note that the resulting intersection solid is transient, in-memory only.
It can be used for calculation and short-term visualisation using the analysis visualisation framework AVF, but it cannot be added to the Revit database or made persistent.

This class was introduced in the Revit 2012 API, and the
[associated webcast](http://thebuildingcoder.typepad.com/blog/2011/05/revit-2012-api-webcast.html) showed
it being used impressively in a much improved version of the FindColumns SDK sample.
This version determines the exact intersection solids between all columns and walls instead of roughly approximating the intersection detection by casting a single ray, and displays them to the user through AVF.

The Boolean intersection demo is described in detail and the source can be downloaded from the discussion of a simpler derived sample
[using AVF to highlight rooms](http://thebuildingcoder.typepad.com/blog/2011/12/using-avf-to-display-intersections-and-highlight-rooms.html).

Another example of making use of this class is given by the GeometryCreation\_BooleanOperation SDK sample.

Finally, here are two discussions on element intersection before the introduction of the BooleanOperationsUtils class, on
[intersection between elements in Revit 2011](http://thebuildingcoder.typepad.com/blog/2010/06/intersection-between-elements.html) and
the
[InstanceVoidCutUtils class](http://thebuildingcoder.typepad.com/blog/2011/06/boolean-operations-and-instancevoidcututils.html).

#### AEC Technology Update

If you are interested in a succinct summary of all the key developments in the AEC technology industry in the past couple of months, here is a potantial cancidate: the
[AEC Technology Updates, Fall 2012](http://www.aecbytes.com/newsletter/2012/issue_59.html),
including some snippets of Autodesk related news such as Revit LT and the Vela acquisition.