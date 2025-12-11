---
ai_optimized: true
author: Jeremy Tammik
complexity_score: 5.5
content_type: code_example
optimization_date: '2025-12-11T11:44:14.590651'
original_url: https://thebuildingcoder.typepad.com/blog/0787_real_world_corners.html
post_number: 0787
reading_time_minutes: 6
series: general
slug: real_world_corners
source_file: 0787_real_world_corners.htm
tags:
- csharp
- elements
- family
- filtering
- geometry
- python
- revit-api
- walls
title: Real-World Concrete Corner Coordinates
word_count: 1270
---

### Real-World Concrete Corner Coordinates

I mentioned that I worked together with Paul Hellawell of
[GHD](http://www.ghd.com)
on a full-blown end-user-capable little application for the automatic creation of setout points for on-site location and construction of structural elements at the
[Melbourne DevLab](http://thebuildingcoder.typepad.com/blog/2012/03/melbourne-devlab.html).

Paul provided a
[description](http://thebuildingcoder.typepad.com/blog/2012/03/melbourne-devlab.html#3) of
the task and the initial idea for an implementation approach.

We got the application to a useful working state in the two days during the DevLab, and Paul has provided it to his designers to use in real projects ever since.

Before we look at the full-fledged application, let's explore the core API functionality required:

- [Filtering for structural concrete elements](#2).- Retrieving their corners, i.e. geometry traversal to [retrieve unique vertices](#3).- [Converting from Revit model to real-world coordinates](#4).

Once this is all in place, we can explain how to use the core functionality to implement the real-world end-user application.

#### Filtering for Structural Concrete Elements

We already looked at
[retrieving structural elements](http://thebuildingcoder.typepad.com/blog/2010/07/retrieve-structural-elements.html).

Just like there, we check for certain specific classes like Wall and Floor, and also for generic family instances with categories from our list of interest, including structural columns, framing, foundation, floors and ramps.

In this case, we are only interested in concrete elements, so we apply two structural material type filters as well, for concrete and precast concrete.

This is the method we ended up with to suit our purposes:
```csharp
/// <summary>
/// Retrieve all structural elements that we are
/// interested in using to define setout points.
/// We are looking at concrete for the moment.
/// This includes: columns, framing, floors,
/// foundations, ramps, walls.
/// </summary>
FilteredElementCollector GetStructuralElements(
  Document doc )
{
  // What categories of family instances
  // are we interested in?

  BuiltInCategory[] bics = new BuiltInCategory[] {
    BuiltInCategory.OST\_StructuralColumns,
    BuiltInCategory.OST\_StructuralFraming,
    BuiltInCategory.OST\_StructuralFoundation,
    BuiltInCategory.OST\_Floors,
    BuiltInCategory.OST\_Ramps
  };

  IList<ElementFilter> a
    = new List<ElementFilter>( bics.Length );

  foreach( BuiltInCategory bic in bics )
  {
    a.Add( new ElementCategoryFilter( bic ) );
  }

  LogicalOrFilter categoryFilter
    = new LogicalOrFilter( a );

  // Filter only for structural family
  // instances using concrete or precast
  // concrete structural material:

  List<ElementFilter> b
    = new List<ElementFilter>( 2 );

  b.Add( new StructuralMaterialTypeFilter(
    StructuralMaterialType.Concrete ) );

  b.Add( new StructuralMaterialTypeFilter(
    StructuralMaterialType.PrecastConcrete ) );

  LogicalOrFilter structuralMaterialFilter
    = new LogicalOrFilter( b );

  List<ElementFilter> c
    = new List<ElementFilter>( 3 );

  c.Add( new ElementClassFilter(
    typeof( FamilyInstance ) ) );

  c.Add( structuralMaterialFilter );
  c.Add( categoryFilter );

  LogicalAndFilter familyInstanceFilter
    = new LogicalAndFilter( c );

  IList<ElementFilter> d
    = new List<ElementFilter>( 6 );

  d.Add( new ElementClassFilter(
    typeof( Wall ) ) );

  d.Add( new ElementClassFilter(
    typeof( Floor ) ) );

  //d.Add( new ElementClassFilter(
  //  typeof( ContFooting ) ) );

#if NEED\_LOADS
  d.Add( new ElementClassFilter(
    typeof( PointLoad ) ) );

  d.Add( new ElementClassFilter(
    typeof( LineLoad ) ) );

  d.Add( new ElementClassFilter(
    typeof( AreaLoad ) ) );
#endif

  d.Add( familyInstanceFilter );

  LogicalOrFilter classFilter
    = new LogicalOrFilter( d );

  FilteredElementCollector col
    = new FilteredElementCollector( doc )
      .WhereElementIsNotElementType()
      .WherePasses( classFilter );

  return col;
}
```

#### Geometry Traversal to Retrieve Unique Vertices

Once the required elements have been retrieved, we analyse their geometry to determine all corners, i.e. geometry vertices.

This goes back to the second day of the
[Melbourne Revit API training](http://thebuildingcoder.typepad.com/blog/2012/03/melbourne-day-two.html),
where we looked at
[retrieving unique geometry vertices](http://thebuildingcoder.typepad.com/blog/2012/03/melbourne-day-two.html#2) from
a selected element.

This involves comparing XYZ points, i.e.
[real number equality testing](http://thebuildingcoder.typepad.com/blog/2012/05/connector-orientation.html#2).

First of all, we need an equality comparer for points:
```python
/// <summary>
/// Define equality for Revit XYZ points.
/// Very rough tolerance, as used by Revit itself.
/// </summary>
class XyzEqualityComparer : IEqualityComparer<XYZ>
{
  const double \_sixteenthInchInFeet
    = 1.0 / ( 16.0 \* 12.0 );

  public bool Equals( XYZ p, XYZ q )
  {
    return p.IsAlmostEqualTo( q,
      \_sixteenthInchInFeet );
  }

  public int GetHashCode( XYZ p )
  {
    return PointString( p ).GetHashCode();
  }
}
```

With that in hand, we can retrieve all unique vertices of a given solid retrieved from the element geometry:
```csharp
/// <summary>
/// Return all the "corner" vertices of a given solid.
/// Note that a circle in Revit consists of two arcs
/// and will return a "corner" at each of the two arc
/// end points.
/// </summary>
Dictionary<XYZ,int> GetCorners( Solid solid )
{
  Dictionary<XYZ, int> corners
    = new Dictionary<XYZ, int>(
      new XyzEqualityComparer() );

  foreach( Face f in solid.Faces )
  {
    foreach( EdgeArray ea in f.EdgeLoops )
    {
      foreach( Edge e in ea )
      {
        XYZ p = e.AsCurveFollowingFace( f )
          .get\_EndPoint( 0 );

        if( !corners.ContainsKey( p ) )
        {
          corners[p] = 0;
        }
        ++corners[p];
      }
    }
  }
  return corners;
}
```

The solid is retrieved by traversing the element geometry and picking the first non-empty one found.

Special handling is required for family instances, of course, since they have an additional transformation that we have to take account of.
The family definition defines its own local coordinate system, and we need to transform the solid from that to the Revit model space.

This implementation processes all the cases we have run into so far correctly and elegantly:
```python
/// <summary>
/// Retrieve the first non-empty solid found for
/// the given element. In case the element is a
/// family instance, it may have its own non-empty
/// solid, in which case we use that. Otherwise we
/// search the symbol geometry. If we use the
/// symbol geometry, we have to keep track of the
/// instance transform to map it to the actual
/// instance project location.
/// </summary>
Solid GetSolid( Element e, Options opt )
{
  GeometryElement geo = e.get\_Geometry( opt );

  Solid solid = null;
  GeometryInstance inst = null;
  Transform t = Transform.Identity;

  // Some columns have no solids, and we have to
  // retrieve the geometry from the symbol;
  // others do have solids on the instance itself
  // and no contents in the instance geometry
  // (e.g. in rst\_basic\_sample\_project.rvt).

  foreach( GeometryObject obj in geo )
  {
    solid = obj as Solid;

    if( null != solid
      && 0 < solid.Faces.Size )
    {
      break;
    }

    inst = obj as GeometryInstance;
  }

  if( null == solid && null != inst )
  {
    geo = inst.GetSymbolGeometry();
    t = inst.Transform;

    foreach( GeometryObject obj in geo )
    {
      solid = obj as Solid;

      if( null != solid
        && 0 < solid.Faces.Size )
      {
        break;
      }
    }
  }
  return solid;
}
```

#### Transforming Revit Model Coordinates to the Real World

After the solid vertices have been retrieved in Revit model space, we convert them to real-world coordinates using the project location.

We initially tried to achieve this piecewise by fiddling with the base point offset and manually rotating to project north.
This was both complicated and returned incorrect results.

The correct solution is very simple and was already discussed a couple of times in the past:

- [Transformations](http://thebuildingcoder.typepad.com/blog/2010/01/transformations.html)- [Project location](http://thebuildingcoder.typepad.com/blog/2010/01/project-location.html)- [Transformed family instance geometry](http://thebuildingcoder.typepad.com/blog/2011/06/get-transformed-family-instance-geometry.html)

This led us to implement the following GetProjectLocationTransform method:
```csharp
/// <summary>
/// Return the project location transform.
/// </summary>
Transform GetProjectLocationTransform( Document doc )
{
  // Retrieve the active project location position.

  ProjectPosition projectPosition
    = doc.ActiveProjectLocation.get\_ProjectPosition(
      XYZ.Zero );

  // Create a translation vector for the offsets

  XYZ translationVector = new XYZ(
    projectPosition.EastWest,
    projectPosition.NorthSouth,
    projectPosition.Elevation );

  Transform translationTransform
    = Transform.get\_Translation(
      translationVector );

  // Create a rotation for the angle about true north

  Transform rotationTransform
    = Transform.get\_Rotation( XYZ.Zero,
      XYZ.BasisZ, projectPosition.Angle );

  // Combine the transforms

  Transform finalTransform
    = translationTransform.Multiply(
      rotationTransform );

  return finalTransform;
}
```

Making use of the resulting transform later is trivial:
```csharp
  Transform projectLocationTransform
    = GetProjectLocationTransform( doc );

  for each concrete corner point XYZ p:
  {
    // Transform insertion point by applying
    // project location transformation.

    XYZ r2 = projectLocationTransform.OfPoint( p );
  }
```

Now I just need to find some more time to discuss how this can all be put together and wrapped into a useful real-world end-user application.