---
post_number: "0905"
title: "Curve Evaluation and Song of the Gulls"
slug: "nurb_spline_eval_gulls"
author: "Jeremy Tammik"
tags: ['elements', 'family', 'filtering', 'geometry', 'parameters', 'references', 'revit-api', 'transactions', 'views', 'walls']
source_file: "0905_nurb_spline_eval_gulls.htm"
original_url: "https://thebuildingcoder.typepad.com/blog/0905_nurb_spline_eval_gulls.html"
---

﻿

### Curve Evaluation and Song of the Gulls

Below, we take a look at one aspect of
[parametric NURB spline curve evaluation](#2) and
an
[overview of recent AEC DevBlog posts](#3).

First, however, another update on my vacation :-)

#### Marvels of the Song of the Gulls

I am winding up my vacation on the
[Island of Ischia](http://en.wikipedia.org/wiki/Ischia) that
I first discovered three years back.

It is one of my favourite places in the world, where I visited natural wonders such as hot thermal springs of Sorgeto in
[Panza](http://en.wikipedia.org/wiki/Panza),
where you can build your own bathtub to mix the hot spring water with the cold waves from the sea,
and the antique natural roman bath of Cavascura on the Maronti Beach that includes a natural sauna in hewn rock that has been running uninterrupted for 3000 years.

On the way here, I first got off at the wrong ferry stop in Procida.
That also proved very nice, with all the families taking their numerous kids for a Sunday walk, to chat and play in the main square.

![View from Piano Liguori towards Procida and Naples](file:///p/2013/2013-03-06_ischia/piano_liguori.jpeg)

From the ferry, I walked up to the Chiesa della Annunziata and on to the Piano Liguori and then explored the peninsula of San Pancrazio with its spectacular old church and caves.
Here is the view from Piano Liguori back towards Procida and Naples.

![Cave of San Pancrazio](file:///p/2013/2013-03-06_ischia/jeremy_san_pancrazio_cave.jpeg)

I visited the Trani family restaurant, olives and vineyard.
They even built their own staircase access to the sea, tunnelling through the almost vertical rock facing the water.

![Rock staircase and tunnel down to the sea](file:///p/2013/2013-03-06_ischia/san_pancrazio_rock_tunnel.jpeg)

It was a huge pleasure chatting with one of the Trani sons and listening to his joy over the traditional agricultural work amidst the 'canto dei gabbiani', the song of the gulls, and the challenges of making it economically viable through some interaction with tourism.

I later saw this wonderful example of a scalable implementation and thinking outside the box:

![Scalable water box](file:///p/2013/2013-03-06_ischia/p1000504_water_box.jpg)

#### Parametric NURB Spline Curve Evaluation

**Question:** I use the Curve.Evaluate method to obtain equally spaced points along a curve.

This works fine for most curve types, but not on a NURB Spline that I created.

When I evaluate with regularly incremented parameter values, e.g. 0, .2, .4, .6, .8 and 1, the distances of the resulting positions along the curve are not visually equal.
Using the same curve to create a ruled surface shows the lines and points appearing where one would expect.

Here is an example with the green lines displayed where ruled lines on the face occur, and text letters at the locations returned by the evaluate function:

![NURB spline evaluation](img/evaluate_nurb.png)

**Answer:** The
[parameterisation of a NURB spline](http://thebuildingcoder.typepad.com/blog/2010/01/curves.html) is
such that regions of strong curvature are 'longer' in the parameter space than the actual curve length.

The assumption that a parameter measures uniformly along its length can only be made for lines and arcs.
For ellipses and splines the parameter does not typically follow this pattern, because of the mathematics involved.

This is mentioned in the
[Developer Guide](http://wikihelp.autodesk.com/Revit/enu/2013/Help/00006-API_Developer%27s_Guide) description
of the parameter used in
[curve parameterisation](http://wikihelp.autodesk.com/Revit/enu/2013/Help/00006-API_Developer%27s_Guide/0074-Revit_Ge74/0108-Geometry108/0110-Geometry110/Curves/Curve_Parameterization):

"A ‘normalized’ parameter.
The start value of the parameter is 0.0, and the end value is 1.0.
For some curve types, this makes evaluation of the curve along its extents very easy; for example, the midpoint of a line is at parameter 0.5.
(Note that for more complex curve equations like Splines this assumption cannot always be made)."

It is also explained in more detail in the
[Wikihelp community](http://wikihelp.autodesk.com/Revit/enu/Community)
article on
[points hosted on curve edges](http://wikihelp.autodesk.com/Revit/enu/Community/Articles/Measurement_Types_for_Points_Hosted_on_Curves%2f%2fEdges).

The Revit Geometry API currently does not offer the ability to measure by segment-length or normalized segment-length.
If it did, a more iterative solution would be easy to describe.

#### Recent AEC DevBlog Overview

Here is an overview of some of the posts by my colleagues on the AEC DevBlog in the past few weeks:

- [Bubble end and free end arguments](http://adndevblog.typepad.com/aec/2013/01/bubble-end-and-free-end-in-newreferenceplane.html): explores
  the detailed meaning of the two XYZ parameters bubbleEnd and freeEnd to the NewReferencePlane method.
- [Regenerating the model](http://adndevblog.typepad.com/aec/2013/01/it-is-easy-to-miss-this-regenerating-the-model.html): provides yet another example in the long list of situations requiring an
  [extra regeneration or transaction](http://thebuildingcoder.typepad.com/blog/2012/12/extra-transaction-or-regeneration-required.html).
- [SQLite version conflict with a Revit add-in](http://adndevblog.typepad.com/aec/2013/01/sqlite-version-conflict-with-revit-add-ins.html) discusses
  methods to resolve a DLL version conflict.
- [Hiding sections in ViewPlan](http://adndevblog.typepad.com/aec/2013/02/hiding-sections-in-viewplans-using-revit-api.html) presents
  sample code to filter out the plan views, filter the sections created on each one of them, and call HideElements to suppress them in the view.
- [Determining if a family instance requires a host](http://adndevblog.typepad.com/aec/2013/02/determining-if-a-family-instance-requires-a-host.html) can
  be achieved using RevitLookup and the HOST parameter on the family itself.
- [Retrieving paint material for walls](http://adndevblog.typepad.com/aec/2013/02/retrieving-paint-material-for-walls-using-revit-api.html) uses
  the Face HasRegions property and GetRegions method.
- [Disassociating a family parameter from an element parameter](http://adndevblog.typepad.com/aec/2013/02/disassociating-family-parameter-from-an-element-parameter.html) is
  possible using the FamilyManager AssociateElementParameterToFamilyParameter method.
- [Accessing the material value of a panel](http://adndevblog.typepad.com/aec/2013/02/accessing-material-value-of-a-panel-using-revit-api.html) uses
  the MATERIAL\_ID\_PARAM built-in parameter on the panel type referenced by the panel element.
- [Using Rebar.CreateFromCurves with curves](http://adndevblog.typepad.com/aec/2013/03/rebarcreatefromcurves-throws-exception-with-two-curves-.html):
  if a Rebar shape includes any straight edges, then its first and last curves must be straight lines, unless it is completely made up of arcs.
- [Revit API error in projects with web references](http://adndevblog.typepad.com/aec/2013/03/revit-api-error-in-projects-with-web-references.html) can
  be fixed by changing the 'Generate Serialization Assembly' setting from 'Auto' to 'Off' in the Visual Studio project properties build tab.