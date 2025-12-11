---
ai_optimized: true
author: Jeremy Tammik
complexity_score: 1.3
content_type: qa
optimization_date: '2025-12-11T11:44:14.019852'
original_url: https://thebuildingcoder.typepad.com/blog/0480_constructability.html
post_number: 0480
reading_time_minutes: 2
series: general
slug: constructability
source_file: 0480_constructability.htm
tags:
- csharp
- elements
- family
- geometry
- revit-api
title: Complexity versus Constructability
word_count: 340
---

### Complexity versus Constructability

Here is another contribution from Ritchie Jackson of the
[Adaptive Architecture and Computation](http://www.aac.bartlett.ucl.ac.uk) programme
at UCL, the
[University College London](http://en.wikipedia.org/wiki/University_College_London), who
recently showed us his code and results working with
[blends, Hermite splines and derivatives](http://thebuildingcoder.typepad.com/blog/2010/11/blends-hermite-splines-and-derivatives.html).
This time Ritchie addresses the theme of how to ensure and demonstrate the constructability of complex doubly curved shapes:

With contemporary international focus on the design of complex, double-curved forms I thought I'd use the API to try and rationalise the process as an aid to fabrication and assembly.

Using a Roller-Coaster Reception Facility as a test rig, a 3D-spline was sketched in a conceptual mass family and the points exported to create setouts for C# driven components in a generic model family. The latter was purposely chosen to force simplification of geometry in order to address the buildability question, with all elements being reduced to planar forms (arcs and lines)

The images below outline the process. Although the 3D spline cannot be visualised in the generic model family, it can still be used as an underlying generator for geometry. It was evaluated at regular intervals along its length to provide setout points for the arc sub-components. The division number was progressively increased until the component junctions visually appeared to be smooth (tangents almost co-linear). Curve derivatives were extracted at these points and the tangent vectors used to create plane-normals for the cross-bracing and rafters:-

![Roller-Coaster Setout](img/ritchie-RollerCoaster-Setout.jpg)

The bracing points were tapered via linear interpolation and the rafter spans adjusted using a sine function:-

![Roller-Coaster Model](img/ritchie-RollerCoaster-Model.jpg)

Finally, a reception desk was added using a similar process (spline as generator) to give a sense of scale to the project:-

![Roller-Coaster Render](img/ritchie-RollerCoaster-Render.jpg)

Many thanks to Ritchie for these inspiring ideas and images!