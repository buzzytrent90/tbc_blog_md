---
post_number: "1035"
title: "LINQ DIY Transformed Geometry Bounding Box"
slug: "geo_bound_box"
author: "Jeremy Tammik"
tags: ['csharp', 'elements', 'family', 'geometry', 'revit-api', 'walls', 'windows']
source_file: "1035_geo_bound_box.htm"
original_url: "https://thebuildingcoder.typepad.com/blog/1035_geo_bound_box.html"
---

### LINQ DIY Transformed Geometry Bounding Box

We looked at a couple of transformation issues in the past, e.g.:

- [Polygon Transformation](http://thebuildingcoder.typepad.com/blog/2008/12/polygon-transformation.html)
- [Transform](http://thebuildingcoder.typepad.com/blog/2009/03/transform.html)
- [Transform Instance Coordinates](http://thebuildingcoder.typepad.com/blog/2009/03/transform-instance-coordinates.html)
- [Transform an Element](http://thebuildingcoder.typepad.com/blog/2009/05/transform-an-element.html)
- [Transformations](http://thebuildingcoder.typepad.com/blog/2010/01/transformations.html)
- [Get Transformed Family Instance Geometry](http://thebuildingcoder.typepad.com/blog/2011/06/get-transformed-family-instance-geometry.html)
- [Planar Face Transform](http://thebuildingcoder.typepad.com/blog/2011/11/planar-face-transform.html)

Now Jon Smith of
[Construction Industry Solutions Ltd](http://coins-global.com) raised
another issue related to transformations.

Unlike the ones listed above, however, this one deals with the transformation and bounding box of the in-memory geometry objects, as opposed to database elements:

**Question:** We need to transform some elements into a different coordinate system before getting their bounding box.
We are doing this by getting the GeometryElement object, calling GetTransformed on it, and then calling GetBoundingBox on that.

This works perfectly well in Revit 2014.
In Revit 2013, however, calling GetBoundingBox on a transformed GeometryElement returns a box with a huge negative maximum, and a huge positive maximum – obviously wrong – and the max is less than the min.
Is there a workaround for this in 2013?
I tried tweaking the geometry options but to no avail.
Also, the actual geometry objects contained in the geometry elements do appear to be valid (or at least there are the same number of solids in the pre and post transformation elements).

Here is a code snippet that demonstrates the problem.
In my real code, I am transforming the GeometryElement by something meaningful, but in this sample case I just use the identity matrix.
Therefore, preTransformBox should be equal to postTransformBox.
One problem with the sample is both max and min are displaying positive in the dialog, but max is actually negative – the values are too large to display properly!
This sample works fine in 2014, but in 2013 the returned BoundingBoxXYZ is garbage:

```csharp
public void test( Element elem )
{
  Options geomOpts = new Options();

  GeometryElement geometryElement
    = elem.get\_Geometry( geomOpts );

  if( geometryElement == null )
    return;

  BoundingBoxXYZ preTransformBox
    = geometryElement.GetBoundingBox();

  GeometryElement geometryElementTransformed
    = geometryElement.GetTransformed(
      Transform.Identity );

  BoundingBoxXYZ blah = new BoundingBoxXYZ();

  BoundingBoxXYZ postTransformBox
    = geometryElementTransformed.GetBoundingBox();

  System.Windows.Forms.MessageBox.Show(
    "Pre Min: " + preTransformBox.Min.ToString()
    + "\nPre Max: " + preTransformBox.Min.ToString()
    + "\nPost Min: " + postTransformBox.Min.ToString()
    + "\nPost Max: " + postTransformBox.Min.ToString() );
}
```

**Answer:** Interesting topic.
I have not used the geometry element transformation or bounding box functionality in the past, just of the main Revit database element.

What do you use it for, please?

Is there an alternative way to achieve your goal?

Could you use the database element bounding box instead of the geometry element one?

I am glad to hear that the problem does not occur in Revit 2014.

Regarding the situation in Revit 2013, the fact that the normal geometry retrieval seems to be working properly sounds like a good starting point to me.

I would suggest simply traversing that geometry and creating your own min and max bounding box points from it.

There are several samples on the blog that demonstrate the process, e.g. the
[structural setout point](http://thebuildingcoder.typepad.com/blog/2012/08/structural-concrete-setout-point-add-in.html) application.
It collects and marks all vertices of structural concrete element geometry solids.

Instead of collecting all the individual points from the geometry, you could just traverse the points and keep track of and extend the min and max as you go along. If you wish to handle curved elements correctly, you could ask them for their tessellations and process all the intermediate points as well.

**Answer:** Perfect – thank you!
Talking it through has helped me with a shim for 2013.

The reason for using GeometryElement is we can transform the geometry into the elements "ECS" or entity coordinate system, using AutoCAD terminology, so we can create a tight bounding box for it.
We cannot transform the element itself as its geometry may be affected by other elements, e.g. wall joins.

My solution for 2013 (that appears to work from some initial testing) is to extract all triangulated edges for each solid in each non-transformed geometry element, transform the points into the ECS and create the bounding box from those points.

I pulled out the code into the following stand-alone method getAlignedBoundingBoxFromElement – it’s still fairly long but there might be something useful in it.

It includes both ways of retrieving the aligned bounding box in the code – the 2014 method, and the 2013 one as a fallback.
I prefer the 2014 method as it does not rely on a particular type of geometry and will be future-proof if other geometry types are added, or Revit changes its bounding-box algorithm:

```csharp
  /// <summary>
  /// Return an aligned bounding box around a given
  /// element. Only works if the element uses a
  /// LocationCurve (not LocationPoint).
  /// </summary>
  /// <param name="elem">Element to return
  /// bounding box for</param>
  /// <returns>Bounding box</returns>
  public static BoundingBoxXYZ
    getAlignedBoundingBoxFromElement(
      Element elem )
  {
    LocationCurve lc = elem.Location
      as LocationCurve;

    if( lc != null
      && lc.Curve.IsBound )
    {
      Options geomOpts = new Options();

      GeometryElement geometryElement
        = elem.get\_Geometry( geomOpts );

      if( geometryElement != null )
      {

        // transformation matrix from model
        // to the element curve ECS
        // to simplify it, we force the transformation
        // to be aligned to the world XY plane - so
        // just rotated around the Z axis

#if \_REVIT2014\_
        XYZ StartPoint = lc.Curve.GetEndPoint(0);
        XYZ EndPoint = lc.Curve.GetEndPoint(1);
#else
        XYZ StartPoint = lc.Curve.get\_EndPoint( 0 );
        XYZ EndPoint = lc.Curve.get\_EndPoint( 1 );
#endif

        // flatten its Z - where the Z is, is
        // irrelevant as the bounding box will
        // determine the height

        EndPoint = new XYZ( EndPoint.X, EndPoint.Y,
          StartPoint.Z );

        XYZ direction = EndPoint - StartPoint;
        XYZ normal = XYZ.BasisZ;

        Transform t
          = Transform.Identity;

        t.Origin = StartPoint;
        t.BasisX = direction.Normalize();

        t.BasisY = normal.CrossProduct( t.BasisX )
          .Normalize();

        t.BasisZ = t.BasisX.CrossProduct( t.BasisY )
          .Normalize();

        // check we have a valid matrix

        if( !t.IsConformal || t.Determinant < 0 )
          return null;

        Transform modelToElementTransform = t.Inverse;

        // transform the geometry into the ECS we
        // have created to get an aligned bounding box

        GeometryElement geometryElementTransformed
          = geometryElement.GetTransformed(
            modelToElementTransform );

        BoundingBoxXYZ ecsBoundingBox
          = geometryElementTransformed
            .GetBoundingBox();

        // Revit 2013 Shim
        // ===============
        // in Revit 2013, the returned bounding box
        // is garbage - the Max is hugely negative
        // and the Min is hugely positive
        // if this happens then get geometry points
        // in model coordinates, convert to element
        // coordinates and create bounding box from
        // those
        // NB. we could use this code for 2014 as
        // well, but the calculation is probably not
        // as accurate for some situations

        if( ecsBoundingBox.Max.X
          < ecsBoundingBox.Min.X )
        {
          // get points from all edges in all solids
          // - allows for curves by using tessellated
          // edges

          List<XYZ> pts = new List<XYZ>();

          foreach( Solid solid in geometryElement
            .OfType<Solid>() )
          {
            foreach( Edge edge in solid.Edges )
              pts.AddRange( edge.Tessellate() );
          }

          // transform the points into element
          // coordinates

          ecsBoundingBox = new BoundingBoxXYZ();

          pts = pts.Select(
              pt => modelToElementTransform.OfPoint(
                pt ) )
            .ToList();

          if( pts.Any() )
          {
            // calculate the bounding box

            ecsBoundingBox = new BoundingBoxXYZ();

            ecsBoundingBox.Max = new XYZ(
              pts.Max( pt => pt.X ),
              pts.Max( pt => pt.Y ),
              pts.Max( pt => pt.Z ) );

            ecsBoundingBox.Min = new XYZ(
              pts.Min( pt => pt.X ),
              pts.Min( pt => pt.Y ),
              pts.Min( pt => pt.Z ) );
          }
          else
          {
            // fail-case - if element has
            // no solid geometry

            return null;
          }
        }

        // finally apply the ECS to Model
        // transformation back to the bounding box

        ecsBoundingBox.Transform = t.Multiply(
          ecsBoundingBox.Transform );

        return ecsBoundingBox;
      }
    }
    return null;
  }
```

Many thanks to Jon for this discovery, his nice exploration and clean solution.

Please note that his method includes some neat use of generic LINQ methods to succinctly extract all vertices and tessellation points from all solids contained in the element geometry and determine their maximum and minimum coordinate values in a very few lines of code:

#### Extract all Vertices and Tessellation Points from all Solids

Let me extract and highlight Jon's compact example of using LINQ to achieve the following, since it can come in useful and be reused for numerous purposes:

- Extract all solids from the element geometry.
- Extract all vertices and intermediate tessellation points from all edges of each solid.

```csharp
    // get points from all edges in all solids
    // - allows for curves by using tessellated
    // edges
    List<XYZ> pts = new List<XYZ>();
    foreach( Solid solid in geometryElement
      .OfType<Solid>() )
    {
      foreach( Edge edge in solid.Edges )
        pts.AddRange( edge.Tessellate() );
    }
```

#### Isolate Conditional Compilation

I have one little generic suggestion to improve readability of the code above, applicable in many other circumstances as well:

You could eliminate the conditional compilation in the following lines:

```csharp
  #if \_REVIT2014\_
    XYZ StartPoint = lc.Curve.GetEndPoint(0);
    XYZ EndPoint = lc.Curve.GetEndPoint(1);
  #else
    XYZ StartPoint = lc.Curve.get\_EndPoint( 0 );
    XYZ EndPoint = lc.Curve.get\_EndPoint( 1 );
  #endif
```

This could be replaced by implementing an extension method for the Curve class named GetEndpoint, which is conditionally compiled and only defined for Revit versions prior to 2014.

With such an extension method, the new Revit 2014 code could be used throughout, and you would not even see that the extension method routes it through to the old method when needed.