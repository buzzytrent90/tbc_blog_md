---
ai_optimized: true
author: Jeremy Tammik
complexity_score: 1.3
content_type: qa
optimization_date: '2025-12-11T11:44:15.642799'
original_url: https://thebuildingcoder.typepad.com/blog/1269_exporterifcutils.html
post_number: '1269'
reading_time_minutes: 2
series: general
slug: exporterifcutils
source_file: 1269_exporterifcutils.htm
tags:
- revit-api
- rooms
title: ExporterIfcUtils Curve Loop Sort and Validate
word_count: 326
---

### ExporterIfcUtils Curve Loop Sort and Validate

Joel Spahn raised a pertinent
[question](http://thebuildingcoder.typepad.com/blog/2015/01/autodesk-internship-in-california-and-sorting-edges.html?cid=6a00e553e16897883301b8d0be1356970c) on
[sorting face loop edges](http://thebuildingcoder.typepad.com/blog/2015/01/autodesk-internship-in-california-and-sorting-edges.html#3) that
was kindly picked up and answered by Scott Conover and Angel Velez from the Revit development team:

**Question:** It would be nice to know exactly what the following methods do:

- ExporterIfcUtils.SortCurveLoops()
- ExporterIfcUtils.ValidateCurveLoops()

**Answer:** Sorry that the Intellisense documentation is missing for these utilities.

Here is the brief documentation of each:

- **SortCurveLoops**: sorts a set of curve loops such that outer and inner loops are separated.

- Remarks: Outer loops are separated and inner loops are grouped according to their outer loop. Loops are assumed to be non-intersecting, and there will be no nesting of inner loops, i.e., an inner loop of an inner loop is another outer loop.
- Returns: The sorted collection of loops.
- Input:

- loops, the curve loops.

- **ValidateCurveLoops**: performs validity checks on a list of curve loops to ensure that they are all co-planar, closed, and properly oriented.

- Returns the curve loops properly oriented, if possible. If not, the return contains no loops.
- Input:

- curveLoops, the loops to check
- extrDirVec, the normal vector.

In ValidateCurveLoops, the 'extrDirVec' normal vector defines what the 'proper orientation' means, by ensuring that the loops are counter-clockwise relative to this direction vector.

Both methods are perfectly usable outside of any IFC context.

In the long term, there is no reason why they should not move into the regular Revit API.

Thank you, Joel, for raising this very valid question, and to Scott and Angel for their answers.

As I mentioned in the recent discussion
[sorting face loop edges](http://thebuildingcoder.typepad.com/blog/2015/01/autodesk-internship-in-california-and-sorting-edges.html#3),
the RoomEditorApp provides a sample code snippet exercising SortCurveLoops.