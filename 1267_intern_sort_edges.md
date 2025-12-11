---
ai_optimized: true
author: Jeremy Tammik
complexity_score: 2.6
content_type: qa
optimization_date: '2025-12-11T11:44:15.639609'
original_url: https://thebuildingcoder.typepad.com/blog/1267_intern_sort_edges.html
post_number: '1267'
reading_time_minutes: 3
series: general
slug: intern_sort_edges
source_file: 1267_intern_sort_edges.htm
tags:
- elements
- revit-api
- rooms
title: Autodesk Internship in California and Sorting Edges
word_count: 561
---

### Autodesk Internship in California and Sorting Edges

We have an internship opening coming up, and geometrical topics like sorting face loop edges is always fun:

- [Wanted more alive than alive: intern](#2)
- [Sorting face loop edges](#3)

#### Wanted More Alive than Alive: Intern

Are you or someone you know currently pursuing a bachelor or master's degree in computer science or a discipline with a strong computing component? Are you looking for a potentially life-changing internship experience this summer? Look no further; the Autodesk Developer Network is looking for an enthusiastic intern evangelist for this summer in the Bay Area. If you're excited by design and by working with 3D on the web, keep reading.

You will start by learning the API and developing a plan for writing your own sample apps, then implementing them. You blog about your experiences using the APIs as you progress, so others can follow in your steps. The completed samples are posted to our public GitHub account and used by developer evangelists around the world to showcase the API capabilities. There may be opportunities to attend hackathons, meetups and conferences as a sponsor to help spread the word about these amazing APIs.

![Internship](img/internship.png)

In short, your task will be to have as much fun as possible doing cool things with our APIs, and then telling other people about your achievements.

If you are up for the challenge and this sounds like you,
[apply right away](https://autodesk.taleo.net/careersection/adsk_cmp/jobdetail.ftl?job=14WD17104).

#### Sorting Face Loop Edges

Back to the Revit API, here is the nicest of the numerous questions that came up today:

**Question:** The Face.EdgeLoops property returns a bunch of unsorted edges.

How can I sort them into the proper contiguous order and separate them into inner and outer loops?

**Answer:** I do not believe the Revit API provides the full functionality that you are asking for.

One method that does part of the work that you should definitely be aware of is the Edge.AsCurveFollowingFace method that returns a curve corresponding to the edge oriented in its topological direction on the specified face.
That is the simplest option and a good place to start.

I use that myself in the RoomEditorApp, in the module CmdUploadRooms.cs, available from the
[RoomEditorApp GitHub repository](https://github.com/jeremytammik/RoomEditorApp).

Before discovering this method, I implemented similar functionality myself, in the same project, in the GetContiguousCurvesFromSelectedCurveElements method provided by the CurveUtils class in the ContiguousCurveSorter.cs module.

Another bit of Revit API functionality that may be useful in this context is the ExporterIFCUtils.ValidateCurveLoops method.

I also looked at differentiating between inner and outer loops, in one of the very early discussions on The Building Coder.
This also involves
[determining the area of the polygons](http://thebuildingcoder.typepad.com/blog/2008/12/2d-polygon-areas-and-outer-loop.html).

Determining areas of polygons is simple, fast and precise.

The more complex, general case with curved edges is harder.
If you can live with the approximation provided by tessellating the curves, it is trivial to convert the non-linear curved area to an approximated polygonal one.

One potential helpful tool for solving the general case more precisely is a
[2D polygon and Boolean operation library](http://thebuildingcoder.typepad.com/blog/2013/09/boolean-operations-for-2d-polygons.html).