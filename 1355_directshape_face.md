---
ai_optimized: true
author: Jeremy Tammik
complexity_score: 7.7
content_type: qa
optimization_date: '2025-12-11T11:44:15.833572'
original_url: https://thebuildingcoder.typepad.com/blog/1355_directshape_face.html
post_number: '1355'
reading_time_minutes: 16
series: general
slug: directshape_face
source_file: 1355_directshape_face.md
tags:
- csharp
- elements
- family
- filtering
- geometry
- levels
- parameters
- python
- references
- revit-api
- selection
- sheets
- transactions
- views
title: Directshape Face
word_count: 3257
---

### DirectShape From Face and Sketch Plane Reuse
Frode Tørresdal of [Norconsult Informasjonssystemer AS](https://www.nois.no) just raised an interesting issue regarding the creation of a DirectShape element on an interactively selected existing element face.
As it turned out, his specific issue was not related to the DirectShape creation at all, but rather to the transformation that needs to be applied to the face of a family instance returned by the PickObject method to convert it from the family symbol space to the family instance real world coordinates.
Here is our discussion of the problem and the evolution of my
[DirectShapeFromFace](https://github.com/jeremytammik/DirectShapeFromFace) solution that I implemented to address this.
It ends up demonstrating several interesting aspects:
- [Creating a DirectShape element from a face mesh](#2)
- [Determining the real world transform of a family instance face returned by PickObject](#3)
- [Iterating over element geometry to find a specific target geometry object](#4)
- [Reusing sketch planes for model curve creation](#5)
- [Complete solution](#6)
- [Download](#7)
#### Creating a DirectShape Element from a Face Mesh
\*\*Question:\*\*
I have some issues when creating DirectShape elements in Revit. I attached a sample project and a Revit model:
![Faces for DirectShape](img/faces_for_directshape.png)
The sample project creates a DirectShape from a selected face.
Here is the code:

```
  public static void Execute1(
    ExternalCommandData commandData )
  {
    Transaction trans = null;

    UIDocument uidoc = commandData.Application
      .ActiveUIDocument;

    Document doc = uidoc.Document;

    try
    {
      Selection choices = uidoc.Selection;

      Reference reference = choices.PickObject(
        ObjectType.Face );

      Element el = doc.GetElement(
        reference.ElementId );

      trans = new Transaction( doc, "Create elements" );
      trans.Start();

      TessellatedShapeBuilder builder
        = new TessellatedShapeBuilder();

      builder.OpenConnectedFaceSet( false );

      Face face = el.GetGeometryObjectFromReference(
        reference ) as Face;

      Mesh mesh = face.Triangulate();
      List<XYZ> args = new List<XYZ>( 3 );

      XYZ offset = new XYZ();
      if( el.Location is LocationPoint )
      {
        LocationPoint locationPoint = el.Location
          as LocationPoint;
        offset = locationPoint.Point;
      }

      for( int i = 0; i < mesh.NumTriangles; i++ )
      {
        MeshTriangle triangle = mesh.get_Triangle(
          i );

        XYZ p1 = triangle.get_Vertex( 0 );
        XYZ p2 = triangle.get_Vertex( 1 );
        XYZ p3 = triangle.get_Vertex( 2 );

        p1 = p1.Add( offset );
        p2 = p2.Add( offset );
        p3 = p3.Add( offset );

        args.Clear();
        args.Add( p1 );
        args.Add( p2 );
        args.Add( p3 );
        TessellatedFace tesseFace
          = new TessellatedFace( args,
            ElementId.InvalidElementId );

        if( builder.DoesFaceHaveEnoughLoopsAndVertices(
          tesseFace ) )
        {
          builder.AddFace( tesseFace );
        }
      }

      builder.CloseConnectedFaceSet();

      TessellatedShapeBuilderResult result
        = builder.Build(
          TessellatedShapeBuilderTarget.AnyGeometry,
          TessellatedShapeBuilderFallback.Mesh,
          ElementId.InvalidElementId );

      ElementId categoryId = new ElementId(
        BuiltInCategory.OST_GenericModel );

      DirectShape ds = DirectShape.CreateElement(
        doc, categoryId,
        Assembly.GetExecutingAssembly().GetType()
        .GUID.ToString(), Guid.NewGuid().ToString() );

      ds.SetShape( result.GetGeometricalObjects() );

      ds.Name = "MyShape";

      trans.Commit();
    }
    catch( Exception ex )
    {
      if( trans != null )
        trans.RollBack();

      Debug.Print( ex.Message );
    }
  }
```

The code works fine on the columns. I managed this by calculating an offset from the LocationPoint. But this unfortunately affects the big Generic Model. This Generic Model element has been moved and when I run the code on this object, the DirectShape is created in the wrong location. If I remove the lines that get the offset from the location point this works fine on the generic model, but not on the columns. How do I know when to use the offset? Is there a way to write code that works on both cases?
Also in this model there are two columns. The code works fine on one of them, but not the other. Why is that?
I really hope you can help me with this!
#### Determining the Real World Transform of a Family Instance Face returned by PickObject
\*\*Answer:\*\*
Thank you for your interesting query and sample material.
I compiled the add-in and can reproduce the behaviour you describe.
It looks to me as if you are not taking the translation of the selected element properly into account.
Just like you, I would have expected the GetGeometryObjectFromReference method to do that automatically for me.
Maybe it will work better and work for all types of elements if you use a different approach to retrieve the geometry.
For family instances, there is a difference between symbol geometry and the translated family instance geometry.
I have made good experiences using the GeometryElement.GetTransformed method and passing in an identity transform to retrieve element geometry in its real world location.
You can possibly use the geometry reference returned by the pick operation to select the same face from the element geometry with the option ComputeReferences turned on.
Another, more pertinent question:
Are you sure this is an issue with the DirectShape?
I would have thought that it is more an issue of the geometry retrieval, and has nothing to do with the DirectShape creation.
Therefore, I would suggest the following test of the intermediate results:
1. Retrieve the three MeshTriangle vertices.
2. Create a plane containing all three, and a sketch plane. For efficiency, we can cache and reuse already existing sketch planes.
3. Draw three model lines representing the mesh triangle.
Then you can see exactly what geometry is being returned.
I cleaned up your sample a little bit to test that myself.
I created the [DirectShapeFromFace GitHub repository](https://github.com/jeremytammik/DirectShapeFromFace) for it to keep track of my modifications.
You might want to take a look at that in its current state, and maybe synchronise your sample with mine.
I have not finished yet, though, and am still working on it.
I am sure we will find a perfect resolution for this.
\*\*Update:\*\*
I updated my sample code to create model lines displaying the original triangles obtained from the element geometry via the GetGeometryObjectFromReference method.
Look at the new [release 2016.0.0.1](https://github.com/jeremytammik/DirectShapeFromFace/releases/tag/2016.0.0.1).
I can confirm that the mesh triangles are located in a different place than the original element.
There is certainly a really simple solution to this.
In any case, the problem has nothing to do with the DirectShape creation, just with the geometry retrieval.
I have seen and handled similar issues in the past, when traversing element geometry for various export processes.
In those cases, I was using geometry retrieved from the `Element.get_Geometry` property instead of the `GetGeometryObjectFromReference` method.
I therefore know how this issue can be handled.
I would still like to learn what the optimal, simplest and most efficient approach really is.
The Revit API often moves in mysterious ways...
\*\*Question simplified:\*\*
The following geometry retrieval returns a face in a different project location that the original selected element:

```
  Selection choices = uidoc.Selection;

  Reference reference = choices.PickObject(
    ObjectType.Face );

  Element el = doc.GetElement(
    reference.ElementId );

  Face face = el.GetGeometryObjectFromReference(
    reference ) as Face;
```

What is the proper and efficient way to obtain the face in the same location as the original selected element?
The face obtained as shown above is in an unexpected location, often far away from the selected element.
Apparently, the problem is that `PickObject` returns a reference to a face, and that face may be in the symbol geometry, not the instance geometry.
How can I find the correct transformation to the instance geometry location?
I tried applying `FamilyInstance.GetTransform` to it, to no avail.
I also tried iterating through all the (possibly nested) element geometry instances to calculate the appropriate transform, but I cannot find any way to identify the face returned by PickObject.
Both the equality operator `==` and a comparison using the `Face.Reference` property always return false for all the faces that I find.
Very mystifying.
\*\*Answer:\*\*
It is possible that some families (and their representative geometry) are nested several levels deep, and you need all the transforms.
You should be able to compare references by the strings returned by the `ConvertToStableRepresentation` method.
It would be a nice enhancement to make `Reference.Equals` work in this manner. Unfortunately, it does not currently do so.
\*\*Response:\*\*
That really sounds quite bad.
`FamilyInstance.GetTransform` works for some family instances and not for others.
I would love that to work, then the problem would be almost resolved.
Otherwise I have to resort to determining the transform myself.
When a user calls `PickObject( ObjectType.Face )`, nothing is known except the resulting element id and reference to the face.
Are you really telling me that at this point I have to:
1. Check whether the element happens to be a family instance.
2. If so, determine the selected face's ConvertToStableRepresentation string, iterate through all the geometry, possibly through several levels of nested family instances, keep track of all the transforms, find and identify the picked face by checking and comparing the ConvertToStableRepresentation string, use the result of that to decide at which point I need to stop traversing the geometry, exit the traversal when the target is found, put together the list of nested transforms in the proper manner, and finally apply the resulting total transform to the selected face?
I think that should be packaged and provided by the API.
Could you provide example code that implements this in the correct manner?
I have tried to achieve this and not succeeded so far, e.g., like this:

```
///
/// Determine the stack of transforms to apply to
/// the given target geometry object to bring it
/// to the proper location in the project coordinates.
/// Unfortunetely, we have not found any way at all
/// yet to identify the target object we are after.
///
static bool GetTransformStackForObject(
  Stack<Transform> tstack,
  GeometryElement geo,
  GeometryObject targetObj,
  Reference targetRef )
{
  foreach( GeometryObject obj in geo )
  {
    if( obj == targetObj )
    {
      return true;
    }

    GeometryInstance gi = obj as GeometryInstance;

    if( null != gi )
    {
      tstack.Push( gi.Transform );
      return GetTransformStackForObject( tstack,
        gi.GetInstanceGeometry(), targetObj, targetRef );
    }

    Solid solid = obj as Solid;
    if( null != solid )
    {
      if( 0 < solid.Faces.Size )
      {
        foreach( Face face in solid.Faces )
        {
          if( face == targetObj )
          {
            return true;
          }
          if( face.Reference == targetRef )
          {
            return true;
          }
        }
      }
      if( 0 < solid.Edges.Size )
      {
        foreach( Edge edge in solid.Edges )
        {
          if( edge == targetObj )
          {
            return true;
          }
          if( edge.Reference == targetRef )
          {
            return true;
          }
        }
      }
    }
  }
  return false;
}
```

This code never found the target.
Maybe it will if I use `ConvertToStableRepresentation`.
Still, I would appreciate further input and confirmation before I continue in these struggles.
I think 'PickObject( ObjectType.Face )' is pretty hard to use out of the box if it requires all these additional calculations...
\*\*Response 2:\*\*
After further testing, I still cannot identify the picked face within the element geometry in the manner suggested.
Here are the relevant code snippets:

```
  Selection choices = uidoc.Selection;

  Reference faceref = choices.PickObject(
    ObjectType.Face );

  string rep = faceref
    .ConvertToStableRepresentation( doc );

  Debug.Print( "Face reference picked: "
    + rep );

  Element el = doc.GetElement(
    faceref.ElementId );

...

  Transform t = null;

  Options opt = new Options();
  opt.ComputeReferences = true;
  GeometryElement geo = el.get_Geometry( opt );
  Stack<Transform> tstack = new Stack<Transform>();

  if( GetTransformStackForObject( tstack, geo, doc, rep )
    && 0 < tstack.Count )
  {
    t = Transform.Identity;

    while( 0 < tstack.Count )
    {
      t *= tstack.Pop();
    }
  }

...

///
/// Determine the stack of transforms to apply to
/// the given target geometry object to bring it
/// to the proper location in the project coordinates.
/// Unfortunetely, we have not found any way at all
/// yet to identify the target object we are after.
///
static bool GetTransformStackForObject(
  Stack<Transform> tstack,
  GeometryElement geo,
  Document doc,
  string stable_representation )
{
  foreach( GeometryObject obj in geo )
  {
    GeometryInstance gi = obj as GeometryInstance;

    if( null != gi )
    {
      tstack.Push( gi.Transform );

      return GetTransformStackForObject( tstack,
        gi.GetInstanceGeometry(), doc,
        stable_representation );
    }

    Solid solid = obj as Solid;

    if( null != solid )
    {
      string rep;

      if( 0 < solid.Faces.Size )
      {
        foreach( Face face in solid.Faces )
        {
          rep = face.Reference
            .ConvertToStableRepresentation( doc );

          if( rep.Equals( stable_representation ) )
          {
            return true;
          }
        }
      }
      if( 0 < solid.Edges.Size )
      {
        foreach( Edge edge in solid.Edges )
        {
          rep = edge.Reference
            .ConvertToStableRepresentation( doc );

          if( rep.Equals( stable_representation ) )
          {
            return true;
          }
        }
      }
    }
  }
  return false;
}
```

I tested this with a structural concrete rectangular column.
All the faces were visited, and none of them returned a `ConvertToStableRepresentation` string that matched the picked face's one.
#### Iterating over Element Geometry to Find a Specific Target Geometry Object
\*\*Answer:\*\*
Short answer:
`GetInstanceGeometry` is incorrect in extracting usable references – you must call `GetSymbolGeometry` with no arguments instead.
This should result in comparable stable references.
More details of instance transforms etc. are available in Scott Conover's AU courses and have been rolled into material that is in the Developer’s Guide in the Revit online help.
\*\*Response:\*\*
Thank you!
That helped.
It seems to be working now, in [DirectShapeFromFace release 2016.0.0.5](https://github.com/jeremytammik/DirectShapeFromFace/releases/tag/2016.0.0.5):
![DirectShape from face](img/direct_shape_from_face.png)
I am surprised it is so hard.
The sequence of prior attempts and tests is described by the preceding [release messages](https://github.com/jeremytammik/DirectShapeFromFace/releases).
It still needs to be tested with more samples, especially with faces nested several levels deep within nested families.
#### Reusing Sketch Planes for Model Curve Creation
One final cool implementation details to note:
For testing purposes, I create model lines representing the original face mesh triangles as well the final direct shape.
The model lines require a sketch plane to host them.
To avoid recreating hundreds and thousands of identical sketch planes for this purpose, I try to reuse the existing ones as much as possible.
In tried to limit the reuse to my own sketch planes and mark them by specifying their element name, but that does not work.
The always end up named "".
So I modified my reusage algorithm to reuse only such sketch planes, and it seems to work fine, cf. the `SketchPlaneMatches` and `GetSketchPlane` methods below.
#### Complete Solution
To wrap up, here is the complete code implementing this:

```
class CreateDirectShape
{
  const string _sketch_plane_name_prefix
    = "The Building Coder";

  const string _sketch_plane_name_prefix2
    = "";

  #region Geometrical Comparison
  const double _eps = 1.0e-9;

  public static bool IsAlmostZero(
    double a,
    double tolerance )
  {
    return tolerance > Math.Abs( a );
  }

  public static bool IsAlmostZero( double a )
  {
    return IsAlmostZero( a, _eps );
  }

  public static bool IsAlmostEqual( double a, double b )
  {
    return IsAlmostZero( b - a );
  }
  #endregion // Geometrical Comparison

  ///
  /// Return the normal of a plane
  /// spanned by the two given vectors.
  ///
  static XYZ GetNormal( XYZ v1, XYZ v2 )
  {
    return v1
      .CrossProduct( v2 )
      .Normalize();
  }

  ///
  /// Return the normal of a plane spanned by the
  /// three given triangle corner points.
  ///
  static XYZ GetNormal( XYZ[] triangleCorners )
  {
    return GetNormal(
      triangleCorners[1] - triangleCorners[0],
      triangleCorners[2] - triangleCorners[0] );
  }

  ///
  /// Return signed distance from plane to a given point.
  ///
  public static double SignedDistanceTo(
    Plane plane,
    XYZ p )
  {
    Debug.Assert(
      IsAlmostEqual( plane.Normal.GetLength(), 1 ),
        "expected normalised plane normal" );

    XYZ v = p - plane.Origin;

    return plane.Normal.DotProduct( v );
  }

  ///
  /// Return true if the sketch plane belongs to us
  /// and its origin and normal vector match the
  /// given targets.
  /// Nope, we are unable to set the sketch plane
  /// name. However, Revit throws an exception if
  /// we try to draw on the skatch plane named
  /// 'Level 1', so lets ensure we use '
  /// associated>'.
  ///
  static bool SketchPlaneMatches(
    SketchPlane sketchPlane,
    XYZ origin,
    XYZ normal )
  {
    //bool rc = sketchPlane.Name.StartsWith(
    //  _sketch_plane_name_prefix );

    bool rc = sketchPlane.Name.Equals(
      _sketch_plane_name_prefix2 );

    if( rc )
    {
      Plane plane = sketchPlane.GetPlane();

      rc = plane.Normal.IsAlmostEqualTo( normal )
        && IsAlmostZero( SignedDistanceTo(
          plane, origin ) );
    }
    return rc;
  }

  static int _sketch_plane_creation_counter = 0;

  ///
  /// Return a sketch plane through the given origin
  /// point with the given normal, either by creating
  /// a new one or reusing an existing one.
  ///
  static SketchPlane GetSketchPlane(
    Document doc,
    XYZ origin,
    XYZ normal )
  {
    string s = "reusing";

    // If we could reliably set the sketch plane Name
    // property or find some other relaible marker
    // that is reflected in a parameter, we could
    // replace the sketchPlane.Name.Equals check in
    // SketchPlaneMatches by a parameter filter in
    // the filtered element collector framework
    // to move the test into native Revit code
    // instead of post-processing in .NET, which
    // would give a 50% performance enhancement.

    SketchPlane sketchPlane
      = new FilteredElementCollector( doc )
        .OfClass( typeof( SketchPlane ) )
        .Cast<SketchPlane>()
        .FirstOrDefault<SketchPlane>( x =>
          SketchPlaneMatches( x, origin, normal ) );

    if( null == sketchPlane )
    {
      Plane plane = new Plane( normal, origin );

      sketchPlane = SketchPlane.Create( doc, plane );

      //sketchPlane.Name = string.Format(
      //  "{0} {1}", _sketch_plane_name_prefix,
      //  _sketch_plane_creation_counter++ );

      ++_sketch_plane_creation_counter;

      s = "created";
    }
    Debug.Print( "GetSketchPlane: {0} '{1}' ({2})",
      s, sketchPlane.Name,
      _sketch_plane_creation_counter );

    return sketchPlane;
  }

  ///
  /// Create model lines representing a closed
  /// planar loop in the given sketch plane.
  ///
  static void DrawModelLineLoop(
    SketchPlane sketchPlane,
    XYZ[] corners )
  {
    Autodesk.Revit.Creation.Document factory
      = sketchPlane.Document.Create;

    int n = corners.GetLength( 0 );

    for( int i = 0; i < n; ++i )
    {
      int j = 0 == i ? n - 1 : i - 1;

      factory.NewModelCurve( Line.CreateBound(
        corners[j], corners[i] ), sketchPlane );
    }
  }

  ///
  /// Determine the stack of transforms to apply to
  /// the given target geometry object to bring it
  /// to the proper location in the project coordinates.
  /// Unfortunetely, we have not found any way at all
  /// yet to identify the target object we are after.
  ///
  static bool GetTransformStackForObject(
    Stack<Transform> tstack,
    GeometryElement geo,
    Document doc,
    string stable_representation )
  {
    Debug.Print( "enter GetTransformStackForObject "
      + "with tstack count {0}", tstack.Count );

    bool found = false;

    foreach( GeometryObject obj in geo )
    {
      GeometryInstance gi = obj as GeometryInstance;

      if( null != gi )
      {
        tstack.Push( gi.Transform );

        found = GetTransformStackForObject( tstack,
          gi.GetSymbolGeometry(), doc,
          stable_representation );

        if( found ) { return found; }

        tstack.Pop();

        continue;
      }

      Solid solid = obj as Solid;

      if( null != solid )
      {
        string rep;

        bool isFace = stable_representation.EndsWith(
          "SURFACE" );

        bool isEdge = stable_representation.EndsWith(
          "LINEAR" );

        Debug.Assert( isFace || isEdge,
          "GetTransformStackForObject currently only supports faces and edges" );

        if( isFace && 0 < solid.Faces.Size )
        {
          foreach( Face face in solid.Faces )
          {
            rep = face.Reference
              .ConvertToStableRepresentation( doc );

            if( rep.Equals( stable_representation ) )
            {
              return true;
            }
          }
        }

        if( isEdge && 0 < solid.Edges.Size )
        {
          foreach( Edge edge in solid.Edges )
          {
            rep = edge.Reference
              .ConvertToStableRepresentation( doc );

            if( rep.Equals( stable_representation ) )
            {
              return true;
            }
          }
        }
      }
    }
    return false;
  }

  public static void Execute(
    ExternalCommandData commandData )
  {
    Transaction trans = null;

    UIApplication uiapp = commandData.Application;
    UIDocument uidoc = uiapp.ActiveUIDocument;
    Document doc = uidoc.Document;

    try
    {
      Selection choices = uidoc.Selection;

      Reference faceref = choices.PickObject(
        ObjectType.Face );

      string rep = faceref
        .ConvertToStableRepresentation( doc );

      Debug.Assert( rep.EndsWith( ":SURFACE" ),
        "expected stable representation to end with SURFACE" );

      Debug.Print( "Face reference picked: "
        + rep );

      Element el = doc.GetElement(
        faceref.ElementId );

      using( trans = new Transaction( doc ) )
      {
        trans.Start( "Create elements" );

        TessellatedShapeBuilder builder
          = new TessellatedShapeBuilder();

        builder.OpenConnectedFaceSet( false );

        // This may return a face in the family
        // symbol definition with no family instance
        // transform applied. Use the GeometryElement
        // GetTransformed method to retrieve the face
        // with the instance transformation applied.

        Face face = el.GetGeometryObjectFromReference(
          faceref ) as Face;

        Debug.Print( "Face reference property: "
          + ( ( null == face.Reference )
            ? ""
            : face.Reference.ConvertToStableRepresentation( doc ) ) );

        Transform t = null;

        FamilyInstance fi = el as FamilyInstance;

        if( null != fi )
        {
          // Will this handle a face selected
          // in a nested family instance?
          // Some, yes, but not all.

          //t = fi.GetTransform();

          // This also works for some instances
          // but not all.

          //Transform t1 = fi.GetTotalTransform();

          Options opt = new Options();
          opt.ComputeReferences = true;

          GeometryElement geo = el.get_Geometry( opt );

          GeometryElement geo2 = geo.GetTransformed(
            Transform.Identity );

          Stack<Transform> tstack
            = new Stack<Transform>();

          if( GetTransformStackForObject( tstack,
            geo, doc, rep ) && 0 < tstack.Count )
          {
            Debug.Print( "GetTransformStackForObject "
              + "returned true with tstack count {0}",
              tstack.Count );

            t = Transform.Identity;

            while( 0 < tstack.Count )
            {
              t *= tstack.Pop();
            }
          }
        }

        Mesh mesh = face.Triangulate();

        if( null != t )
        {
          mesh = mesh.get_Transformed( t );
        }

        XYZ[] triangleCorners = new XYZ[3];

        for( int i = 0; i < mesh.NumTriangles; i++ )
        {
          MeshTriangle triangle = mesh.get_Triangle( i );

          triangleCorners[0] = triangle.get_Vertex( 0 );
          triangleCorners[1] = triangle.get_Vertex( 1 );
          triangleCorners[2] = triangle.get_Vertex( 2 );

          XYZ normal = GetNormal( triangleCorners );

          SketchPlane sketchPlane = GetSketchPlane(
            doc, triangleCorners[0], normal );

          DrawModelLineLoop( sketchPlane, triangleCorners );

          TessellatedFace tesseFace
            = new TessellatedFace( triangleCorners,
              ElementId.InvalidElementId );

          if( builder.DoesFaceHaveEnoughLoopsAndVertices(
            tesseFace ) )
          {
            builder.AddFace( tesseFace );
          }
        }

        builder.CloseConnectedFaceSet();

        TessellatedShapeBuilderResult result
          = builder.Build(
            TessellatedShapeBuilderTarget.AnyGeometry,
            TessellatedShapeBuilderFallback.Mesh,
            ElementId.InvalidElementId );

        ElementId categoryId = new ElementId(
          BuiltInCategory.OST_GenericModel );

        DirectShape ds = DirectShape.CreateElement(
          doc, categoryId,
          Assembly.GetExecutingAssembly().GetType().GUID.ToString(),
          Guid.NewGuid().ToString() );

        ds.SetShape( result.GetGeometricalObjects() );

        ds.Name = "MyShape";

        trans.Commit();
      }
    }
    catch( Exception ex )
    {
      TaskDialog.Show( "Error", ex.Message );
    }
  }
}
```

#### Download
The most up-to-date version, complete Visual Studio solution and add-in manifest is provided by the
[DirectShapeFromFace GitHub repository](https://github.com/jeremytammik/DirectShapeFromFace).
The version discussed here is
[release 2016.0.0.9](https://github.com/jeremytammik/DirectShapeFromFace/releases/tag/2016.0.0.9)
I hope you find this as interesting and useful as I do.
Many thanks to Frode for raising the issue and providing the original code to create the DirectShape element.