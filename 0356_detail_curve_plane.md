---
ai_optimized: true
author: Jeremy Tammik
complexity_score: 3.5
content_type: code_example
optimization_date: '2025-12-11T11:44:13.801770'
original_url: https://thebuildingcoder.typepad.com/blog/0356_detail_curve_plane.html
post_number: '0356'
reading_time_minutes: 4
series: geometry
slug: detail_curve_plane
source_file: 0356_detail_curve_plane.htm
tags:
- csharp
- elements
- geometry
- parameters
- python
- revit-api
- views
title: Detail Curve Must be in Plane
word_count: 786
---

### Detail Curve Must be in Plane

Here is a recent case concerning some Revit API behaviour that changed in the new release.

**Question:** Were any major changes applied to the NewDetailCurve method?

In the Revit2011 API, I am getting an error saying "Curve must be in the plane".
Do I now have to project the curve object onto the plane myself, or is it still taken care of automatically like in Revit 2010?
The code I am calling looks like this:
```csharp
  DetailCurve detailCurve
    = doc.Create.NewDetailCurve(
      doc.ActiveView, curve );
```

**Answer:** When I look at the Revit 2011 API documentation information in RevitAPI.chm on the ItemFactoryBase.NewDetailCurve method, I see that it is designed to throw an ArgumentException when the curve is not in plane of the view.

In the Revit 2010 API, this exception is not documented.

So there does indeed appear to be a change in behaviour between the two versions, and it does seem that you have to ensure that the geometry curve is projected onto the proper plane before calling this method.

**Response:** So I will have to project the curve onto the plane before I draw a detail line.
This was previously done automatically.
No problem.
Do you have any code showing how to project points to the plane?

**Answer:** I demonstrate some simple projection of points in the discussions of

- [2D polygons](http://thebuildingcoder.typepad.com/blog/2008/12/2d-polygon-areas-and-outer-loop.html)- [2D polygon areas](http://thebuildingcoder.typepad.com/blog/2008/12/3d-polygon-areas.html)- [Polygon transformation](http://thebuildingcoder.typepad.com/blog/2008/12/polygon-transformation.html)

Some basic info on the Transform class is provided in:

- [Transform class](http://thebuildingcoder.typepad.com/blog/2009/03/transform.html)- [Transformations](http://thebuildingcoder.typepad.com/blog/2010/01/transformations.html)

Applying a transform to a curve is also discussed in:

- [Scaling a curve](http://thebuildingcoder.typepad.com/blog/2009/07/scale-a-curve.html)

Here is another example of a curve input argument that must reside in a specific plane:

- [NewSweptBlend](http://thebuildingcoder.typepad.com/blog/2010/03/newsweptblend.html)

I would suggest that you split the algorithm into two parts and implement separate independent methods for each of them:

- Determine the plane that a curve lies in.- Rotate a curve from a given plane to the XY plane.

For the first step, you could ask the curve for its start and end points and some point in the middle that does not lie on the same line as the other two.
The latter can possibly be obtained by asking the curve for its parameter range and evaluating it in the middle, or by tessellation. In case of tessellation, you could iterate through the tessellation points and use each one together with the start and end points to try and determine a valid plane.

In the case of the line, the tessellation only returns two points.
I once heard that that is the only element that does that, all non-linear curves return at least three.
So you could use this property to determine that the curve is a line (and add an assertion as well, if you like).

Once you have three points that are not in a line, you can use those to determine the normal vector of the plane in which the curve lies, assuming that it is planar.

If you are working with tessellation points, you can add debug assertions to ensure that the other tessellation points (if there are any more) are all in the same plane.

Actually, I went ahead and implemented what I describe above:
```python
XYZ GetCurveNormal( Curve curve )
{
  IList<XYZ> pts = curve.Tessellate();
  int n = pts.Count;

  Debug.Assert( 1 < n,
    "expected at least two points "
    + "from curve tessellation" );

  XYZ p = pts[0];
  XYZ q = pts[n - 1];
  XYZ v = q - p;
  XYZ w, normal = null;

  if( 2 == n )
  {
    Debug.Assert( curve is Line,
      "expected non-line element to have "
      + "more than two tessellation points" );

    // for non-vertical lines, use Z axis to
    // span the plane, otherwise Y axis:

    double dxy = Math.Abs( v.X ) + Math.Abs( v.Y );

    w = ( dxy > Util.TolPointOnPlane )
      ? XYZ.BasisZ
      : XYZ.BasisY;

    normal = v.CrossProduct( w ).Normalize();
  }
  else
  {
    int i = 0;
    while( ++i < n - 1 )
    {
      w = pts[i] - p;
      normal = v.CrossProduct( w );
      if( !normal.IsZeroLength() )
      {
        normal = normal.Normalize();
        break;
      }
    }

#if DEBUG
    {
      XYZ normal2;
      while( ++i < n - 1 )
      {
        w = pts[i] - p;
        normal2 = v.CrossProduct( w );
        Debug.Assert( normal2.IsZeroLength()
          || Util.IsZero( normal2.AngleTo( normal ) ),
          "expected all points of curve to "
          + "lie in same plane" );
      }
    }
#endif // DEBUG

  }
  return normal;
}
```

The second step is demonstrated in the [polygon transformation](http://thebuildingcoder.typepad.com/blog/2008/12/polygon-transformation.html) post I mentioned above.