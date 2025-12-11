---
ai_optimized: true
author: Jeremy Tammik
complexity_score: 1.7
content_type: documentation
optimization_date: '2025-12-11T11:44:14.053111'
original_url: https://thebuildingcoder.typepad.com/blog/0495_flatten_profile.html
post_number: 0495
reading_time_minutes: 2
series: general
slug: flatten_profile
source_file: 0495_flatten_profile.htm
tags:
- elements
- parameters
- revit-api
- views
title: Flatten a Non-Planar Extrusion Profile
word_count: 311
---

### Flatten a Non-Planar Extrusion Profile

We arrived safely in Moscow yesterday for the next developer conference, taking place today.
It is not as cold as expected, just 5 degrees Centigrade below zero, quite tolerable after Tel Aviv.
And no beach.

Anyway, here is another contribution from Ritchie Jackson of the
[Adaptive Architecture and Computation](http://www.aac.bartlett.ucl.ac.uk) programme
at UCL, the
[University College London](http://en.wikipedia.org/wiki/University_College_London), who
already presented information on
[blends, Hermite splines and derivatives](http://thebuildingcoder.typepad.com/blog/2010/11/blends-hermite-splines-and-derivatives.html),
[complexity versus constructability](http://thebuildingcoder.typepad.com/blog/2010/11/complexity-versus-constructability.html),
and using the
[project Vasari API](http://thebuildingcoder.typepad.com/blog/2010/11/project-vasari-api.html).
He now discusses an interesting aspect of creating an extrusion of a non-planar profile:

I've found this useful on occasion when needing to create extrusions from profiles consisting of a non-planar element array – it works as long as the end-points of element pairs are coincident.

The 'Extrusion' function will automatically use the z-Coord of the second end-point of the first element in the CurveArray to establish the extrusion's base offset from the working plane.

In the image below the sketch profile consists of an arc and two lines in 3D.
After the extrusion was created using the API the original curves were exposed by selecting 'Edit Extrusion' in the current view showing them to be still in 3D.
However, if the API is used to access the Edges of the bottom Face, all evaluated parameters return a consistent z-Value – enabling a 2D 'Flatten' function.
The curve in plan will no longer be a true arc as the original end-point z-Values were different:-

![Flatten non-planar extrusion profile](img/ritchie-API-Flatten.jpg)