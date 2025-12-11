---
ai_optimized: true
author: Jeremy Tammik
complexity_score: 2.4
content_type: qa
optimization_date: '2025-12-11T11:44:14.750254'
original_url: https://thebuildingcoder.typepad.com/blog/0860_build_perf_analysis.html
post_number: 0860
reading_time_minutes: 3
series: general
slug: build_perf_analysis
source_file: 0860_build_perf_analysis.htm
tags:
- elements
- geometry
- levels
- revit-api
title: Building Performance Analysis and Face Tessellation
word_count: 623
---

### Building Performance Analysis and Face Tessellation

I am busy with Autodesk University and developer conference preparations.
The Mac is happily chugging along.

#### Welcome to Building Performance Analysis

In spite of these busy times, Emile Kfouri and John Kennedy just launched a new blog on the topic of
[Building Performance Analysis](http://autodesk.typepad.com/bpa) that
will probably be of great interest to many of us Revit API aficionados.

Meanwhile, here is an API issue that raises a couple of new points:

#### Triangulation Level of Detail

**Question:** I am triangulating Revit geometry face objects to get the triangular meshes.
When I first tried this for each face of a standard UB-Universal beam I didn't get the root face (the curved face joining the flange and the web).
I found that setting the Options detail level to Fine sorted that issue.

Now I wonder:

1. What is the 'LevelOfDetail' argument for on the Triangulate method?
   Does it relate to the Options detail level?- Does calling the Triangulate method without an argument correspond to calling the same method with 0.5 as the argument?- Is it possible to triangulate a solid element as a whole so that the triangle vertices match along an edge line, rather than tessellate each face individually, which gives non-matching vertices along the edge?

**Answer:** The level of detail argument to the triangulation has to do with the level of precision of the triangulation.
Lower levels of detail will result in a coarser tessellation with less triangles but maybe less accuracy.
Higher levels of detail will result in a more accurate but also more populous tessellation (with more triangles as a result).
Of course this applies only to faces with curvature – planes should be well represented at any level.

But this does not answer the question asked – the face comes from Element.Geometry, and there are different sets of faces typically for DetailLevels Coarse, Medium and Fine.
The Fine geometry will likely have more details such as the curved face mentioned, where the Medium geometry won't.
So it has nothing to do with Triangulate(LOD), the inputs are different from the different geometry so you get different output.

To address your other questions:

2. As usual, a simple question with a not-so-simple answer.
   When the level of detail is not used, it uses the Revit defaults for when Revit triangulates faces for graphics (there may actually be a cache of these facets in some cases).
   There is probably no universal lod set in these situations.- Yes! The Revit 2013 API introduced the SolidUtils.TessellateSolidOrShell method.
     This routine processes the entire solid at once so that there are matched vertices at the edges.
     It takes a SolidOrShellTessellationControls argument including an option about LevelOfDetail with the same range of values and meaning as in the Face.Triangulate case.
     Revit doesn't use this method itself, so this data will never be cached.

**Response:** Thank you for finding out.
I should have searched the 2013 Developer's Guide to find the SolidUtils class myself.
I generally use the 2012 PDF guide since it is easier to read/use than the 2013 WikiHelp.

**Answer:** I understand about the PDF.

However, the Wikihelp has the advantage that Google will return hits in it for you.

I generally use the Revit API help file RevitAPI.chm, and especially I often consult the What's New section.
It would actually be quite useful to have a collection of each release's What's New section.
Maybe I should write a blog post on that.

From then on, all my further search is online.
Obviously, I start with the blog first.
That is what it is there for: my own personal knowledge base :-)