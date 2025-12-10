---
post_number: "0915"
title: "Revit 2014 and Supporting Columns"
slug: "2014_supporting_columns"
author: "Jeremy Tammik"
tags: ['csharp', 'elements', 'family', 'filtering', 'geometry', 'parameters', 'references', 'revit-api', 'schedules', 'selection', 'transactions', 'views', 'windows']
source_file: "0915_2014_supporting_columns.htm"
original_url: "https://thebuildingcoder.typepad.com/blog/0915_2014_supporting_columns.html"
---

### Revit 2014 and Supporting Columns

#### Revit 2014 Announced

As you probably noticed by now, Revit 2014 has been announced.
Here are the
[main product features](http://www.autodesk.com/products/autodesk-revit-family/features):

- Design
  - Enhanced Autodesk Exchange- Dockable window framework- New stairs and railings- Temporary view templates to change view properties temporarily- Point cloud improvements improve appearance and controls of point clouds- Parameter variance for groups to vary the value of parameters assigned to groups- New air terminal on duct enables placement of air terminal device on duct face- New angle constraints to restrict angles for pipe, duct, and cable tray- Cap open ends of pipe or duct content quickly- CSV file removal project, embedding the data into families- New plumbing template- New rebar placement constraints customization- Reinforcement enhancements with more rebar options- Documentation
    - Non-rectangular crop regions- Split elevations- Alternate dimensions- Improved rebar tagging to annotate multiple elements with a single tag- Improved positioning of beams and braces- Enhanced schedules providing greater control of schedule formatting- Visualization
      - Displaced views enable creation of displaced or exploded building design views- Enhanced visualization- New rendering to reduce project costs with cloud-based rendering- Analysis
        - New building element energy analysis- Enhanced structural analytical model- New duct and pipe calculations to API

Each and every one of these is really exciting in itself, and almost all include or are even based on enhanced API support.
Some of the ones that seem most exciting to me are the non-rectangular crop regions, parameter variance for groups, enhanced schedules, displaced views... well, as said, they are really all very exciting.

And this is not even mentioning some of the new API features, such as the possibility to launch a Revit command and control copy and paste operations.
I will get to all that real soon now.

As Harry puts it, the Revit 2014 API is going to
[blow your faces out](http://boostyourbim.wordpress.com/2013/03/26/revit-2014-api-is-going-to-blow-your-faces-out) :-)

#### Determining the Columns Supporting a Beam

Still, ignoring all that busy-ness for the moment, I chug along and return to a topic similar to my discussion on
[retrieving all touching beams](http://thebuildingcoder.typepad.com/blog/2012/09/filter-for-touching-beams-using-solid-intersection.html).

In that post, I showed how to recursively traverse a collection of beams and detect the neighbouring ones at each end using an ElementIntersectsSolidFilter with a sphere.

Today I explore the task of determining the columns supporting a selected beam using various different filtering and intersection methods.

- [Columns supporting a selected beam](#2)
- [Solid and element intersection](#3)
- [Bounding box filter tolerance](#4)
- [Moving the beam downwards in a temporary transaction](#5)
- [GetGeneratingElementIds](#6)
- [Cylinder Along Location Line Offset Downwards](#7)
- [Find references via ray casting](#8)
- [Sweep Along Location Curve Offset Downwards](#9)
- [Conclusion and source code](#10)

#### Columns Supporting a Selected Beam

Here is a view of a simple sample model:

![Beam supported by two columns](img/supporting_columns.png)

The task at hand is to pick the indicated beam and report the columns supporting it:

![Reporting the columns supporting the selected beam](img/supporting_columns_msg.png)

Simple, ain't it?

#### Solid and Element Intersection

Initially, I made some further attempts and experiments using the ElementIntersectsSolidFilter and ElementIntersectsElementFilter functionality.

The sample provided includes code exercising these tests and also the original bounding box implementation.
It is enclosed in 'if' statements checking the setting of the following Boolean variables:
```csharp
bool useBoundingBox = false;
bool useSolid = false;
bool useElement = false;
```

That enables them to be switched on and off interactively in the debugger, if the need arises.

Unfortunately, as it turns out, they both do not work.

The beam solid is not quite big enough to intersect the columns, because the beam is cut back so that it does not actually intersect them.

The same problem also prevents use the element intersection filter.

It would be great if there was a possibility to grow the solid just slightly, e.g. offset all its faces outwards by an inch or two or define a tolerance before executing the intersection check, like for the bounding box.
Unfortunately, growing or shrinking an arbitrary solid is much harder than a bounding box and therefore not implemented.

#### Bounding Box Filter Tolerance

The BoundingBoxIsInsideFilter can be instantiated with an optional double tolerance value that allows control over the match criteria by using the given tolerance in the geometry comparison.

By default, the tolerance is set to zero.
If the tolerance is positive, the iterated element outline may extend the tolerance distance outside of the given outline in each coordinate to be a match.
If the tolerance is negative, the iterated element outline must lie at least the tolerance distance inside the given outline in each coordinate to be a match.

This is exactly what I would need here for the solid and element intersection filters as well.
Unfortunately, as said, such a tolerance is not supported by the solid or element intersection filters.

#### Moving the Beam Downwards in a Temporary Transaction

Lacking the tolerance option, I thought that maybe it would help to move the beam down a bit, and that it would intersect the supporting columns then.
Unfortunately, that does not help either.

The problem is not only that the bottom face of the beam solid may be located above the end of the supporting columns, but also that the ends of the beam are cut back to avoid intersecting the columns, so their solids do not intersect even when the beam is moved slightly downwards.

This temporary movement forces a switch from read-only to manual transaction mode, of course, even though the model is not actually modify in any way in the end.

To make sure that the movement really is executed and the model updated before checking for an intersection, I followed the advice given by Arnošt Löbel on
[enhancing the temporary transaction trick](http://thebuildingcoder.typepad.com/blog/2012/11/temporary-transaction-trick-touchup.html) and
encapsulated the whole operation in a transaction group.
The group is rolled rolled back at the end, but, before doing so, the encapsulated temporary transaction around the movement of the beam can be committed and the updated geometry evaluated.

By the way, these changes temporarily caused an error saying "Attempted to read or write protected memory. This is often an indication that other memory is corrupt."
This was probably due to accessing the list of elements generated within the temporary transaction after the transaction group is rolled back.
I handled that problem by storing the element ids instead of the live elements themselves inside the transaction, and then opening the elements via their id after rolling back the changes.

As said, the temporary downward translation still did not produce any intersections.

Time for another idea.

#### GetGeneratingElementIds

A colleague suggested that if the beam is cut back by the columns, you can iterate over the beam geometry faces and call the Element GetGeneratingElementIds method on each one, which might return the column element ids.
In this example, you might also get faces generated by joins with other beams.

I tried this, encapsulating the code in the section if( useGeneratingIds ).
Unfortunately, all the generating ids belong to the structural framing or floors categories, so that does not help to determine the columns either.
No columns at all are returned by this method.

#### Cylinder Along Location Line Offset Downwards

I then realised that I could create a much simpler independent solid shape to intersect the columns by extruding a cylinder along the beam location line, offset downwards to just below the bottom face of the beam.
It might require extending a little at each end.
All supporting columns should be intersected by it.

I can use code similar to the sample presented to
[find 3D elements by intersection](http://thebuildingcoder.typepad.com/blog/2013/02/whats-new-in-the-revit-2012-api.html).

Yay!
This works!

#### Find References via Ray Casting

A similar approach as the solid intersection with an offset cylinder could obviously also be implemented using the ray casting functionality provided by the FindReferencesByDirection and FindReferencesWithContextByDirection methods and the ReferenceIntersector wrapper class.

I did not implement any sample code demonstrating this, because I wanted to generalise the straight beam cylinder solution to a more general arbitrary curve case.

#### Sweep Along Location Curve Offset Downwards

Once I had the working solution using a cylinder defined by the straight beam location line offset downwards, I realised that it might be nice to use more generic sweep along curve functionality instead.
After all, the beam location curve might not be straight, and a non-linear location curve can quite simply be used to generate a non-linear solid using the CreateSweptGeometry method instead of CreateExtrusionGeometry.

My first attempt caused the CreateSweptGeometry method to throw an exception saying that "The given attachment point don't lie in the plane of the Curve Loop. Parameter name: pathAttachmentCrvIdx & pathAttachmentParam".
That was my fault, though, because I was providing zero for the parameter value, which is probably the *normalised* curve start point parameter.
When providing the *raw* parameter value returned by curve.get\_EndParameter( 0 ) instead, all works fine.

Here is a rather unrealistic sample spline beam that I used for testing:

![A spline beam supported by multiple columns](img/supporting_columns_spline_beam.png)

The algorithm reports the following supporting columns for the spline beam:

![Reporting the columns supporting the selected spline beam](img/supporting_columns_spline_beam_msg.png)

All is well.

#### Conclusion and Source Code

Here is the complete source code of this command, including the test branches that are disabled by default.
As said, they can be enabled and tested by modifying the Boolean switches interactively in the debugger.
```csharp
[Transaction( TransactionMode.ReadOnly )]
public class Command : IExternalCommand
{
  const double \_eps = 0.1e-9;

  double SignedDistanceTo( Plane plane, XYZ p )
  {
    XYZ v = plane.Normal;
    return v.DotProduct( p )
      - v.DotProduct( plane.Origin );
  }

  class BeamPickFilter : ISelectionFilter
  {
    public bool AllowElement( Element e )
    {
      return null != e.Category
        && e.Category.Id.IntegerValue.Equals(
          (int) BuiltInCategory.OST\_StructuralFraming );
    }

    public bool AllowReference( Reference r, XYZ p )
    {
      return true;
    }
  }

  public Result Execute(
    ExternalCommandData commandData,
    ref string message,
    ElementSet elements )
  {
    UIApplication uiapp = commandData.Application;
    UIDocument uidoc = uiapp.ActiveUIDocument;
    Application app = uiapp.Application;
    Document doc = uidoc.Document;
    Element beam = null;

    try
    {
      Selection sel = uidoc.Selection;

      Reference r = sel.PickObject(
        ObjectType.Element,
        new BeamPickFilter(),
        "Please select a beam" );

      beam = doc.GetElement( r );
    }
    catch( RvtOperationCanceledException )
    {
      return Result.Cancelled;
    }

    List<ElementId> columnIds = null;

    // Optionally switch between different tests
    // by modifying these values in the debugger

    bool useBoundingBox = false;
    bool useSolid = false;
    bool useElement = false;
    bool useGeneratingIds = false;
    bool useLocation = true;

    #region Obsolete previous attempts
    if( useBoundingBox )
    {
      BoundingBoxXYZ box = beam.get\_BoundingBox( null );

      Outline outline = new Outline( box.Min, box.Max );

      ElementFilter bbfilter = new BoundingBoxIntersectsFilter(
        outline, 0.1 );

      FilteredElementCollector columns
        = new FilteredElementCollector( doc )
          .WhereElementIsNotElementType()
          .OfCategory( BuiltInCategory.OST\_StructuralColumns )
          .WherePasses( bbfilter );
    }

    if( useSolid )
    {
      Options opt = app.Create.NewGeometryOptions();
      GeometryElement geo = beam.get\_Geometry( opt );
      Solid solid = null;

      foreach( GeometryObject obj in geo )
      {
        solid = obj as Solid;

        if( null != solid
          && 0 < solid.Faces.Size )
        {
          break;
        }
      }

      ElementFilter beamIntersectFilter
        = new ElementIntersectsSolidFilter( solid );

      FilteredElementCollector columns
        = new FilteredElementCollector( doc )
          .WhereElementIsNotElementType()
          .OfCategory( BuiltInCategory.OST\_StructuralColumns )
          .WherePasses( beamIntersectFilter );
    }

    if( useElement )
    {
      // Initially, no columns are found to
      // intersect the beam. Maybe it will help to
      // move the beam down a bit?

      using( TransactionGroup txg = new TransactionGroup( doc ) )
      {
        txg.Start( "Find Columns Intersecting Beam" );

        using( Transaction tx = new Transaction( doc ) )
        {
          tx.Start( "Temporarily Move Beam Down a Little" );

          ElementTransformUtils.MoveElement(
            doc, beam.Id, -0.1 \* XYZ.BasisZ );

          tx.Commit();
        }

        ElementFilter beamIntersectFilter
          = new ElementIntersectsElementFilter( beam );

        FilteredElementCollector columns
          = new FilteredElementCollector( doc )
            .WhereElementIsNotElementType()
            .OfCategory( BuiltInCategory.OST\_StructuralColumns )
            .WherePasses( beamIntersectFilter );

        columnIds = new List<ElementId>(
          columns.ToElementIds() );

        // We do not commit the transaction group,
        // because no modifications should be saved.
        // The transaction group is only created and
        // started to encapsulate the transactions
        // required by the IsolateElementTemporary
        // method. Since the transaction group is not
        // committed, the changes are automatically
        // discarded.

        //txg.Commit();
      }
    }

    if( useGeneratingIds )
    {
      Options opt = app.Create.NewGeometryOptions();
      GeometryElement geo = beam.get\_Geometry( opt );

      foreach( GeometryObject obj in geo )
      {
        Solid solid = obj as Solid;

        if( null != solid )
        {
          foreach( Face f in solid.Faces )
          {
            ICollection<ElementId> ids
              = beam.GetGeneratingElementIds( f );

            foreach( ElementId id in ids )
            {
              Element e = doc.GetElement( id );
              if( null != e.Category
                && e.Category.Id.IntegerValue.Equals(
                  (int) BuiltInCategory.OST\_StructuralColumns ) )
              {
                columnIds.Add( id );
              }
            }
          }
        }
      }
    }
    #endregion // Obsolete previous attempts

    if( useLocation )
    {
      // Determine beam location curve for
      // extrusion direction and length

      LocationCurve lc = beam.Location as LocationCurve;

      Curve curve = lc.Curve;

      Solid solid = null;

      // Handle generic curve parameters.
      // See below for simplified linear case.

      XYZ p = curve.get\_EndPoint( 0 );
      double param = curve.get\_EndParameter( 0 );
      Transform transform = curve.ComputeDerivatives( param, false );
      Debug.Assert( p.IsAlmostEqualTo( transform.Origin ),
        "expected derivative origin to equal evaluation curve point" );
      XYZ tangent = transform.BasisX;

      // Use bounding box to determine elevation of
      // bottom of beam and how far downwards to
      // offset location line -- one inch below
      // beam bottom.

      BoundingBoxXYZ bb = beam.get\_BoundingBox( null );

      Debug.Assert( .001 > bb.Min.Z - bb.Max.Z,
        "expected horizontal beam" );

      double inch = 1.0 / 12.0;
      double beamBottom = bb.Min.Z;

      XYZ arcCenter = new XYZ( p.X, p.Y,
        beamBottom - inch );

      Plane plane = new Plane( tangent, arcCenter );

      CurveLoop profileLoop = new CurveLoop();

      Autodesk.Revit.Creation.Application creapp
        = app.Create;

      Arc arc1 = creapp.NewArc(
        plane, inch, 0, Math.PI );

      Arc arc2 = creapp.NewArc(
        plane, inch, Math.PI, 2 \* Math.PI );

      profileLoop.Append( arc1 );
      profileLoop.Append( arc2 );

      List<CurveLoop> loops = new List<CurveLoop>( 1 );
      loops.Add( profileLoop );

      // Switch this on to handle a straight beam as
      // a separate simplified case using
      // CreateExtrusionGeometry instead of the
      // generic CreateSweptGeometry solution.

      bool checkForLine = false;

      if( checkForLine && curve is Line )
      {
        XYZ q = curve.get\_EndPoint( 1 );
        XYZ v = q - p;

        Debug.Assert( 0.01 > v.Z,
          "expected horizontal beam" );

        Debug.Assert( v.IsAlmostEqualTo( tangent ),
          "expected straight beam vector to equal start tangent" );

        solid = GeometryCreationUtilities
          .CreateExtrusionGeometry( loops, v, v.GetLength() );
      }
      else
      {
        // Offset location curve downward
        // one inch  below beam bottom face

        XYZ offset = arcCenter - p;

        transform = Transform.get\_Translation(
          offset );

        CurveLoop sweepPath = new CurveLoop();

        sweepPath.Append( curve.get\_Transformed(
          transform ) );

        solid = GeometryCreationUtilities
          .CreateSweptGeometry(
            sweepPath, 0, param, loops );
      }

      ElementFilter beamIntersectFilter
        = new ElementIntersectsSolidFilter( solid );

      columnIds = new List<ElementId>(
        new FilteredElementCollector( doc )
          .WhereElementIsNotElementType()
          .OfCategory( BuiltInCategory.OST\_StructuralColumns )
          .WherePasses( beamIntersectFilter )
          .ToElementIds() );
    }

    int n = (null == columnIds)
      ? 0
      : columnIds.Count<ElementId>();

    string s1 = string.Format(
      "Selected beam is supported by {0} column{1}{2}",
      n,
      ( 1 == n ? "" : "s" ),
      ( 0 == n ? "." : ":" ) );

    string s2 = "<None>";

    if( 0 < n )
    {
      uidoc.Selection.Elements.Clear();

      foreach( ElementId id in columnIds )
      {
        Element e = doc.GetElement( id );
        uidoc.Selection.Elements.Add( e );
      }

      s2 = string.Join( ", ",
        columnIds.ConvertAll<string>(
          id => id.IntegerValue.ToString() ) );
    }

    TaskDialog.Show( s1, s2 );

    return Result.Succeeded;
  }
}
```

For your convenience, here is
[GetBeamColumns06.zip](zip/GetBeamColumns06.zip) containing
the complete source code, Visual Studio solution and add-in manifest of the GetBeamColumns external command.

I hope you find this both interesting and useful as a basis for your own variants.