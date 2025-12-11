---
ai_optimized: true
author: Jeremy Tammik
complexity_score: 7.0
content_type: code_example
optimization_date: '2025-12-11T11:44:14.900166'
original_url: https://thebuildingcoder.typepad.com/blog/0922_bounding_box_xyz_int.html
post_number: 0922
reading_time_minutes: 12
series: geometry
slug: bounding_box_xyz_int
source_file: 0922_bounding_box_xyz_int.htm
tags:
- csharp
- family
- geometry
- python
- revit-api
- schedules
- views
- windows
title: Curve Following Face and Bounding Box Implementation
word_count: 2406
---

### Curve Following Face and Bounding Box Implementation

Continuing the research and development for my
[cloud-based round-trip 2D Revit model editing project](http://thebuildingcoder.typepad.com/blog/2013/03/cloud-mobile-extensible-storage-data-use-in-schedules.html#3),
I looked at using the
[ExtrusionAnalyzer](http://thebuildingcoder.typepad.com/blog/2013/04/extrusion-analyser-and-plan-view-boundaries.html)
to create a plan view boundary profile for the furniture and equipment family instances and implemented a utility method
[SortCurvesContiguous](http://thebuildingcoder.typepad.com/blog/2013/03/sort-and-orient-curves-to-form-a-contiguous-loop.html) to
sort and re-orient the curves it returns into a closed contiguous loop.

An alternative and more effective method is to use the Edge AsCurveFollowingFace call instead of AsCurve, as we shall see below.

In any case, we want to check the results, e.g. to ensure that we really have obtained the valid closed boundary loops we expect.

The easiest way to do so is to visualise them graphically.
To visualise something, you mostly need to know how big it is first.
You often need to scale it to fit into your visualisation space, e.g. a fixed-size window, i.e. transform from its given initial size and location to the target coordinate space.
A useful way to determine the size of a boundary loop is to calculate the bounding box of its collection of vertices.

That leads to the following topics of today's post, including one additional unrelated issue that just came in:

- [Using AsCurveFollowingFace](#2)
- [XYZ point bounding box calculation](#3)
- [2D integer-based point bounding box calculation](#4)
- [Overview of sweep sample code and issue resolutions](#5)

#### Use Curve Following Face

I already had one stab at using a built-in Revit API method instead of implementing my own
[SortCurvesContiguous](http://thebuildingcoder.typepad.com/blog/2013/03/sort-and-orient-curves-to-form-a-contiguous-loop.html) solution:
Rudolf Honke suggested testing the
[ExporterIFCUtils ValidateCurveLoops](http://thebuildingcoder.typepad.com/blog/2013/04/extrusion-analyser-and-plan-view-boundaries.html#4) method instead, but that did not work.

Now Bettina Zimmermann provided another idea:

*I don’t understand why you write your own code for direction sorting of the curves when Revit can provide it for you: Edge.AsCurveFollowingFace.
Scott Conover's
[Geometry API document](zip/cp4011_conover.pdf) describes
it a bit but not very detailed.

I have written some code using Revit API to find the curves sorted in the right direction for using GeometryCreationUtilities.CreateExtrusionGeometry, so I found out it is possible using this call.*

I tested that, simply replacing the call to Edge.AsCurve by AsCurveFollowingFace:
```python
  Plane plane = new Plane( XYZ.BasisX,
    XYZ.BasisY, XYZ.Zero );

  ExtrusionAnalyzer extrusionAnalyzer = null;

  try
  {
    extrusionAnalyzer = ExtrusionAnalyzer.Create(
      solid, plane, XYZ.BasisZ );
  }
  catch( Autodesk.Revit.Exceptions
    .InvalidOperationException )
  {
    ++nExtrusionAnalysisFailures;
    return nAdded;
  }

  Face face = extrusionAnalyzer
    .GetExtrusionBase();

  foreach( EdgeArray a in face.EdgeLoops )
  {
    int nEdges = a.Size;

    List<Curve> curves
      = new List<Curve>( nEdges );

    XYZ p0 = null; // loop start point
    XYZ p; // edge start point
    XYZ q = null; // edge end point

    foreach( Edge e in a )
    {
      // This requires post-processing using
      // SortCurvesContiguous:

      Curve curve = e.AsCurve();

      if( \_debug\_output )
      {
        p = curve.get\_EndPoint( 0 );
        q = curve.get\_EndPoint( 1 );
        Debug.Print( "{0} --> {1}",
          Util.PointString( p ),
          Util.PointString( q ) );
      }

      // This returns the curves already
      // correctly oriented:

      curve = e.AsCurveFollowingFace(
        face );

      if( \_debug\_output )
      {
        p = curve.get\_EndPoint( 0 );
        q = curve.get\_EndPoint( 1 );
        Debug.Print( "{0} --> {1} following face",
          Util.PointString( p ),
          Util.PointString( q ) );
      }

      curves.Add( curve );
    }

    CurveUtils.SortCurvesContiguous(
      creapp, curves, \_debug\_output );

    q = null;

    JtLoop loop = new JtLoop( nEdges );

    foreach( Curve curve in curves )
    {
      // Todo: handle non-linear curve.
      // Especially: if two long lines have a
      // short arc in between them, skip the arc
      // and extend both lines.

      p = curve.get\_EndPoint( 0 );

      loop.Add( new Point2dInt( p ) );

      Debug.Assert( null == q
        || q.IsAlmostEqualTo( p, 1e-05 ),
        string.Format(
          "expected last endpoint to equal current start point, not distance {0}",
          (null == q ? 0 : p.DistanceTo( q ))  ) );

      q = curve.get\_EndPoint( 1 );

      if( \_debug\_output )
      {
        Debug.Print( "{0} --> {1}",
          Util.PointString( p ),
          Util.PointString( q ) );
      }

      if( null == p0 )
      {
        p0 = p; // save loop start point
      }
    }
    Debug.Assert( q.IsAlmostEqualTo( p0, 1e-05 ),
      string.Format(
        "expected last endpoint to equal current start point, not distance {0}",
        p0.DistanceTo( q ) ) );

    loops.Add( loop );

    ++nAdded;
  }
```

I also added new code to the SortCurvesContiguous method to report correctly sorted and oriented curves.
A typical snippet of output looks like this now:

```
(2.18,8.4,0) --> (2.18,5.78,0)
(2.18,8.4,0) --> (2.18,5.78,0) following face
(2.18,5.78,0) --> (7.31,5.78,0)
(2.18,5.78,0) --> (7.31,5.78,0) following face
(7.31,5.78,0) --> (7.31,8.4,0)
(7.31,5.78,0) --> (7.31,8.4,0) following face
(7.31,8.4,0) --> (6.75,8.4,0)
(7.31,8.4,0) --> (6.75,8.4,0) following face
(6.75,8.4,0) --> (6.75,8.46,0)
(6.75,8.4,0) --> (6.75,8.46,0) following face
(6.75,8.46,0) --> (6.41,8.46,0)
(6.75,8.46,0) --> (6.41,8.46,0) following face
(6.41,8.46,0) --> (6.41,8.4,0)
(6.41,8.46,0) --> (6.41,8.4,0) following face
(6.41,8.4,0) --> (3.08,8.4,0)
(6.41,8.4,0) --> (3.08,8.4,0) following face
(3.08,8.46,0) --> (3.08,8.4,0)
(3.08,8.4,0) --> (3.08,8.46,0) following face
(2.74,8.46,0) --> (3.08,8.46,0)
(3.08,8.46,0) --> (2.74,8.46,0) following face
(2.74,8.4,0) --> (2.74,8.46,0)
(2.74,8.46,0) --> (2.74,8.4,0) following face
(2.74,8.4,0) --> (2.18,8.4,0)
(2.74,8.4,0) --> (2.18,8.4,0) following face
0 endPoint (2.18,5.78,0)
1 start point match, no need to swap
1 endPoint (7.31,5.78,0)
2 start point match, no need to swap
2 endPoint (7.31,8.4,0)
3 start point match, no need to swap
3 endPoint (6.75,8.4,0)
4 start point match, no need to swap
4 endPoint (6.75,8.46,0)
5 start point match, no need to swap
5 endPoint (6.41,8.46,0)
6 start point match, no need to swap
6 endPoint (6.41,8.4,0)
7 start point match, no need to swap
7 endPoint (3.08,8.4,0)
8 start point match, no need to swap
8 endPoint (3.08,8.46,0)
9 start point match, no need to swap
9 endPoint (2.74,8.46,0)
10 start point match, no need to swap
10 endPoint (2.74,8.4,0)
11 start point match, no need to swap
11 endPoint (2.18,8.4,0)
(2.18,8.4,0) --> (2.18,5.78,0)
(2.18,5.78,0) --> (7.31,5.78,0)
(7.31,5.78,0) --> (7.31,8.4,0)
(7.31,8.4,0) --> (6.75,8.4,0)
(6.75,8.4,0) --> (6.75,8.46,0)
(6.75,8.46,0) --> (6.41,8.46,0)
(6.41,8.46,0) --> (6.41,8.4,0)
(6.41,8.4,0) --> (3.08,8.4,0)
(3.08,8.4,0) --> (3.08,8.46,0)
(3.08,8.46,0) --> (2.74,8.46,0)
(2.74,8.46,0) --> (2.74,8.4,0)
(2.74,8.4,0) --> (2.18,8.4,0)
FamilyInstance Furniture Desk <212646 1525 x 762mm> has 1 loop:
  0: (664,2561), (664,1761), (2227,1761), (2227,2561),
    (2056,2561), (2056,2580), (1954,2580), (1954,2561),
    (937,2561), (937,2580), (836,2580), (836,2561)
```

As you can see, AsCurveFollowingFace does indeed return all the curves correctly oriented and sorted, and SortCurvesContiguous has nothing at all left to do.

Thank you very much Bettina for this valuable suggestion!

Also thanks to Scott Conover for his valuable Geometry API document from the Autodesk University 2011 class, which I already mentioned numerous times in the past but was not previously published in PDF format and therefore not included in web searches until now.

#### Bounding Box Determination for XYZ Points

As explained above, I would like to check the validity of the boundary loop listed above.
The easiest way to do so is by displaying it visually.
To do so, I need to transform it from its native coordinate space to my display space, for instance Windows device or HTML5 Canvas coordinates.
This normally requires scaling and translation, and in turn knowledge of the size and location of the native coordinate space.
The latter can be easily determined by calculating a bounding box.

I therefore set out to implement a bounding box determination for XYZ points.

I could use the existing Revit API BoundingBoxXyz class, because it does provide both read and write access to its Min and Max properties.
However, it includes a number of additional features that are of no interest to me at this point, so I prefer to create my own lightweight class for this.

My first idea was to use a generic Tuple class, e.g.

```csharp
  class JtBoundingBoxXyz : Tuple<XYZ, XYZ>
```

However, the components of such a tuple are read-only and cannot be changed after instantiation, so I cannot easily update the min and max values as I add new points using this approach.

My next idea was to implement my own class and manage Revit XYZ max and min member variables within it.
However, the components of an XYZ are read-only and cannot be changed except by re-instantiation, so extending the bounding box to include new points would have sub-optimal performance in this case as well.

The simplest solution appears to be managing six individual doubles instead, and instantiating XYZ return values when needed, like this:

```python
/// <summary>
/// A bounding box for a collection of XYZ instances.
/// The components of a tuple are read-only and cannot
/// be changed after instantiation, so I cannot use
/// that easily.
/// The components of an XYZ are read-only and cannot
/// be changed except by re-instantiation, so I cannot
/// use that easily either.
/// </summary>
class JtBoundingBoxXyz // : Tuple<XYZ, XYZ>
{
  /// <summary>
  /// Minimum and maximum X, Y and Z values.
  /// </summary>
  double xmin, ymin, zmin, xmax, ymax, zmax;

  /// <summary>
  /// Initialise to infinite values.
  /// </summary>
  public JtBoundingBoxXyz()
  {
    xmin = ymin = zmin = double.MaxValue;
    xmax = ymax = zmax = double.MinValue;
  }

  /// <summary>
  /// Return current lower left corner.
  /// </summary>
  public XYZ Min
  {
    get { return new XYZ( xmin, ymin, zmin ); }
  }

  /// <summary>
  /// Return current upper right corner.
  /// </summary>
  public XYZ Max
  {
    get { return new XYZ( xmax, ymax, zmax ); }
  }

  public XYZ MidPoint
  {
    get { return 0.5 \* ( Min + Max ); }
  }

  /// <summary>
  /// Expand bounding box to contain
  /// the given new point.
  /// </summary>
  public void ExpandToContain( XYZ p )
  {
    if( p.X < xmin ) { xmin = p.X; }
    if( p.Y < ymin ) { ymin = p.Y; }
    if( p.Z < zmin ) { zmin = p.Z; }
    if( p.X > xmax ) { xmax = p.X; }
    if( p.Y > ymax ) { ymax = p.Y; }
    if( p.Z > zmax ) { zmax = p.Z; }
  }
}
```

#### Bounding Box Determination for 2D Integer Points

Applying the same principles, here is a pretty optimal implemenetation of a bounding box for 2D integer-based points.
It includes a constructor taking a collection of loops, which is the output I am generating from my furniture and equipment plan view calculation:

```python
/// <summary>
/// A bounding box for a collection
/// of 2D integer points.
/// </summary>
class JtBoundingBox2dInt
{
  /// <summary>
  /// Minimum and maximum X and Y values.
  /// </summary>
  int xmin, ymin, xmax, ymax;

  /// <summary>
  /// Initialise to infinite values.
  /// </summary>
  public JtBoundingBox2dInt()
  {
    xmin = ymin = int.MaxValue;
    xmax = ymax = int.MinValue;
  }

  /// <summary>
  /// Return current lower left corner.
  /// </summary>
  public Point2dInt Min
  {
    get { return new Point2dInt( xmin, ymin ); }
  }

  /// <summary>
  /// Return current upper right corner.
  /// </summary>
  public Point2dInt Max
  {
    get { return new Point2dInt( xmax, ymax ); }
  }

  /// <summary>
  /// Return current center point.
  /// </summary>
  public Point2dInt MidPoint
  {
    get
    {
      return new Point2dInt(
        (int)(0.5 \* ( xmin + xmax )),
        (int)(0.5 \* ( ymin + ymax )) );
    }
  }

  /// <summary>
  /// Expand bounding box to contain
  /// the given new point.
  /// </summary>
  public void ExpandToContain( Point2dInt p )
  {
    if( p.X < xmin ) { xmin = p.X; }
    if( p.Y < ymin ) { ymin = p.Y; }
    if( p.X > xmax ) { xmax = p.X; }
    if( p.Y > ymax ) { ymax = p.Y; }
  }

  /// <summary>
  /// Instantiate a new bounding box containing
  /// the given loops.
  /// </summary>
  public JtBoundingBox2dInt( JtLoops loops )
  {
    foreach( JtLoop loop in loops )
    {
      foreach( Point2dInt p in loop )
      {
        ExpandToContain( p );
      }
    }
  }
}
```

Stay tuned to see how I use this to finally display and verify the extrusion analyser output and my integer-based boundary loop management implementation, coming next.

#### Overview of Sweep Sample Code and Issue Resolutions

Before closing, here is one last issue that just came up and I would like to mention here as well:

**Question:** I am trying to create a sweep in a new family using the Revit API, and am having some difficulties.

I found only one single example in the Revit SDK Samples, and can only get it working using your template.
When I use my own, it fails.

Where can I find some more information on this, please?

**Answer:** I discussed some very advanced sweep issues with Bill Adkison, and we published the essence of his results on
[sweep path tolerance criteria](http://thebuildingcoder.typepad.com/blog/2012/10/sweep-pickpath-tolerance-criteria.html).
Normally, sweep issues are nowhere near as hard as the ones he encountered and mastered.

The most recent sample code demonstrating sweep generation was for the
[sweep family performance enhancement](http://thebuildingcoder.typepad.com/blog/2013/03/sweep-family-performance-enhancement.html).

Furthermore, the AEC DevBlog discusses a number of sweep creation issues and provides several source code samples to generate them:

- [Sweep creation](http://adndevblog.typepad.com/aec/2012/06/sweep-creation.html)
- [Sweep paths and sketch planes](http://adndevblog.typepad.com/aec/2012/08/sweep-with-the-same-pathprofile-produce-different-results-when-different-path-sketch-planes-are-used.html)
- [NewSweptBlend code sample](http://adndevblog.typepad.com/aec/2012/08/newsweptblend-code-sample.html)
- [NewSweptBlend profile prerequisites](http://adndevblog.typepad.com/aec/2012/05/prerequisites-for-input-profiles-in-newsweptblend-form-creation-with-revit-api.html)

I hope this helps.
Good luck!