---
ai_optimized: true
author: Jeremy Tammik
complexity_score: 7.9
content_type: code_example
optimization_date: '2025-12-11T11:44:14.132078'
original_url: https://thebuildingcoder.typepad.com/blog/0539_dim_walls_it_face.html
post_number: 0539
reading_time_minutes: 11
series: general
slug: dim_walls_it_face
source_file: 0539_dim_walls_it_face.htm
tags:
- csharp
- elements
- family
- geometry
- levels
- parameters
- python
- references
- revit-api
- selection
- transactions
- views
- walls
title: Dimension Walls by Iterating Faces
word_count: 2192
---

### Dimension Walls by Iterating Faces

I have not seen many published examples of creating dimensioning elements using the Revit API.

The one sample that is provided by the Revit SDK, CreateDimensions, shows how to add dimensioning between the start and end points of selected structural wall, but is limited to exactly that, dimensioning a structural wall, and only in Revit Structure.

To create a dimensioning element, one has to provide an array of stable references to some geometrical features of the building geometry.
CreateDimensions simplifies the process of providing the references by using the wall's analytical model, which is only accessible in Revit Structure.

The
[Family API labs](http://thebuildingcoder.typepad.com/blog/2009/10/revit-family-creation-api-labs.html) provide
another example of
[setting up a reference array](http://thebuildingcoder.typepad.com/blog/2009/10/revit-family-creation-api-labs.html#1) in
order to create some dimensioning and alignment elements.

So how about creating some dimensioning elements with no additional support from the analytical model, that runs in all flavours of Revit?

Well, there are several ways to obtain references through the Revit API, two of which are:

- Query the element geometry and ask it to
  [compute references](http://thebuildingcoder.typepad.com/blog/2010/01/geometry-options.html).- Use the
    [FindReferencesByDirection](http://thebuildingcoder.typepad.com/blog/2010/01/findreferencesbydirection.html) method.

Let's explore the usage of these two approaches to create a dimensioning element between two walls, prompted by the following question:

**Question:** Could you please provide the simplest possible code that shows dimensioning between two inside surfaces of two walls in plan view?
For example, I have a shaft and I want to dimension its inside width and height.

**Answer:** Before getting into the nuts and bolts of determining references on existing walls, let's discuss some basics:

#### Creating Linear Dimensioning

Linear dimensioning such as you describe is created using the NewDimension method provided by the Creation.Document class.
The Revit SDK developer guide "Revit 2011 API Developer Guide.pdf" includes a snippet of sample code demonstrating how use this method in Code Region 16-3: Duplicating a dimension with NewDimension:
```csharp
  public void DuplicateDimension(
    Document doc,
    Dimension dimension )
  {
    Line line = dimension.Curve as Line;

    if( null != line )
    {
      Autodesk.Revit.DB.View view = dimension.View;

      ReferenceArray references = dimension.References;

      Dimension newDimension = doc.Create.NewDimension(
        view, line, references );
    }
  }
```

Another sample of interest in this context is Code Region 13-9: Labelling a dimension, since it shows in more detail how the view, reference array, and line arguments can be determined and populated.
It works in the context of a family document, however:
```csharp
  public Dimension CreateLinearDimension(
    Document doc )
  {
    Application app = doc.Application;

    // first create two lines

    XYZ pt1 = new XYZ( 5, 5, 0 );
    XYZ pt2 = new XYZ( 5, 10, 0 );
    Line line = app.Create.NewLine( pt1, pt2, true );
    Plane plane = app.Create.NewPlane(
      pt1.CrossProduct( pt2 ), pt2 );

    SketchPlane skplane = doc.FamilyCreate
      .NewSketchPlane( plane );

    ModelCurve modelcurve1 = doc.FamilyCreate
      .NewModelCurve( line, skplane );

    pt1 = new XYZ( 10, 5, 0 );
    pt2 = new XYZ( 10, 10, 0 );
    line = app.Create.NewLine( pt1, pt2, true );
    plane = app.Create.NewPlane(
      pt1.CrossProduct( pt2 ), pt2 );

    skplane = doc.FamilyCreate
      .NewSketchPlane( plane );

    ModelCurve modelcurve2 = doc.FamilyCreate
      .NewModelCurve( line, skplane );

    // now create a linear dimension between them

    ReferenceArray ra = new ReferenceArray();
    ra.Append( modelcurve1.GeometryCurve.Reference );
    ra.Append( modelcurve2.GeometryCurve.Reference );

    pt1 = new XYZ( 5, 10, 0 );
    pt2 = new XYZ( 10, 10, 0 );
    line = app.Create.NewLine( pt1, pt2, true );
    Dimension dim = doc.FamilyCreate
      .NewLinearDimension( doc.ActiveView, line, ra );

    // create a label for the dimension called "width"

    FamilyParameter param = doc.FamilyManager
      .AddParameter( "width",
        BuiltInParameterGroup.PG\_CONSTRAINTS,
        ParameterType.Length, false );

    dim.Label = param;

    return dim;
  }
```

In this sample code, two new model lines are created and the dimension element is generated between them.

#### Determining References to Wall Faces

Returning to your question: to dimension two existing walls instead, you need to obtain references to their respective inside surfaces and pass them in to the NewLinearDimension method.

I discussed various methods to determine specific surfaces on the solids of walls and other elements, e.g.

- [Slab boundary](http://thebuildingcoder.typepad.com/blog/2008/10/slab-boundary.html).
- [Slab side faces](http://thebuildingcoder.typepad.com/blog/2008/11/slab-side-faces.html).
- [Wall elevation profile](http://thebuildingcoder.typepad.com/blog/2008/11/wall-elevation-profile.html).
- [2D polygon areas and outer loop](http://thebuildingcoder.typepad.com/blog/2008/12/2d-polygon-areas-and-outer-loop.html).
- [3D polygon areas](http://thebuildingcoder.typepad.com/blog/2008/12/3d-polygon-areas.html).

For the case you describe, one possibility would be to determine the two closest parallel vertical faces of two respective selected opposing walls.

When querying the wall geometry for their solids and faces, one needs to explicitly specify that references be computed, since these are required for the dimensioning creation.

Since I was unable to find a simple ready-made sample that demonstrates this, and your query is a very valid one, I implemented a new Building Coder sample command CmdDimensionWallsIterateFaces to show the exact steps required.

It is as short and simple as I was able to make it, but still rather complex for a Revit API beginner.

First of all, it makes use of a couple of trivial geometrical comparison and helper methods:
```csharp
  const double \_eps = 1.0e-9;

  /// <summary>
  /// Check whether two real numbers are equal
  /// </summary>
  static public bool IsEqual( double a, double b )
  {
    return Math.Abs( a - b ) < \_eps;
  }

  /// <summary>
  /// Check whether two vectors are parallel
  /// </summary>
  static public bool IsParallel( XYZ a, XYZ b )
  {
    double angle = a.AngleTo( b );
    return \_eps > angle || IsEqual( angle, Math.PI );
  }

  /// <summary>
  /// Return the midpoint between two points.
  /// </summary>
  static XYZ Midpoint( XYZ p, XYZ q )
  {
    return p + 0.5 \* ( q - p );
  }

  /// <summary>
  /// Return the midpoint of a Line.
  /// </summary>
  static XYZ Midpoint( Line line )
  {
    return Midpoint( line.get\_EndPoint( 0 ),
      line.get\_EndPoint( 1 ) );
  }

  /// <summary>
  /// Return the normal of a Line in the XY plane.
  /// </summary>
  static XYZ Normal( Line line )
  {
    XYZ p = line.get\_EndPoint( 0 );
    XYZ q = line.get\_EndPoint( 1 );
    XYZ v = q - p;

    //Debug.Assert( IsZero( v.Z ),
    //  "expected horizontal line" );

    return v.CrossProduct( XYZ.BasisZ ).Normalize();
  }
```

Using these, we can implement a helper method GetClosestFace which returns the closest planar face with a given normal vector to a given point p on the element e:
```csharp
static Face GetClosestFace(
  Element e,
  XYZ p,
  XYZ normal,
  Options opt )
{
  Face face = null;
  double min\_distance = double.MaxValue;
  GeometryElement geo = e.get\_Geometry( opt );
  GeometryObjectArray objects = geo.Objects;
  foreach( GeometryObject obj in objects )
  {
    Solid solid = obj as Solid;
    if( solid != null )
    {
      FaceArray fa = solid.Faces;
      foreach( Face f in fa )
      {
        PlanarFace pf = f as PlanarFace;

        Debug.Assert( null != pf,
          "expected planar wall faces" );

        if( null != pf
          && Util.IsParallel( normal, pf.Normal ) )
        {
          XYZ v = p - pf.Origin;
          double d = v.DotProduct( -pf.Normal );
          if( d < min\_distance )
          {
            face = f;
            min\_distance = d;
          }
        }
      }
    }
  }
  return face;
}
```

Once the references on the closest opposing wall faces have been determined and collected into an array, the following helper method creates the dimensioning.

It creates a new dimension element using the given references and dimension line end points.
It also opens and commits its own transaction, assuming that no transaction is open yet and manual transaction mode is being used.

Note that I only tested this so far using references to surfaces on planar walls in a plan view:
```csharp
public static void CreateDimensionElement(
  View view,
  XYZ p1,
  Reference r1,
  XYZ p2,
  Reference r2 )
{
  Document doc = view.Document;
  Application app = doc.Application;

  // creation objects, or factories, for database
  // and non-database resident instances:

  Autodesk.Revit.Creation.Application creApp
    = app.Create;

  Autodesk.Revit.Creation.Document creDoc
    = doc.Create;

  ReferenceArray ra = new ReferenceArray();

  ra.Append( r1 );
  ra.Append( r2 );

  Line line = creApp.NewLineBound( p1, p2 );

  Transaction t = new Transaction( doc,
    "Dimension Two Walls" );

  t.Start();

  Dimension dim = creDoc.NewDimension(
    doc.ActiveView, line, ra );

  t.Commit();
}
```

Finally, here is the code of the external command and its Execute method putting all of this to use:
```python
/// <summary>
/// Dimension two opposing parallel walls.
/// For simplicity, the dimension is defined from
/// wall midpoint to midpoint, so the walls have
/// to be exactly opposite each other for it to work.
/// Iterate the wall solid faces to find the two
/// closest opposing faces and use references to
/// them to define the dimension element.
/// </summary>
[Transaction( TransactionMode.Manual )]
[Regeneration( RegenerationOption.Manual )]
class CmdDimensionWallsIterateFaces : IExternalCommand
{
  const string \_prompt
    = "Please select two parallel opposing straight walls.";
  public Result Execute(
    ExternalCommandData commandData,
    ref string message,
    ElementSet elements )
  {
    UIApplication uiapp = commandData.Application;
    UIDocument uidoc = uiapp.ActiveUIDocument;
    Application app = uiapp.Application;
    Document doc = uidoc.Document;

    // obtain the current selection and pick
    // out all walls from it:

    Selection sel = uidoc.Selection;
    List<Wall> walls = new List<Wall>( 2 );
    foreach( Element e in sel.Elements )
    {
      if( e is Wall )
      {
        walls.Add( e as Wall );
      }
    }

    if( 2 != walls.Count )
    {
      message = \_prompt;
      return Result.Failed;
    }

    // ensure the two selected walls are straight and
    // parallel; determine their mutual normal vector
    // and a point on each wall for distance
    // calculations:

    List<Line> lines = new List<Line>( 2 );
    List<XYZ> midpoints = new List<XYZ>( 2 );
    XYZ normal = null;

    foreach( Wall wall in walls )
    {
      LocationCurve lc = wall.Location as LocationCurve;
      Curve curve = lc.Curve;

      if( !( curve is Line ) )
      {
        message = \_prompt;
        return Result.Failed;
      }

      Line l = curve as Line;
      lines.Add( l );
      midpoints.Add( Util.Midpoint( l ) );

      if( null == normal )
      {
        normal = Util.Normal( l );
      }
      else
      {
        if( !Util.IsParallel( normal, Util.Normal( l ) ) )
        {
          message = \_prompt;
          return Result.Failed;
        }
      }
    }

    // find the two closest facing faces on the walls;
    // they are vertical faces that are parallel to the
    // wall curve and closest to the other wall.

    Options opt = app.Create.NewGeometryOptions();

    opt.ComputeReferences = true;

    List<Face> faces = new List<Face>( 2 );
    faces.Add( GetClosestFace( walls[0], midpoints[1], normal, opt ) );
    faces.Add( GetClosestFace( walls[1], midpoints[0], normal, opt ) );

    // create the dimensioning:

    CreateDimensionElement( doc.ActiveView,
      midpoints[0], faces[0].Reference,
      midpoints[1], faces[1].Reference );

    return Result.Succeeded;
  }
}
```

For simplicity, the dimension is defined from wall midpoint to midpoint, so the walls have to be exactly opposite for it to work.

Here is a pretty horrible example of some dimensioning elements created by this command, by repeatedly selecting two opposing parallel walls and then launching it:
![Dimensioning two opposing walls](img/DimensionTwoWalls.png)

This sample has the following limitations:

- The dimension line is defined from wall midpoint to wall midpoint.
  Since the dimension line must be perpendicular to the wall faces, this means that the two opposing wall midpoints must be exactly opposite to each other, i.e. the walls must both be centred respective to the other.
  This fits your requirements for a lift shaft, I assume, but is a rather special case.
  A more general approach would allow the user to pick a point on one of the walls to define where the dimension line should be placed.- The creation of the dimension element requires an array of references to the wall solid faces being dimensioned.
    The solution presented iterates over all the wall solid faces to determine the two required faces and their references.
    A completely different approach to obtain these references which may even be simpler than the explicit iteration presented is to use the high-level FindReferencesByDirection Revit API method.
    It shoots a ray through the Revit model and reports the elements and their faces and the points on the faces intersected by the ray.
    This is exactly what we need to decide where the dimension line should be located and which two walls and faces are closest to each other from a given point along a given line.
    This provides an idea for an obvious second sample implementation demonstrating that approach.

#### Determine Normal Vector Using Cross Product

**Response:** Although I have success in getting this part of my dimensioning correct,
I'm not exactly sure how your function 'static XYZ Normal' is working, with the call to v.CrossProduct( XYZ.BasisZ ).Normalize();
I must admit I'm real fuzzy on what is happening here.
Is there any way that I can get some type of good and simple explanation on this line of code is doing?
I'm sure this would help me with my next step of dimensioning.

**Answer:** You ask about the meaning of the cross product
```csharp
  v.CrossProduct( XYZ.BasisZ ).Normalize()
```

That calculates the cross product between two vectors.
Here is an explanation of the term
[Euclidian vector, and its
[length and normalisation](http://en.wikipedia.org/wiki/Euclidean_vector).
The calculation of the
[cross product](http://en.wikipedia.org/wiki/Cross_product) of
two vectors produces a third vector perpendicular to both of the input vectors.
The code snippet above does just that and then normalises the result, i.e. scales it so that its length equals one.

I will continue with this dimensioning topic and address the next step, using FindReferencesByDirection, in a follow-up post.

Here is
[version 2011.0.86.0](zip/bc_11_86.zip)
of The Building Coder samples including the complete source code and Visual Studio solution of the new command.
Actually, to tell the truth, it includes the next yet-to-be-discussed command as well... psst!](http://en.wikipedia.org/wiki/Euclidean_vector)