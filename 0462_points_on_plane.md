---
post_number: "0462"
title: "Plane Normal and Points on Plane"
slug: "points_on_plane"
author: "Jeremy Tammik"
tags: ['elements', 'geometry', 'references', 'revit-api']
source_file: "0462_points_on_plane.htm"
original_url: "https://thebuildingcoder.typepad.com/blog/0462_points_on_plane.html"
---

### Plane Normal and Points on Plane

**Question:** I have an origin point and a normal vector and wish to determine three points to create a reference plane. How can I calculate the third point using normal and origin?

**Answer:** This is very elementary geometry.
If you have an origin point and a normal vector, you can easily calculate three points in a plane.
Conversely, if you have three points in a plane, it is easy to calculate the normal vector.

Let 'x' denote the vector
[cross product](http://en.wikipedia.org/wiki/Cross_product).

Given an origin point P and a normal vector N:

- Select an arbitrary vector V that is not collinear with N, e.g. the X or Y axis.- Set U = N x V to obtain a vector that is perpendicular to N.- Set V = N x U to obtain another vector perpendicular to both N and V.- Set Q = P + U and R = P + V to obtain three points P, Q and R in the plane.

Given three points P, Q and R in the plane:

- Set V = Q - P and W = R - P to obtain two perpendicular vectors in the plane.- Set N = V x W to obtain the plane normal.

I published a number of posts demonstrating this.
The most useful ones for you might be the ones describing the GetCurveNormal method:

- [Detail curve must be in a plane](http://thebuildingcoder.typepad.com/blog/2010/05/detail-curve-must-be-in-plane.html)- [Model curve creator](http://thebuildingcoder.typepad.com/blog/2010/05/model-curve-creator.html)