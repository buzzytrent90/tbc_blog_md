---
post_number: "1241"
title: "Picking Pairs and Dimensioning Family Instance Origin"
slug: "dim_fam_inst_orig"
author: "Jeremy Tammik"
tags: ['csharp', 'elements', 'family', 'filtering', 'geometry', 'python', 'references', 'revit-api', 'selection', 'transactions', 'views', 'walls']
source_file: "1241_dim_fam_inst_orig.htm"
original_url: "https://thebuildingcoder.typepad.com/blog/1241_dim_fam_inst_orig.html"
---

### Picking Pairs and Dimensioning Family Instance Origin

We looked at various aspects of
[creating dimensioning](http://thebuildingcoder.typepad.com/blog/about-the-author.html#5.45) in
the past.
There is still one glaring omission, though: how to dimension to a family instance origin point.

Let's take a look at that, after quickly mentioning the official announcement of the movie matching game that we created at the recent Berlin hackathon.

Actually, coincidentally, I discuss two topics today that are both completely and utterly dedicated to picking pairs: the Movie Match MasterY game, matching pairs of cards representing films, and my new pair picker utility class, for selecting pairs of Revit elements of a given type:

- [Movie Match MasterY](#2)
- [Creating dimensioning referencing family instance origin](#3)
- [JtPairPicker pair picker utility class](#4)
- [Download](#5)

#### Movie Match MasterY

As reported during the
[Berlin hackathon](http://thebuildingcoder.typepad.com/blog/2014/10/berlin-hackathon-begin.html), its
[progress](http://thebuildingcoder.typepad.com/blog/2014/10/attached-detail-groups-and-inverse-relationships.html) and
[results](http://thebuildingcoder.typepad.com/blog/2014/10/berlin-hackathon-results-3d-viewer-and-web-news.html),
I was part of the
[m3my team](https://github.com/m3my) creating the
[Movie Match MasterY](https://m3my.github.io) movie matching memory game.

We were forced to rename it from the original *MovieMemory*, because the German company Ravensburger has a trademark on 'Memory' in numerous countries, causing Apple to
[ban all games whose name contains 'Memory' from its AppStore](https://www.google.de/webhp?sourceid=chrome-instant&ion=1&espv=2&ie=UTF-8#q=memory%20ravensburger%20trademark).

Anyway, here is now the (German)
[Movie Match MasterY announcement](http://blog.neofonie.de/2014/11/12/wie-mit-hilfe-von-textanalysen-und-natural-language-processing-ein-moviegame-entstand/) on the
[Neofonie](http://www.neofonie.de) tech blog.

#### Creating Dimensioning Referencing Family Instance Origin

Back to the Revit API and the main topic for today:

**Question:** I know how to create dimensioning between two family edges.

How can I create dimensioning between the centre point or origin point of the family, however?

Here is an example showing the two situations with manually generated dimensioning:

![Dimensioning family instance edge versus origin](img/dimension_instance_edge_or_origin_before.png)

**Answer:** The secret lies in accessing the hidden elements embedded within the family instance and definition.

In your case, you can use the non-visible family instance geometry origin point.

Turn on both ComputeReferences and IncludeNonVisibleObjects when you request the geometry from the family instances:

```csharp
  \_opt = new Options();
  \_opt.ComputeReferences = true;
  \_opt.IncludeNonVisibleObjects = true;
```

The geometry of the hidden elements includes a point from which you can retrieve a reference to use to define your dimensioning.

The current version of RevitLookup enables you to query and explore non-visible objects in the element geometry:

![RevitLookup accessing non-visible geometry](img/snoop_non_visible_objects.png)

Navigate into the geometry element collection of geometry objects to explore the point object we are looking for:

![RevitLookup displaying family instance geometry element point](img/snoop_non_visible_geometry_element_point.png)

As always, the most up-to-date version of RevitLookup is available from the
[RevitLookup GitHub repository](https://github.com/jeremytammik/RevitLookup).

Based on that, I implemented the following rather compact helper method retrieving the non-visible family instance element geometry, extracting the Point object and returning a reference to it:

```csharp
  /// <summary>
  /// Retrieve the given family instance's
  /// non-visible geometry point reference.
  /// </summary>
  static Reference GetFamilyInstancePointReference(
    FamilyInstance fi )
  {
    return fi.get\_Geometry( \_opt )
      .OfType<Point>()
      .Select<Point, Reference>( x => x.Reference )
      .FirstOrDefault();
  }
```

I implemented a new external command CmdDimensionInstanceOrigin in The Building Coder samples to make use of that to pick the two family instances in your sample model and create dimensioning between them like this:

```python
[Transaction( TransactionMode.Manual )]
class CmdDimensionInstanceOrigin : IExternalCommand
{
  static Options \_opt = null;

  public Result Execute(
    ExternalCommandData commandData,
    ref string message,
    ElementSet elements )
  {
    UIApplication app = commandData.Application;
    UIDocument uidoc = app.ActiveUIDocument;
    Document doc = uidoc.Document;

    JtPairPicker<FamilyInstance> picker
      = new JtPairPicker<FamilyInstance>( uidoc );

    Result rc = picker.Pick();

    if( Result.Failed == rc )
    {
      message = "We need at least two "
        + "FamilyInstance elements in the model.";
    }
    else if( Result.Succeeded == rc )
    {
      IList<FamilyInstance> a = picker.Selected;

      \_opt = new Options();
      \_opt.ComputeReferences = true;
      \_opt.IncludeNonVisibleObjects = true;

      XYZ[] pts = new XYZ[2];
      Reference[] refs = new Reference[2];

      pts[0] = ( a[0].Location as LocationPoint ).Point;
      pts[1] = ( a[1].Location as LocationPoint ).Point;

      refs[0] = GetFamilyInstancePointReference( a[0] );
      refs[1] = GetFamilyInstancePointReference( a[1] );

      CmdDimensionWallsIterateFaces
        .CreateDimensionElement( doc.ActiveView,
        pts[0], refs[0], pts[1], refs[1] );
    }
    return rc;
  }
}
```

Besides accessing the required point and reference, it makes use of my new [JtPairPicker class described below](#3) to select the two family instances.

Here is the result of running the new command in your sample model:

![Programmatically generated dimensioning to family instance origin](img/dimension_instance_edge_or_origin_after.png)

The dimensioning element at the bottom right is the programmatically generated one.

#### JtPairPicker Pair Picker Utility Class

The external command implementation above is very succinct.

To a large extent, this is due to my new JtPairPicker pair picker utility class.

It can be used whenever you wish to automatically retrieve or interactively select two elements of the same type.

For a maximum of flexibility, comfort and efficiency in testing the command, it supports three different possibilities for selecting the two elements:

- Run in a model containing exactly two parallel offset pipe elements, they will be automatically selected.
- If no two elements of the required type exist in the model, the command is aborted.
- If more than two such elements exist, we check whether some have been pre-selected before launching the command. If so, the first two are used.
- Otherwise, the user is prompted to interactively (post-) select two elements of the required type.

The implementation is based on my code selecting two pipes for creating
[rolling offset pipe elbow fittings](http://thebuildingcoder.typepad.com/blog/2014/01/explicitly-placing-rolling-offset-pipe-elbow-fittings.html),
in which the
[pipe selection implementation](http://thebuildingcoder.typepad.com/blog/2014/01/calculating-a-rolling-offset-between-two-pipes.html#2) constitutes
the major part of the code.

Extracting that into a separate utility class saves a lot of space!

I therefore converted this to a generic templated class and used it in the dimensioning example above to retrieve or pick the two family instances.

Here is the full class implementation:

```python
/// <summary>
/// Pick a pair of elements of a specific type.
/// If exactly two exist in the entire model,
/// take them. If there are less than two, give
/// up. If elements have been preselected, use
/// those. Otherwise, prompt for interactive
/// picking.
/// </summary>
class JtPairPicker<T> where T : Element
{
  UIDocument \_uidoc;
  Document \_doc;
  List<T> \_a;

  /// <summary>
  /// Allow selection of elements of type T only.
  /// </summary>
  class ElementsOfClassSelectionFilter<T2> : ISelectionFilter
  {
    public bool AllowElement( Element e )
    {
      return e is T2;
    }

    public bool AllowReference( Reference r, XYZ p )
    {
      return true;
    }
  }

  public JtPairPicker( UIDocument uidoc )
  {
    \_uidoc = uidoc;
    \_doc = \_uidoc.Document;
  }

  /// <summary>
  /// Return selection result.
  /// </summary>
  public IList<T> Selected
  {
    get
    {
      return \_a;
    }
  }

  /// <summary>
  /// Run the automatic or interactive
  /// selection process.
  /// </summary>
  public Result Pick()
  {
    // Retrieve all T elements in the entire model.

    \_a = new List<T>(
      new FilteredElementCollector( \_doc )
        .OfClass( typeof( T ) )
        .ToElements()
        .Cast<T>() );

    int n = \_a.Count;

    // If there are less than two,
    // there is nothing we can do.

    if( 2 > n )
    {
      return Result.Failed;
    }

    // If there are exactly two, pick those.

    if( 2 == n )
    {
      return Result.Succeeded;
    }

    // There are more than two to choose from.
    // Check for a pre-selection.

    \_a.Clear();

    Selection sel = \_uidoc.Selection;

    ICollection<ElementId> ids
      = sel.GetElementIds();

    n = ids.Count;

    Debug.Print( "{0} pre-selected elements.", n );

    // If two or more T elements were pre-
    // selected, use the first two encountered.

    if( 1 < n )
    {
      foreach( ElementId id in ids )
      {
        T e = \_doc.GetElement( id ) as T;

        Debug.Assert( null != e,
          "only elements of type T can be picked" );

        \_a.Add( e );

        if( 2 == \_a.Count )
        {
          Debug.Print( "Found two pre-selected "
            + "elements of desired type and "
            + "ignoring everything else." );

          break;
        }
      }
    }

    // None or less than two elements were pre-
    // selected, so prompt for an interactive
    // post-selection instead.

    if( 2 != \_a.Count )
    {
      \_a.Clear();

      // Select first element.

      try
      {
        Reference r = sel.PickObject(
          ObjectType.Element,
          new ElementsOfClassSelectionFilter<T>(),
          "Please pick first element." );

        \_a.Add( \_doc.GetElement( r.ElementId )
          as T );
      }
      catch( Autodesk.Revit.Exceptions
        .OperationCanceledException )
      {
        return Result.Cancelled;
      }

      // Select second element.

      try
      {
        Reference r = sel.PickObject(
          ObjectType.Element,
          new ElementsOfClassSelectionFilter<T>(),
          "Please pick second element." );

        \_a.Add( \_doc.GetElement( r.ElementId )
          as T );
      }
      catch( Autodesk.Revit.Exceptions
        .OperationCanceledException )
      {
        return Result.Cancelled;
      }
    }
    return Result.Succeeded;
  }
}
```

#### Download

As always, the most up to date version of The Building Coder samples is provided in
[its GitHub repository](https://github.com/jeremytammik/the_building_coder_samples),
and the version including the new JtPairPicker class and CmdDimensionInstanceOrigin external command described above is
[release 2015.0.116.0](https://github.com/jeremytammik/the_building_coder_samples/releases/tag/2015.0.116.0).