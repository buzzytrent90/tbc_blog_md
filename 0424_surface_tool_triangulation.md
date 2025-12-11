---
ai_optimized: true
author: Jeremy Tammik
complexity_score: 1.3
content_type: qa
optimization_date: '2025-12-11T11:44:13.925309'
original_url: https://thebuildingcoder.typepad.com/blog/0424_surface_tool_triangulation.html
post_number: '0424'
reading_time_minutes: 1
series: family
slug: surface_tool_triangulation
source_file: 0424_surface_tool_triangulation.htm
tags:
- revit-api
- walls
- family
title: Surface Triangulation Tool
word_count: 287
---

### Surface Triangulation Tool

I exchanged
[some](http://thebuildingcoder.typepad.com/blog/2008/11/model-line-creation.html?cid=6a00e553e1689788330133f2f0958e970b#comment-6a00e553e1689788330133f2f0958e970b)
[comments](http://thebuildingcoder.typepad.com/blog/2008/11/model-line-creation.html?cid=6a00e553e1689788330133f2f44c3b970b#comment-6a00e553e1689788330133f2f44c3b970b)
with Anthony of
[Clendon Burns & Park Ltd](http://www.cbp.co.nz)
in the course of the last couple of days and suggested the use of my
[NewSketchPlaneContainCurve method](http://thebuildingcoder.typepad.com/blog/2010/05/model-curve-creator.html), with the following happy end:

My app now works perfectly.
I'm generating some model lines from a topo surface, this enables our users to edit the top of retaining walls etc. to follow the ground line cut away by building pads.

Anthony has very friendlily published his resulting
[surface triangulation tool](http://forums.augi.com/showthread.php?p=1089399#post1089399) on AUGI.
Here is what he says about it there:

Have you ever wanted to trim the top of a retaining wall to the top of your toposurface?
Well now you can.

User instructions:

1. Create a building pad that follows your retaining wall.- Draw your retaining walls so they project above the surface make sure location line is set to core centre line).- Select the toposurface and run the surfacetool command.- Edit the walls profile, by snapping to the end points of the new model lines.

Install instructions:

1. Unzip the attached file to C:\surfacetool.- Copy the add-in file to your Revit addins directory.

Please visit
[Anthony's AUGI thread](http://forums.augi.com/showthread.php?p=1089399#post1089399)
to download the solution file Surface.zip and make sure he gets your appreciation!

For completeness' sake, here is the
[source code](zip/SurfaceTool.zip) as well.