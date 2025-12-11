---
ai_optimized: true
author: Jeremy Tammik
complexity_score: 2.8
content_type: code_example
optimization_date: '2025-12-11T11:44:14.707086'
original_url: https://thebuildingcoder.typepad.com/blog/0839_slab_boundary.html
post_number: 0839
reading_time_minutes: 5
series: general
slug: slab_boundary
source_file: 0839_slab_boundary.htm
tags:
- elements
- geometry
- python
- references
- revit-api
- selection
- transactions
- walls
- windows
title: Slab Boundary Revisited
word_count: 992
---

### Slab Boundary Revisited

After
[yesterday's rejuvenation](http://thebuildingcoder.typepad.com/blog/2012/10/wall-footing-relationship-revisited.html) of
the old wall footing
[host reference relationship detection](http://thebuildingcoder.typepad.com/blog/2009/06/host-reference.html),
today raises another old question prompting me to update and retest The Building Coder samples yet again:

**Question:** How can I obtain the boundary of a floor slab using the Revit API, please?

**Answer:** I implemented a CmdSlabBoundary external command to
[determine the slab boundary](http://thebuildingcoder.typepad.com/blog/2008/10/slab-boundary.html) back
in the dawn of time, in 2008, in one of the first posts on this blog.

It determines the boundary edges of a floor slab, including holes, and creates a set of model curves along the bottom edges of the slab to highlight them.

Since then it has just been flat ported every year from one version to the next.

To ensure that it still works, I updated it slightly and tested in Revit 2013.
All I did this time around was to change the transaction mode from automatic to manual, since
[automatic transaction mode is considered obsolete](http://thebuildingcoder.typepad.com/blog/2012/05/read-only-and-automatic-transaction-modes.html) nowadays.

Here is the updated implementation:
```python
[Transaction( TransactionMode.Manual )]
class CmdSlabBoundary : IExternalCommand
{
  /// <summary>
  /// Offset the generated boundary polygon loop
  /// model lines downwards to separate them from
  /// the slab edge.
  /// </summary>
  const double \_offset = 0.1;

  /// <summary>
  /// Determine the boundary polygons of the lowest
  /// horizontal planar face of the given solid.
  /// </summary>
  /// <param name="polygons">Return polygonal boundary
  /// loops of lowest horizontal face, i.e. profile of
  /// circumference and holes</param>
  /// <param name="solid">Input solid</param>
  /// <returns>False if no horizontal planar face was
  /// found, else true</returns>
  static bool GetBoundary(
    List<List<XYZ>> polygons,
    Solid solid )
  {
    PlanarFace lowest = null;
    FaceArray faces = solid.Faces;
    foreach( Face f in faces )
    {
      PlanarFace pf = f as PlanarFace;
      if( null != pf && Util.IsHorizontal( pf ) )
      {
        if( ( null == lowest )
          || ( pf.Origin.Z < lowest.Origin.Z ) )
        {
          lowest = pf;
        }
      }
    }
    if( null != lowest )
    {
      XYZ p, q = XYZ.Zero;
      bool first;
      int i, n;
      EdgeArrayArray loops = lowest.EdgeLoops;
      foreach( EdgeArray loop in loops )
      {
        List<XYZ> vertices = new List<XYZ>();
        first = true;
        foreach( Edge e in loop )
        {
          IList<XYZ> points = e.Tessellate();
          p = points[0];
          if( !first )
          {
            Debug.Assert( p.IsAlmostEqualTo( q ),
              "expected subsequent start point"
              + " to equal previous end point" );
          }
          n = points.Count;
          q = points[n - 1];
          for( i = 0; i < n - 1; ++i )
          {
            XYZ v = points[i];
            v -= \_offset \* XYZ.BasisZ;
            vertices.Add( v );
          }
        }
        q -= \_offset \* XYZ.BasisZ;
        Debug.Assert( q.IsAlmostEqualTo( vertices[0] ),
          "expected last end point to equal"
          + " first start point" );
        polygons.Add( vertices );
      }
    }
    return null != lowest;
  }

  /// <summary>
  /// Return all floor slab boundary loop polygons
  /// for the given floors, offset downwards from the
  /// bottom floor faces by a certain amount.
  /// </summary>
  static public List<List<XYZ>> GetFloorBoundaryPolygons(
    List<Element> floors,
    Options opt )
  {
    List<List<XYZ>> polygons = new List<List<XYZ>>();

    foreach( Floor floor in floors )
    {
      GeometryElement geo = floor.get\_Geometry( opt );

      //GeometryObjectArray objects = geo.Objects; // 2012
      //foreach( GeometryObject obj in objects ) // 2012

      foreach( GeometryObject obj in geo ) // 2013
      {
        Solid solid = obj as Solid;
        if( solid != null )
        {
          GetBoundary( polygons, solid );
        }
      }
    }
    return polygons;
  }

  public Result Execute(
    ExternalCommandData commandData,
    ref string message,
    ElementSet elements )
  {
    UIApplication app = commandData.Application;
    UIDocument uidoc = app.ActiveUIDocument;
    Document doc = uidoc.Document;

    // retrieve selected floors, or all floors, if nothing is selected:

    List<Element> floors = new List<Element>();

    if( !Util.GetSelectedElementsOrAll(
      floors, uidoc, typeof( Floor ) ) )
    {
      Selection sel = uidoc.Selection;

      message = ( 0 < sel.Elements.Size )
        ? "Please select some floor elements."
        : "No floor elements found.";

      return Result.Failed;
    }

    Options opt = app.Application.Create.NewGeometryOptions();

    List<List<XYZ>> polygons
      = GetFloorBoundaryPolygons( floors, opt );

    int n = polygons.Count;

    Debug.Print(
      "{0} boundary loop{1} found.",
      n, Util.PluralSuffix( n ) );

    Creator creator = new Creator( doc );

    using( Transaction t = new Transaction( doc ) )
    {
      t.Start( "Draw Slab Boundaries" );

      creator.DrawPolygons( polygons );

      t.Commit();
    }

    return Result.Succeeded;
  }
}
```

I ran this command on a minimal sample model containing just one floor slab with a couple of holes in it.
That produces the following result, in which I modified the model lines graphics display to distinguish them better – don't know why the model ellipse is not picking up the dotted line as well as the straight segments – maybe the tessellation is chopping it up too finely:

![Floor slab boundaries](img/floor_slab_boundaries.png)

Here is
[version 2013.0.99.4](zip/bc_13_99_4.zip) of
The Building Coder samples including the updated CmdSlabBoundary external command.

#### Programmatically Invoke Snoop Objects Dialogue

Yesterday I discovered an pretty cool post by Daren Thomas, the father of
[RevitPythonShell](http://code.google.com/p/revitpythonshell),
describing how you can
[programmatically invoke the RevitLookup Snoop Objects dialogue](http://darenatwork.blogspot.de/2011/08/using-snoop-objects-dialog-from.html).

His code is in Python, though the concepts are equally valid in and port perfectly to any other .NET language as well.

Definitely worth taking a look at!

#### HTCPCP – Hyper Text Coffee Pot Control Protocol

Another little item that you absolutely must be aware of in the cloud and mobile arena is the
[Hyper Text Coffee Pot Control Protocol standard HTCPCP](http://tools.ietf.org/html/rfc2324).
It is worthwhile noting the date of publication, also :-)

#### Closing Revit by Posting WM\_CLOSE to its Main Window Handle

Before closing this post, one last Revit API related note on closing Revit itself...

Shutting down and exiting the Revit session is not supported by the official Revit API.

I discussed
[closing the active document](http://thebuildingcoder.typepad.com/blog/2010/10/closing-the-active-document-and-why-not-to.html) and
why not to do so.
The same caveats obviosuly apply to shutting down the Revit session itself.

Still, Augusto Goncalves now presents a neat little solution to obtain the Revit main window handle and
[close down Revit](http://adndevblog.typepad.com/aec/2012/10/close-revit-from-the-api.html) by
using SendMessage to post a WM\_CLOSE request to it.