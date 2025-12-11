---
ai_optimized: true
author: Jeremy Tammik
complexity_score: 3.6
content_type: documentation
optimization_date: '2025-12-11T11:44:13.674899'
original_url: https://thebuildingcoder.typepad.com/blog/0283_abg04_curves.html
post_number: 0283
reading_time_minutes: 6
series: geometry
slug: abg04_curves
source_file: 0283_abg04_curves.htm
tags:
- geometry
- levels
- parameters
- revit-api
- views
title: Curves
word_count: 1178
---

### Curves

This is part 4 of Scott Conover's AU 2009 class on
[analysing building geometry](http://thebuildingcoder.typepad.com/blog/2010/01/analyse-building-geometry.html).

#### Curve Types

Revit uses a variety of curve types to represent curve geometry in a document.
The curve types and corresponding Revit API classes include:

- Bounded line – Line- Unbound line – Line- Arc – Arc- Circle – Arc- Elliptical arc – Ellipse- Ellipse – Ellipse- NURBS – NurbSpline- Hermite – HermiteSpline

Here are the definitions and some notes on each of these curve types:

- Bounded line:
  A line segment defined by its endpoints.
  Obtain endpoints from Curve.get\_Endpoint().- Unbounded line: An infinite line defined by a location and direction.
    Identify these with Curve.IsBound.
    Evaluate point and tangent vector at raw parameter zero to find the input parameters for the equation of the line.- Arc:
      A bound circular arc.
      Begin and end at a certain angle.
      These angles can be obtained by the raw parameter values at each end of the arc.- Circle:
        An unbound circle.
        Identify with Curve.IsBound.
        Use raw parameter for evaluation (from 0 to 2π).- Elliptical arc:
          A bound elliptical segment.- Ellipse:
            An unbound ellipse.
            Identify with Curve.IsBound.
            Use raw parameter for evaluation (from 0 to 2π).- Nurbs spline:
              A non-uniform rational B-spline.
              Used for splines sketched in various Revit tools, plus imported geometry.- Hermite spline:
                A spline interpolate between a set of points.
                Used for tools like Curve by Points and flexible ducts/pipes, plus imported geometry.

#### Mathematical representation

Mathematical representations of all of the Revit curve types are given in Appendix A of Scott's
handout document,
and we include them here as well for easy online access.
This section describes the curve types encountered in Revit geometry, their properties, and their mathematical representations.

##### Bounded Line

Bound lines are defined by their endpoints.
In the Revit API, obtain the endpoints of the line from the Curve-level get\_EndPoint() method.

The equation for a point on a bound line in terms of the normalized parameter 'u' and the line endpoints is

P(u) = P1 + u (P2 - P1)

##### Unbound lines

Unbound lines are handled specially in the Revit API.
Most curve properties cannot be used, however, Evaluate and ComputeDerivatives can be used to obtain locations along the curve when a raw parameter is provided.

The equation for a point for an unbound line in terms of the raw parameter 'u', the line origin P and normalized direction vector V is

P(u) = P0 + u V

##### Arcs and Circles

Arcs and Circles are represented in the Revit API by the Arc class.
They are defined in terms of their radius, center and vector normal to the plane of the arc, which are accessible in the Revit API directly from the Arc class as properties.

Circles have the IsBound property set to true.
This means they can only be evaluated by using a raw parameter (range from 0 to 2π), and the equation for a point on the circle in terms of the raw parameter is

P(u) = C + r ( nx cos(u) + ny sin(u) )

Here the assumption is made that the circle lies in the XY plane.

Arcs begin and end at a certain angle.
These angles can be obtained by the raw parameter values at each end of the arc, and angular values between these values can be plugged into the same equation as above.

##### Ellipse and Elliptical Arcs

Ellipses and elliptical arcs segments are represented in the Revit API by the Ellipse class.
Similar to arcs and circles, they are defined in a given plane in terms of their X and Y radii, center, and vector normal to the plane of the ellipse.

Full ellipses have the IsBound property set to true.
Similar to circles, they can be evaluated via raw parameter values between 0 and 2π:

P(u) = C + nx rx cos(u) + ny ry sin(u)

##### NurbSpline

NURBS (non-uniform rational B-splines) are used for spline segments sketched by the user as curves or portions of 3D object sketches.
They are also used to represent some types of imported geometry data.

The data for the NurbSpline include:

- The control points array P, of length n + 1.- The weights array w, also of length n + 1.- The curve degree, whose value is equal to one less than the curve order k.- The knot vector N, of length n + k + 1.

The parametric formula for a point on the curve is given by

P(u) = ∑ Pi wi Ni,k(u)   /   ∑ wi Ni,k(u)

The sums are over i = 0,...n, and u is limited to the interval 0 < u ≤ umax.

##### HermiteSpline

Hermite splines are used for curves which are interpolated between a set of control points, like Curve by Points and flexible ducts and pipes in MEP.
They are also used to represent some types of imported geometry data.
In the Revit API, the HermiteSpline class offers access to the arrays of points, tangent vectors and parameters through the ControlPoints, Tangents, and Parameters properties.

The equation for the curve between two nodes in a Hermite spline is

P(u) = h00(u) Pk + (uk+1 - uk) h10(u) Mk + h01(u) Pk+1 + (uk+1 - uk) h11(u) Mk+1

Here Pk and Pk+1 represent the points at each node, Mk and Mk+1 the tangent vectors, and uk and uk+1 the parameters at the nodes, and the basis functions are:

h00(u) = 2u3 - 3u2 + 1

h10(u) = u3 - 2u2 + u

h01(u) = - 2u3 + 3u2

h11(u) = u3 - u2

#### Curve analysis and processing

There are several Curve members which are tools suitable for use in geometric analysis.
In some cases, these APIs do more than you might expect by a quick review of their names.
We will discuss three of them:

- Intersect- Project- Tessellate

##### Intersect

The Intersect method allows you compare two curves to find how they differ or how they are similar. It can be used in the manner you might expect, to obtain the point or point(s) where two curves intersect one another, but it can also be used to identify:

- Collinear lines- Overlapping lines- Identical curves- Totally distinct curves with no intersections

The return value identifies these different results, and the output IntersectionSetResult contains information on the intersection point(s).

##### Project

The Project method projects a point onto the curve and returns information about the nearest point on the curve, its parameter, and the distance from the projection point.

##### Tessellate

This method splits the curve into a series of linear segments, accurate within a default tolerance. For Curve.Tessellate, the tolerance is slightly larger than 1/16'. This tolerance of approximation is the tolerance used internally by Revit as adequate for display purposes.

Note that only lines may be split into output of only two tessellation points; non-linear curves will always output more than two points, even if the curve has an extremely large radius which might mathematically equate to a straight line.

In the next instalment of this series, we will look the Revit API handling of faces.