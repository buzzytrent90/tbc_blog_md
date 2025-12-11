---
ai_optimized: true
author: Jeremy Tammik
complexity_score: 2.2
content_type: documentation
optimization_date: '2025-12-11T11:44:13.668549'
original_url: https://thebuildingcoder.typepad.com/blog/0279_abg02_curve_parameterisation.html
post_number: 0279
reading_time_minutes: 2
series: geometry
slug: abg02_curve_parameterisation
source_file: 0279_abg02_curve_parameterisation.htm
tags:
- csharp
- geometry
- parameters
- revit-api
- vbnet
- walls
title: Curve Parameterisation
word_count: 474
---

### Curve Parameterisation

This is part 2 of Scott Conover's AU 2009 class on
[analysing building geometry](http://thebuildingcoder.typepad.com/blog/2010/01/analyse-building-geometry.html).

Curves in the Revit API can be described as mathematical functions of an input parameter 'u', where the location of the curve at any given point in XYZ space is a function of 'u'. In Revit, the parameter can be represented in two ways:

- A 'normalized' parameter. The start value of the parameter is 0.0, and the end value is 1.0. This makes evaluation of the curve along its extents very easy, for example, the midpoint is at parameter 0.5. This is the typical way of addressing curve locations.- A 'raw' parameter. The start and end value of the parameter can be any value. For a given curve, the value of the minimum and maximum raw parameter can be obtained through Curve.get\_EndParameter(int) in C# or Curve.EndParameter(int) in VB.NET. Raw parameters are useful because their units are the same as the Revit default units (feet). So to obtain a location 5 feet along the curve from the start point, you can take the raw parameter at the start and add 5 to it. Raw parameters are also the only way to evaluate unbound and cyclic curves.

The methods Curve.ComputeNormalizedParameter and Curve.ComputeRawParameter automatically scale between the two parameter types. The method Curve.IsInside evaluates a raw parameter to see if it lies within the bounds of the curve.

You can use the parameter to evaluate a variety of properties of the curve at any given location:

- The XYZ location of the given curve. This is returned from Curve.Evaluate. Either the raw or normalized parameter can be supplied. If you are also calling ComputeDerivatives, this is also the .Origin property of the Transform returned by that method.- The first derivative/tangent vector of the given curve. This is the BasisX property of the Transform returned by Curve.ComputeDerivatives.- The second derivative/normal vector of the given curve. This is the BasisY property of the Transform returned by Curve.ComputeDerivatives.- The binormal vector of the given curve, defined as the cross-product of the tangent and normal vector.
        This is the BasisZ property of the Transform returned by Curve.ComputeDerivatives.

All of the vectors returned are non-normalized.
You can normalize any vector in the Revit API with XYZ.Normalize.

Note that there will be no value set for the normal and binormal vector when the curve is a straight line.
You can calculate a normal vector to the straight line in a given plane using the tangent vector.

In the next post of this series, we will look at an example using the tangent vector to the wall location curve to find exterior walls that face south.