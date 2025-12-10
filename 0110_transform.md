---
post_number: "0110"
title: "Transform"
slug: "transform"
author: "Jeremy Tammik"
tags: ['csharp', 'elements', 'geometry', 'levels', 'revit-api', 'walls']
source_file: "0110_transform.htm"
original_url: "https://thebuildingcoder.typepad.com/blog/0110_transform.html"
---

### Transform

In some upcoming posts, I would like to take a look at the how to handle transformations of Revit elements, as well as locations of nested linked files.
Before we get into that, let us take a closer look at the underlying Revit API Transform class.
We briefly mentioned it in the discussion of the Revit API
[geometry library](http://thebuildingcoder.typepad.com/blog/2008/09/geometry-librar.html),
and demonstrated some basic use of it for
[transforming a polygon](http://thebuildingcoder.typepad.com/blog/2008/12/polygon-transformation.html)
from 3D to 2D.

The Revit API Transform class is defined in the Autodesk.Revit.Geometry namespace and represents a transformation of the affine 3-space.
It provides the typical properties and methods expected for a transformation class:

- Multiply, multiply two transforms.
- ScaleBasis, scale the basis vectors.
- ScaleBasisAndOrigin, scale the basis vectors and the origin.
- Basis, BasisX, BasisY, BasisZ, return the base axes.
- Determinant, return the determinant.
- HasReflection, to tell whether this transformation is a reflection.
- Identity, the identity transformation.
- Inverse, the inverse transformation.
- IsConformal, to tell whether this transformation is [conformal](http://en.wikipedia.org/wiki/Conformal_map).
- IsIdentity, to tell whether this transformation is the identity.
- IsTranslation, to tell whether this transformation is a translation.
- Origin, the origin.
- Reflection, returns a transformation that reflects about a specified plane.
- Rotation, returns a transformation that rotates by a specified angle about a specified axis and point.
- Scale, return the scale of the transformation.
- Translation, returns a transformation that translates by a specified vector.

In some other APIs, transformations are handled by matrices.
The Revit API generally tries to address a higher level of abstraction, so the class is defined as a generic transformation class, and the underlying implementation is not explicitly exposed by the API.

The Creation.Application does not provide any methods to create transform objects.
A Transform instance can be created using the new operator as a copy constructor, or by using some of the static methods of the Transform class.
The transform default constructor initializes it to identity:

```csharp
Transform trans1, trans2, trans3;
```

Here are examples of using the static Transform class methods to define rotation, translation, scaling and mirroring transforms:

```csharp
XYZ ptOrigin = XYZ.Zero;
XYZ ptXAxis = XYZ.BasisX;
XYZ ptYAxis = XYZ.BasisY;

trans1 = Transform.get\_Rotation( // rotation
  ptOrigin, ptXAxis, 90 );

trans2 = Transform.get\_Translation( // translation
  ptXAxis );

trans1 = trans2.ScaleBasis( 2.0 ); // scaling

Plane plane1 = creApp.NewPlane(
  ptXAxis, ptYAxis );

trans3 = Transform.get\_Reflection( // mirror
  plane1 );
```

One simple use of a transform is to translate a point:

```csharp
p3 = trans1.OfPoint( p2 );
```

Some further examples are given by the Revit API
[tips and tricks sample code](zip/rac_tips_20090303.zip),
in the region '1. XYZ and Transform' in RevitGeometry.cs.
Besides these transforms, they also show how to invert and concatenate transformations.

The polygon transformation is implemented by code in the region 'Transform 3D plane to horizontal' in The Building Coder sample code external command class
[CmdWallProfileArea](http://thebuildingcoder.typepad.com/blog/2008/12/polygon-transformation.html),
specifically in the methods GetTransformToZ and ApplyTransform:

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

    t = Transform.get\_Rotation( XYZ.Zero,
      axis, a );
  }
  return t;
}

List<XYZ> ApplyTransform(
  List<XYZ> polygon,
  Transform t )
{
  int n = polygon.Count;

  List<XYZ> polygonTransformed
    = new List<XYZ>( n );

  foreach( XYZ p in polygon )
  {
    polygonTransformed.Add( t.OfPoint( p ) );
  }
  return polygonTransformed;
}
```

GetTransformToZ calculates the rotation required to transform a planar polygon from its arbitrary orientation in 3D space to a plane parallel to the XY plane.
The algorithm used is described in detail in the discussion on
[transforming a polygon](http://thebuildingcoder.typepad.com/blog/2008/12/polygon-transformation.html).
ApplyTransform applies the rotation to the 3D polygon to obtain a rotated copy which is parallel to the XY plane.
It does so by iterating through the points of the original polygon and calculating the rotated point using Transform.OfPoint on each.