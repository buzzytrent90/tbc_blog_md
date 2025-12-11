---
ai_optimized: true
author: Jeremy Tammik
complexity_score: 2.7
content_type: code_example
optimization_date: '2025-12-11T11:44:13.721264'
original_url: https://thebuildingcoder.typepad.com/blog/0309_newellipse.html
post_number: 0309
reading_time_minutes: 2
series: general
slug: newellipse
source_file: 0309_newellipse.htm
tags:
- geometry
- parameters
- python
- revit-api
title: NewEllipse Parameters
word_count: 473
---

### NewEllipse Parameters

Here I am back at work again.
I see I have a large number of backed up comments to answer, and lots of other interesting new information to discuss.
One recent case handled by my colleague Joe Ye discovered an issue with the API documentation of the NewEllipse method that is worth pointing out:

**Question:** I am having an issue converting ellipses from AutoCAD into Revit.
I have the geometry of the ellipse converted correctly, but there are some issues with the length of partial ellipses.
I have checked my calculations and conversions, and to me they seem to be correct.
I am wondering whether there may be something wrong with the Application.Create.NewEllipse method.
Please let me know whether there is an issue with the Revit API in this area.

**Answer:** I have a good message for you.
The issue is not caused by the NewEllipse method itself, but from its description in the API documentation.
The last two arguments are not the start and end angle, but represent the start and end parameter values of the partial ellipse.

Given the start and end angles, however, we may not know the start and end point parameter values before creating the ellipse.
To solve this, one can first create a complete ellipse, then determine the start and end point parameters by geometrical analysis, and finally reset the start and end parameter values using the Ellipse.MakeBound method.

Here is a code snippet showing how to create an elliptical arc geometry object using this approach:

```python
Ellipse CreateEllipse( Application app )
{
  XYZ center = XYZ.Zero;

  double radX = 30;
  double radY = 50;

  XYZ xVec = XYZ.BasisX;
  XYZ yVec = XYZ.BasisY;

  double param0 = 0.0;
  double param1 = 2 \* Math.PI;

  Ellipse e = app.Create.NewEllipse( center,
    radX, radY, xVec, yVec, param0, param1 );

  // Create a line from ellipse center in
  // direction of target angle:

  double targetAngle = Math.PI / 3.0;

  XYZ direction = new XYZ(
    Math.Cos( targetAngle ),
    Math.Sin( targetAngle ),
    0 );

  Line line = app.Create.NewLineUnbound(
    center, direction );

  // Find intersection between line and ellipse:

  IntersectionResultArray results;
  e.Intersect( line, out results );

  // Find the shortest intersection segment:

  foreach( IntersectionResult result in results )
  {
    double p = result.UVPoint.U;
    if( p < param1 )
    {
      param1 = p;
    }
  }

  // Apply parameter to the ellipse:

  e.MakeBound( param0, param1 );

  return e;
}
```

The API documentation will be updated soon.

**Response:** I updated my code and it now works using the StartParameter and EndParameter obtained directly from the AutoCAD Ellipse entity.
The calculations are basically the same as the angles and the results are correct now.
Getting the parameter value directly from AutoCAD saves some coding and improves performance.

Here is
[version 1.1.0.62](zip/bc11062.zip)
of the complete Building Coder source code and Visual Studio solution including a new external command CmdEllipticalArc implementing this method.

Many thanks to Joe for this solution!