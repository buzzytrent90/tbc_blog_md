---
post_number: "0540"
title: "Dimension Walls using FindReferencesByDirection"
slug: "dim_walls_find_refs"
author: "Jeremy Tammik"
tags: ['csharp', 'elements', 'filtering', 'geometry', 'levels', 'parameters', 'python', 'references', 'revit-api', 'selection', 'transactions', 'views', 'walls']
source_file: "0540_dim_walls_find_refs.htm"
original_url: "https://thebuildingcoder.typepad.com/blog/0540_dim_walls_find_refs.html"
---

### Dimension Walls using FindReferencesByDirection

On Monday I presented a new Building Coder sample command
CmdDimensionWallsIterateFaces
demonstrating how to create dimensioning between two opposing wall faces by explicitly iterating over their solid geometry and its faces, asking the Revit geometry library to
[compute stable references](http://thebuildingcoder.typepad.com/blog/2010/01/geometry-options.html) to
attach the dimensioning elements to.
The explicit iteration over geometry, searching for the right faces, and extraction of their references can all be simplified by making use of the higher-level specialised
[FindReferencesByDirection](http://thebuildingcoder.typepad.com/blog/2010/01/findreferencesbydirection.html) method
provided for this purpose by the Revit API.

I continued working on the topic and implemented the alternative approach using FindReferencesByDirection as well in a second Building Coder sample external command implementation named CmdDimensionWallsFindRefs.

It removes the following limitations of the previous sample:

- The dimension line was defined from wall midpoint to wall midpoint: In the new solution, there is no limitation on the location of the dimension line relative to the walls. The user selects the location of the dimension line somewhere on the second wall. The two walls must be parallel, and the projection of the first wall onto the second wall's location line must intersect with the point picked for the dimension line location.- The array of references required for the creation of the dimension element is created directly from the references returned by the high-level FindReferencesByDirection method. It shoots a ray through the Revit model and reports the elements and their faces and the points on the faces intersected by the ray. We pick out all references which refer to surfaces on the two selected walls and use those directly to create the dimension element.

The resulting command is more robust and foolproof than the first solution.

The new sample also demonstrates use of the PickObject method to select the walls in conjunction with a selection filter implementation of the ISelectionFilter interface. It also demonstrates that this method returns a pick point which can be evaluated to ask the user to select the dimension line location at the same time as selecting the second wall.

Here is the selection filter implementation that only allows the user to pick a point on a wall:
```python
class WallSelectionFilter : ISelectionFilter
{
  public bool AllowElement( Element e )
  {
    return e is Wall;
  }

  public bool AllowReference( Reference r, XYZ p )
  {
    return true;
  }
}
```

The call to FindReferencesByDirection requires a 3D view to be provided.
We assume that a suitable 3D view exists in the document and implement the following simple helper method making a call to a filtered element collector to retrieve the first 3D view encountered:
```csharp
private View3D Get3DView( Document doc )
{
  FilteredElementCollector collector
    = new FilteredElementCollector( doc );

  collector.OfClass( typeof( View3D ) );

  foreach( View3D v in collector )
  {
    // skip view templates here because they
    // are invisible in project browsers:

    if( v != null && !v.IsTemplate && v.Name == "{3D}" )
    {
      return v;
    }
  }
  return null;
}
```

With these two methods in place and making use of the CreateDimensionElement helper method used in the previous CmdDimensionWallsIterateFaces command, the new external command implementation looks like this:
```python
/// <summary>
/// Dimension two opposing parallel walls.
/// Prompt user to select the first wall, and
/// the second at the point at which to create
/// the dimensioning. Use FindReferencesByDirection
/// to determine the wall face references.
/// </summary>
[Transaction( TransactionMode.Manual )]
[Regeneration( RegenerationOption.Manual )]
class CmdDimensionWallsFindRefs : IExternalCommand
{
  const string \_prompt
    = "Please select two parallel straight walls"
      + " with a partial projected overlap.";

  public Result Execute(
    ExternalCommandData commandData,
    ref string message,
    ElementSet elements )
  {
    UIApplication uiapp = commandData.Application;
    UIDocument uidoc = uiapp.ActiveUIDocument;
    Application app = uiapp.Application;
    Document doc = uidoc.Document;

    // select two walls and the dimension line point:

    Selection sel = uidoc.Selection;
    ReferenceArray refs = new ReferenceArray();

    try
    {
      WallSelectionFilter f
        = new WallSelectionFilter();

      refs.Append( sel.PickObject(
        ObjectType.Element, f,
        "Please select first wall" ) );

      refs.Append( sel.PickObject(
        ObjectType.Element, f,
        "Please pick dimension line "
        + "point on second wall" ) );
    }
    catch( OperationCanceledException )
    {
      message = "No two walls selected";
      return Result.Failed;
    }

    // ensure the two selected walls are straight and
    // parallel; determine their mutual normal vector
    // and a point on each wall for distance
    // calculations:

    Wall[] walls = new Wall[2];
    List<int> ids = new List<int>( 2 );
    XYZ[] pts = new XYZ[2];
    Line[] lines = new Line[2];
    IntersectionResult ir;
    XYZ normal = null;
    int i = 0;

    foreach( Reference r in refs )
    {
      Wall wall = r.Element as Wall;
      walls[i] = wall;
      ids.Add( wall.Id.IntegerValue );

      // obtain location curve and
      // check that it is straight:

      LocationCurve lc = wall.Location
        as LocationCurve;

      Curve curve = lc.Curve;
      lines[i] = curve as Line;

      if( null == lines[i] )
      {
        message = \_prompt;
        return Result.Failed;
      }

      // obtain normal vectors
      // and ensure that they are equal,
      // i.e. walls are parallel:

      if( null == normal )
      {
        normal = Util.Normal( lines[i] );
      }
      else
      {
        if( !Util.IsParallel( normal,
          Util.Normal( lines[i] ) ) )
        {
          message = \_prompt;
          return Result.Failed;
        }
      }

      // obtain pick points and project
      // onto wall location lines:

      XYZ p = r.GlobalPoint;
      ir = lines[i].Project( p );

      if( null == ir )
      {
        message = string.Format(
          "Unable to project pick point {0} "
          + "onto wall location line.",
          i );

        return Result.Failed;
      }

      pts[i] = ir.XYZPoint;

      Debug.Print(
        "Wall {0} id {1} at {2}, {3} --> point {4}",
        i, wall.Id.IntegerValue,
        Util.PointString( lines[i].get\_EndPoint( 0 ) ),
        Util.PointString( lines[i].get\_EndPoint( 1 ) ),
        Util.PointString( pts[i] ) );

      if( 0 < i )
      {
        // project dimension point selected on second wall
        // back onto first wall, and ensure that normal
        // points from second wall to first:

        ir = lines[0].Project( pts[1] );
        if( null == ir )
        {
          message = string.Format(
            "Unable to project selected dimension "
            + "line point {0} on second wall onto "
            + "first wall's location line.",
            Util.PointString( pts[1] ) );

          return Result.Failed;
        }
        pts[0] = ir.XYZPoint;
      }

      ++i;
    }

    XYZ v = pts[0] - pts[1];
    if( 0 > v.DotProduct( normal ) )
    {
      normal = -normal;
    }

    // invoke FindReferencesByDirection, shooting ray
    // back from second picked wall towards first:

    View3D view = Get3DView( doc );

    refs = doc.FindReferencesByDirection(
      pts[1], normal, view );

    Debug.Print( "Shooting ray from {0} direction "
      + "{1} returns {2} references",
      Util.PointString( pts[1] ),
      Util.PointString( normal ),
      refs.Size );

    // store the references to the wall surfaces:

    Reference[] surfrefs = new Reference[2] {
      null, null };

    // find the two closest intersection
    // points on each of the two walls:

    double[] minDistance = new double[2] {
      double.MaxValue,
      double.MaxValue };

    foreach( Reference r in refs )
    {
      Element e = r.Element;

      if( e is Wall )
      {
        i = ids.IndexOf( e.Id.IntegerValue );

        if( -1 < i
          && ElementReferenceType.REFERENCE\_TYPE\_SURFACE
            == r.ElementReferenceType )
        {
          GeometryObject g = r.GeometryObject;

          if( g is PlanarFace )
          {
            PlanarFace face = g as PlanarFace;

            Line line = ( e.Location as LocationCurve )
              .Curve as Line;

            Debug.Print(
              "Wall {0} at {1}, {2} surface {3} "
              + "normal {4} proximity {5}",
              e.Id.IntegerValue,
              Util.PointString( line.get\_EndPoint( 0 ) ),
              Util.PointString( line.get\_EndPoint( 1 ) ),
              Util.PointString( face.Origin ),
              Util.PointString( face.Normal ),
              r.ProximityParameter );

            // first reference: assert it is a face on this wall
            // and the distance is half the wall thickness
            //
            // second reference: the first reference on the other
            // wall; assert the distance between the two references
            // equals the distance between the wall location lines
            // minus half of the sum of the two wall thicknesses.

            if( r.ProximityParameter < minDistance[i] )
            {
              surfrefs[i] = r;
              minDistance[i] = r.ProximityParameter;
            }
          }
        }
      }
    }

    if( null == surfrefs[0] )
    {
      message = "No suitable face intersection "
        + "points found on first wall.";

      return Result.Failed;
    }

    if( null == surfrefs[1] )
    {
      message = "No suitable face intersection "
        + "points found on second wall.";

      return Result.Failed;
    }

    CmdDimensionWallsIterateFaces
      .CreateDimensionElement( doc.ActiveView,
      pts[0], surfrefs[0], pts[1], surfrefs[1] );

    return Result.Succeeded;
  }
}
```

Here is a slightly less horrible example than the previous one of a horizontal and a vertical dimensioning element created by this command:
![Dimensioning two opposing walls using FindReferencesByDirection](img/DimensionTwoWallsFindReferencesByDirection.png)

Here is
[version 2011.0.86.0](zip/bc_11_86.zip)
of The Building Coder samples including the complete source code and Visual Studio solution with both of the two new commands.