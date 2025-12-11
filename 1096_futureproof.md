---
ai_optimized: true
author: Jeremy Tammik
complexity_score: 4.2
content_type: code_example
optimization_date: '2025-12-11T11:44:15.296869'
original_url: https://thebuildingcoder.typepad.com/blog/1096_futureproof.html
post_number: '1096'
reading_time_minutes: 5
series: general
slug: futureproof
source_file: 1096_futureproof.htm
tags:
- elements
- family
- filtering
- python
- references
- revit-api
- selection
- sheets
- transactions
- views
- walls
title: Future Proofing The Building Coder Samples
word_count: 1094
---

### Future Proofing The Building Coder Samples

We've reached the add-in future-proofing season again, the time of year to eliminate compiler warnings and deprecated calls for easier migration to an updated API, just like last year in
[January](http://thebuildingcoder.typepad.com/blog/2012/01/eliminating-compiler-warnings-and-deprecated-calls.html) and
[February](http://thebuildingcoder.typepad.com/blog/2013/02/eliminating-compiler-warnings-and-deprecated-calls.html).

It is very easy this time around, since we only have
[two warnings](zip/bc_migr_2014_j.txt) left,
about use of the obsolete TitleBlocks property and FindReferencesWithContextByDirection method:

- CmdSheetSize.cs: 'Autodesk.Revit.DB.Document.TitleBlocks' is obsolete: 'This method is obsolete in Revit 2014. Use FilteredElementCollector with a FamilySymbol class filter and an appropriate category filter instead.'
- CmdDimensionWallsFindRefs.cs: 'Autodesk.Revit.DB.Document.FindReferencesWithContextByDirection(Autodesk.Revit.DB.XYZ, Autodesk.Revit.DB.XYZ, Autodesk.Revit.DB.View3D)' is obsolete: 'This method is deprecated in Revit 2014. Use the ReferenceIntersector class instead.'

The former warning is caused by code explicitly left in for comparison with the new suggested workaround listed in the warning message, so we can leave it there as long as it still compiles, until the very last minute.

I fixed the latter by updating the CmdDimensionWallsFindRefs external command implementation, simply following the suggestion to replace the obsolete FindReferencesWithContextByDirection by the simplified interface provided by the ReferenceIntersector wrapper class.

The original CmdDimensionWallsFindRefs implementation showed how to
[dimension walls using FindReferencesByDirection](http://thebuildingcoder.typepad.com/blog/2011/02/dimension-walls-using-findreferencesbydirection.html).

I later updated it, replacing the FindReferencesByDirection method by FindReferencesWithContextByDirection when
[migrating to the Revit 2012 API](http://thebuildingcoder.typepad.com/blog/2011/04/migrating-the-building-coder-samples-to-revit-2012.html).

This newest update makes it much shorter and simpler, since the ReferenceIntersector.FindNearest method enables us to eliminate all the code previously needed to determine the closest reference ourselves.

Here is the new code:

```python
/// <summary>
/// Dimension two opposing parallel walls.
/// Prompt user to select the first wall, and
/// the second at the point at which to create
/// the dimensioning. Use FindReferencesByDirection
/// to determine the wall face references.
///
/// Second sample solution for case 1263303 [Case
/// Number 1263071 [Revit 2011 Dimension Wall]].
/// </summary>
[Transaction( TransactionMode.Manual )]
class CmdDimensionWallsFindRefs : IExternalCommand
{
  const string \_prompt
    = "Please select two parallel straight walls"
      + " with a partial projected overlap.";

  #region WallSelectionFilter
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
  #endregion // WallSelectionFilter

  #region Get3DView
  /// <summary>
  /// Return a 3D view from the given document.
  /// </summary>
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
  #endregion // Get3DView

  public Result Execute(
    ExternalCommandData commandData,
    ref string message,
    ElementSet elements )
  {
    UIApplication uiapp = commandData.Application;
    UIDocument uidoc = uiapp.ActiveUIDocument;
    Application app = uiapp.Application;
    Document doc = uidoc.Document;

    // Select two walls and the dimension line point:

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

    // Ensure the two selected walls are straight and
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
      // 'Autodesk.Revit.DB.Reference.Element' is
      // obsolete: Property will be removed. Use
      // Document.GetElement(Reference) instead.
      //Wall wall = r.Element as Wall; // 2011

      Wall wall = doc.GetElement( r ) as Wall; // 2012

      walls[i] = wall;
      ids.Add( wall.Id.IntegerValue );

      // Obtain location curve and
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

      // Obtain normal vectors
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

      // Obtain pick points and project
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
        Util.PointString( lines[i].GetEndPoint( 0 ) ),
        Util.PointString( lines[i].GetEndPoint( 1 ) ),
        Util.PointString( pts[i] ) );

      if( 0 < i )
      {
        // Project dimension point selected on second wall
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

    // Shoot a ray back from the second
    // picked wall towards first:

    Debug.Print(
      "Shooting ray from {0} in direction {1}",
      Util.PointString( pts[1] ),
      Util.PointString( normal ) );

    View3D view = Get3DView( doc );

    if( null == view )
    {
      message = "No 3D view named '{3D}' found; "
        + "run the View > 3D View command once "
        + "to generate it.";

      return Result.Failed;
    }

    //refs = doc.FindReferencesByDirection(
    //  pts[1], normal, view ); // 2011

    //IList<ReferenceWithContext> refs2
    //  = doc.FindReferencesWithContextByDirection(
    //    pts[1], normal, view ); // 2012

    // In the Revit 2014 API, the call to
    // FindReferencesWithContextByDirection causes a
    // warning saying:
    // "FindReferencesWithContextByDirection is obsolete:
    // This method is deprecated in Revit 2014.
    // Use the ReferenceIntersector class instead."

    ReferenceIntersector ri
      = new ReferenceIntersector(
        walls[0].Id, FindReferenceTarget.Element, view );

    ReferenceWithContext ref2
      = ri.FindNearest( pts[1], normal );

    if( null == ref2 )
    {
      message = "ReferenceIntersector.FindNearest"
        + " returned null!";

      return Result.Failed;
    }

    CmdDimensionWallsIterateFaces
      .CreateDimensionElement( doc.ActiveView,
      pts[0], ref2.GetReference(), pts[1], refs.get\_Item( 1 ) );

    return Result.Succeeded;
  }
}
```

I committed those changes to The Building Coder samples
[release 2014.0.106.7](https://github.com/jeremytammik/the_building_coder_samples/releases/tag/2014.0.106.7).

In a second step, I also incremented the copyright year from 2013 to 2014, affecting all the modules, and saved that as
[release 2014.0.106.8](https://github.com/jeremytammik/the_building_coder_samples/releases/tag/2014.0.106.8).

Normally, you would simply grab
[The Building Coder samples](https://github.com/jeremytammik/the_building_coder_samples) current
master branch, of course.