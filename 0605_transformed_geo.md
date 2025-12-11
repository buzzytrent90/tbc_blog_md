---
ai_optimized: true
author: Jeremy Tammik
complexity_score: 5.7
content_type: qa
optimization_date: '2025-12-11T11:44:14.248366'
original_url: https://thebuildingcoder.typepad.com/blog/0605_transformed_geo.html
post_number: '0605'
reading_time_minutes: 4
series: geometry
slug: transformed_geo
source_file: 0605_transformed_geo.htm
tags:
- csharp
- elements
- family
- geometry
- revit-api
- views
title: Get Transformed Family Instance Geometry
word_count: 765
---

### Get Transformed Family Instance Geometry

One of the new methods added to the
[Revit 2012 API](http://thebuildingcoder.typepad.com/blog/2011/03/revit-2012-api-features.html)
is the GeometryElement GetTransformed method, which returns a transformed copy of the geometry in this element, and can also be used to obtain the geometry of a family instance in its model space coordinates.

This led to the following interesting question by Olli Kattelus of
[Progman Oy](http://magicad.com/node/40):

**Question:** First of all a salute for new features of Revit 2012 API and the improved handling of geometry.
The API now gives variety of tools to handle geometry i.e. in linked models.
Great!

I have faced a little bit strange behaviour and I would like to know what the reason is.

It is related to the linked DWG and the coordinate transformation.
If I apply transform to the DWG link geometry instance, it somehow applies a double transformation to the geometry returned.

I am using the following helper method to make the call to GetTransformed:
```csharp
protected Mesh fetchSomeMesh(
  GeometryElement gElem,
  Transform transform )
{
  // Apply transformation and seek for Meshes

  GeometryElement transformed
    = gElem.GetTransformed( transform );

  foreach( GeometryObject obj in transformed.Objects )
  {
    Mesh gMesh = obj as Mesh;
    if( null != gMesh )
    {
      return gMesh;
    }
  }

  // Loop and seek for geometry instances

  foreach( GeometryObject obj in gElem.Objects )
  {
    GeometryInstance gInstance
      = obj as GeometryInstance;

    if( null != gInstance )
    {
      // If it's GeometryInstance, combine
      // transformations and go recursive

      Transform combinedTransform = gInstance
        .Transform.Multiply( transform );

      return fetchSomeMesh(
        gInstance.SymbolGeometry,
        combinedTransform );
    }
  }
  return null;
}
```

The helper method is called and the first resulting vertex printed out like this:
```csharp
void checkLinkedDwg( Element linked )
{
  Instance inst = linked as Instance;
  Transform transform = inst.GetTransform();

  Options opt = new Options();
  opt.View = \_doc.ActiveView;

  GeometryElement gElem = inst.get\_Geometry( opt );

  Debug.Print( "Point without Transformation: "
    + fetchSomeMesh( gElem ).Vertices[0].ToString() );

  XYZ vertex = fetchSomeMesh( gElem, transform ).Vertices[0];
  Debug.Print( "Point when Transformed: " + vertex.ToString() );

  Debug.Print( "Point when Transformed + inverse: "
    + transform.Inverse.OfPoint( vertex ).ToString() );
}
```

**Answer:** I ported your original managed C++ version to C# and tested it, and I see the following results executing it on a sample model:

```
Point without Transformation: (0.0, 0.0, 0.0)
Point when Transformed: (2379.1, 1307.1, 0.0)
Point when Transformed + inverse: (1203.5, 662.6, 0.0)
```

It does indeed look as if the transform is applied twice in the second row, and the inverse transformation removes one of the two translations that were applied.

Looking at your code and guessing the internals of the GetTransformed method, I had a hunch that you need not pass in the transformation to it at all.
That is an optional extra.
The original transformation applied to the family instance itself is already built in to the method call.
So I implemented a new and simpler version of your fetchSomeMesh method called fetchSomeMeshTransformed which takes no transform argument at all, and simply always passes in the identity transform:
```csharp
protected Mesh fetchSomeMeshTransformed(
  GeometryElement gElem )
{
  // Apply transformation and seek for Meshes

  GeometryElement transformed
    = gElem.GetTransformed( Transform.Identity );

  foreach( GeometryObject obj in transformed.Objects )
  {
    Mesh gMesh = obj as Mesh;
    if( null != gMesh )
    {
      return gMesh;
    }
  }

  // Loop and seek for geometry instances

  foreach( GeometryObject obj in gElem.Objects )
  {
    GeometryInstance gInstance
      = obj as GeometryInstance;

    if( null != gInstance )
    {
      return fetchSomeMeshTransformed(
        gInstance.SymbolGeometry );
    }
  }
  return null;
}
```

I added another row of output to the testing call:
```csharp
  Debug.Print( "Point without Transformation: "
    + fetchSomeMesh( gElem ).Vertices[0].ToString() );

  XYZ vertex = fetchSomeMesh( gElem, transform ).Vertices[0];
  Debug.Print( "Point when Transformed: " + vertex.ToString() );

  Debug.Print( "Point when Transformed + inverse: "
    + transform.Inverse.OfPoint( vertex ).ToString() );

  Debug.Print( "Point with ID Transformation: "
    + fetchSomeMeshTransformed( gElem ).Vertices[0].ToString() );
```

Lo and behold, the sample now produces the following new and correct result in the fourth row:

```
Point without Transformation: (0.0, 0.0, 0.0)
Point when Transformed: (2379.1, 1307.1, 0.0)
Point when Transformed + inverse: (1203.5, 662.6, 0.0)
Point with ID Transformation: (1203.5, 662.6, 0.0)
```

In other words, and to be completely clear about this, the transform argument to the GetTransformed method can be used to define an additional transformation of the element geometry, and is concatenated with the transformation already applied to the symbol geometry to place it appropriately for a given instance.

If you just need the element geometry at its current location in the model, you can pass in an identity transform.

For completeness' sake, here is the source code and Visual Studio solution for the original managed C++ version
[transform2.zip](zip/transform2.zip),
and the C# port
[Transform3.zip](zip/Transform3.zip).