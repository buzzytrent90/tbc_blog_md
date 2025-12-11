---
ai_optimized: true
author: Jeremy Tammik
complexity_score: 2.5
content_type: qa
optimization_date: '2025-12-11T11:44:13.297419'
original_url: https://thebuildingcoder.typepad.com/blog/0057_polygon_transformation.html
post_number: '0057'
reading_time_minutes: 4
series: geometry
slug: polygon_transformation
source_file: 0057_polygon_transformation.htm
tags:
- csharp
- levels
- revit-api
- walls
- geometry
title: Polygon Transformation
word_count: 776
---

### Polygon Transformation

As promised in the discussion on
[three-dimensional polygon area calculation](http://thebuildingcoder.typepad.com/blog/2008/12/3d-polygon-areas.html),
I have now implemented some test code to transform the 3D polygon into a horizontal position.
I can then ignore its Z coordinate and apply the 2D polygon area calculation algorithm to it as well and then compare the results.
To my surprise, the results of the 2D and 3D algorithms do indeed exactly match.
This is a continuation of the analysis for determining the profile boundary loop polygons for
[floor slabs](http://thebuildingcoder.typepad.com/blog/2008/10/slab-boundary.html)
and
[walls](http://thebuildingcoder.typepad.com/blog/2008/11/wall-elevation-profile.html),
and on the calculation of areas for
[2D](http://thebuildingcoder.typepad.com/blog/2008/12/2d-polygon-areas-and-outer-loop.html)
and
[3D](http://thebuildingcoder.typepad.com/blog/2008/12/3d-polygon-areas.html)
polygons.

First let us discuss the top level code for transforming the polygon, so that we can understand and discuss the required lower level functions afterwards.
Since this is only for testing purposes, I have enclosed it in a debug pragma.
We start out with the 3D planar polygon obtained from a wall face and located somewhere arbitrarily in space.
We have also determined the normal vector of the plane it is lying in.
The goal is to transform it into the XY plane so we can apply the 2D polygon area calculation algorithm to it.
This can be achieved through the following steps:

- Determine the transform to rotate the polygon so it is parallel to the XY plane.
- Apply this transform to the polygon.
- Flatten the polygon by dropping the Z coordinate.
- Calculate the 2D polygon area.

Here is the code implementing these steps:

```csharp
#if DEBUG
  Transform t = GetTransformToZ( normal );

  List<XYZ> polygonHorizontal
    = ApplyTransform( polygon, t );

  List<UV> polygon2d
    = CmdSlabBoundaryArea.Flatten(
      polygonHorizontal );

  double a2
    = CmdSlabBoundaryArea.GetSignedPolygonArea(
      polygon2d );

  Debug.Assert( Util.IsEqual( a, a2 ),
    "expected same area from 2D and 3D calculations" );
#endif
```

This code transforms the 3D polygon into a horizontal plane so we can use the 2D GetSignedPolygonArea() and compare its results with the 3D calculation.
Next, let us look at the detailed implementation of the required low level functions.

To determine the transform for rotating the polygon parallel to the XY plane, we first calculate the angle α between the normal vector and the Z axis.
We have the following cases to consider:

- α is zero, so the normal vector is identical to the Z axis; in this case, the required transform is the identity.
- α equals π, so the normal vector is equal to the negative Z axis; in this case, we have to rotate the polygon by 180 degrees around an arbitrary axis perpendicular to the Z axis, for instance the X axis.
- In all other cases, we can determine a rotation axis that is perpendicular to both the normal vector and the Z axis, and rotate around that by the angle α.

Here is the code returning such a transformation:

```csharp
Transform GetTransformToZ( XYZ v )
{
  Transform t;

  double a = XYZ.BasisZ.Angle( v );

  if( Util.IsZero( a ) )
  {
    t = Transform.Identity;
  }
  else
  {
    XYZ axis = Util.IsEqual( a, Math.PI )
      ? XYZ.BasisX
      : v.Cross( XYZ.BasisZ );

    t = Transform.get\_Rotation( XYZ.Zero, axis, a );
  }
  return t;
}
```

The other three steps are trivial or have already been discussed.
Applying the transform to the polygon is achieved by applying it individually to each vertex.
Flattening is done by simply dropping the Z coordinate and implemented in
[CmdSlabBoundaryArea](http://thebuildingcoder.typepad.com/blog/2008/12/2d-polygon-areas-and-outer-loop.html),
as is the 2D polygon area calculation.

As previously mentioned, it would also be interesting to compare the relative speed of transforming the 3D situation to 2D and using 2D area calculation versus direct 3D area calculation.
Unfortunately, I still have not installed Visual Studio 2008, and my 2005 version is just a so-called professional edition.
Apparently, I should have installed the team edition to have access to the profiling functionality.
I hope to be able to do so with the 2008 version which I will soon be switching to.

I am also looking forward to making use of the enhanced generic functionality and libraries provided by the new versions of .NET and C# to improve the implementation of some of the polygon transformation methods which work by applying some transformation to each vertex.

Here is
[version 1.0.0.16](http://thebuildingcoder.typepad.com/blog/files/bc10016.zip)
of the complete Visual Studio solution with the updated code discussed here, still using the VS 2005 platform.