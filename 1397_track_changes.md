---
post_number: "1397"
title: "Track Changes"
slug: "track_changes"
author: "Jeremy Tammik"
tags: ['csharp', 'doors', 'elements', 'family', 'filtering', 'geometry', 'levels', 'parameters', 'python', 'revit-api', 'rooms', 'sheets', 'transactions', 'views', 'walls']
source_file: "1397_track_changes.md"
original_url: "https://thebuildingcoder.typepad.com/blog/1397_track_changes.html"
---

### Tracking Element Modification
Several people asked me recently about how to determine and track element modification, and I have heard this same question dozens of times in the more distant past as well.
Now it was brought up again here at the Munich Cloud Accelerator by John Allan-Jones of [Atkins](http://atkinsglobal.com), and I was motivated to implement a solution:
- [Two approaches](#0)
- [Task analysis](#1)
- [Modification tracker](#2)
- [Creating an element state snapshot](#3)
- [Determining which elements to track](#4)
- [Implementation](#5)
- [Geometrical comparison](#5.1)
- [String formatting](#5.2)
- [Retrieve solid vertices](#5.3)
- [GetTrackedElements – retrieve elements of interest](#5.4)
- [GetElementState – store element state](#5.5)
- [Creating a database state snapshot](#5.6)
- [Report differences](#5.7)
- [External command mainline Execute method](#5.8)
- [Sample runs](#6)
- [Demo recording](#7)
- [Download](#8)
For a quick first impression, you can jump to the [two-and-a-half minute video recording](#7) showing it in action and read [an initial comment or two](http://forums.autodesk.com/t5/revit-api/tracking-revit-bim-database-and-individual-element-property/m-p/5998729).
#### Two Approaches
Most ideas for approaches to this I heard so far were pretty fixed on tracking individual events in the Revit database, e.g. using the DocumentChanged event or a DMU framework dynamic model updater and keeping track of every modification.
In both cases, the add-in will be receiving and processing huge amounts of useless data and struggling to filter out what is really needed.
I suggest a radically different approach that is completely independent of events, Revit internals, real-time analysis and continuous tracking.
I suggest simply creating a snapshot that captures all the properties of interest and reporting the differences between two such snapshots.
#### Task Analysis
Let's take a step back and think about what we really are after.
To track changes, you need to consider what kind of changes you are interested in.
These changes have to do with two things:
- Which BIM elements are of interest?
- What defines the BIM element state that is of interest?
The former issue is a simple and pretty obvious matter of defining the correct filtered element collector filters to apply.
The second aspect is one that I'll address in more depth.
It is hopefully possible to capture the BIM element state that is of interest by querying its properties, e.g.:
- Name
- Class, family, type
- Location
- Geometry
- Parameter data
- Etc.
If somebody makes a change to the BIM model that you wish to track, how and where will that change affect the model?
Certainly there must be a property somewhere in the database that you can use to determine the state before and after and identify that the modification happened.
So let's use these thoughts to define a super simple and efficient modification tracker.
#### Modification Tracker
I suggest the following approach to implement a simple and efficient modification tracker:
- Start tracking by capturing and storing the relevant state of all elements of interest.
- Stop tracking by capturing an updated version of that data.
- Report changes by comparing the two snapshots.
No need for events!
No need for cumbersome instrumentation!
No continuous data collection overhead!
My sample implements one single command, which alternates between two actions:
Create and store a snapshot of the current database state if none is already present.
If a previous snapshot is already present, create a new one, compare them, and report the modifications.
You obviously might, if you wish, automatically execute the first action on document load and the second on document save.
If you create an external application to host a button triggering these two action, you would obviously make use of the possibility
to [roll your own toggle button](http://thebuildingcoder.typepad.com/blog/2012/11/roll-your-own-toggle-button.html).
#### Creating an Element State Snapshot
Retrieve all the element data that you need to capture the state of the element of interest to you, e.g., its location, geometry, parameter data or any other characteristics that you would like to track.
Encode that information into a string.
Compute a hash code for that string.
We use the hash code to determine whether the state has been modified compared to a new element state snapshot made at a later time.
We could obviously also store the entire original string representation instead of using a hash code. The hash code is small and handy, whereas the entire string contains all the original data. It is up to you to choose which you would like to use.
You need to ensure that every relevant change made to the tracked element really does make a difference to the string representation that you generate, and that the hash code you compute really is affected by every modification made.
This concept is similar to my appeal
to [create your own key](http://thebuildingcoder.typepad.com/blog/2012/03/great-ocean-road-and-creating-your-own-key.html#2),
and touches on some aspects of my more
recent [Revit project identification](http://the3dwebcoder.typepad.com/blog/2015/07/fireratingcloud-round-trip-and-on-mongolab.html#2).
One interesting sub-topic here is also how to create a sensible and useful canonical snapshot of the element geometry.
I had a stab back in 2009 analysing [nested instance geometry](http://thebuildingcoder.typepad.com/blog/2009/05/nested-instance-geometry.html) and implementing the `GetVertices` method that simply returns a sorted list of all unique vertices of a given solid in lexicographical order.
Since the element geometry can contain multiple solids, and they can be nested in a whole hierarchy of transformed instances, I need to traverse the element geometry, retrieve the solids from each level and apply the appropriate transformations to them, similarly to the approach used
to retrieve [real-world concrete corner coordinates](http://thebuildingcoder.typepad.com/blog/2012/06/real-world-concrete-corner-coordinates.html).
For family instances, however, I chose to ignore the symbol geometry. The family type or symbol should be considered a constant. Any change to the geometry in a family instance should be reflected in a corresponding change to the family type.
Therefore, I skip the geometry traversal and analysis for family instances and assume that these kind of changes to the family definition will be tracked elsewhere.
For a family instance, the snapshot just stores the family name, type and category instead of the geometry vertices.
The current initial implementation grabs the following element properties to create a snapshot of its state:
- Location
- Parameter names and values
- Family instance: family, type and category
- Not a family instance: bounding box and geometry vertices
The parameter retrieval and storage is based on similar code that I implemented and used in
the [RoomEditorApp](https://github.com/jeremytammik/RoomEditorApp) and
the [CompHound](https://github.com/CompHound/CompHound.github.io)
uploader [add-in](https://github.com/CompHound/CompHoundRvt).
Note that a parameter set is unordered. We need to sort the parameters by name to ensure a canonical reproducible representation, just like we need to sort the geometry vertices lexicographically for the same purpose. If we do not, they might be retrieved in an arbitrary different order later on and our string comparison would fail, even though the element data has not really changed.
As an example, here is the first element state string representation we generated:

```
Location=(0,0,0), (22.97,0,0),
Box=((-0.33,-0.33,0),(23.29,0.33,13.12)),
Vertices=(-0.33,-0.33,0), (-0.33,-0.33,13.12), (-0.33,0.33,0), (-0.33,0.33,13.12), (0.33,0.33,0), (0.33,0.33,13.12), (5.08,-0.33,3.94), (5.08,-0.33,5.94), (5.08,0.33,3.94), (5.08,0.33,5.94), (6.41,-0.33,3.94), (6.41,-0.33,5.94), (6.41,0.33,3.94), (6.41,0.33,5.94), (9.98,-0.33,0), (9.98,-0.33,7), (9.98,0.33,0), (9.98,0.33,7), (12.98,-0.33,0), (12.98,-0.33,7), (12.98,0.33,0), (12.98,0.33,7), (16.56,-0.33,3.94), (16.56,-0.33,5.94), (16.56,0.33,3.94), (16.56,0.33,5.94), (17.89,-0.33,3.94), (17.89,-0.33,5.94), (17.89,0.33,3.94), (17.89,0.33,5.94), (22.64,0.33,0), (22.64,0.33,13.12), (23.29,-0.33,0), (23.29,-0.33,13.12), (23.29,0.33,0), (23.29,0.33,13.12),
Parameters={"Area":"26 m²","Base Constraint":"Level 1","Base Extension Distance":"0","Base is Attached":"No","Base Offset":"0","Comments":"","Enable Analytical Model":"No","Image":"","Length":"7000","Location Line":"Wall Centerline","Mark":"","Phase Created":"New Construction","Phase Demolished":"None","Related to Mass":"No","Room Bounding":"Yes","Structural Usage":"Non-bearing","Structural":"No","Top Constraint":"Up to level: Level 2","Top Extension Distance":"0","Top is Attached":"No","Top Offset":"0","Unconnected Height":"4000","Volume":"5.27 m³"}
```

For the current final implementation, please refer to the [GetElementState method implementation](5.5) below.
#### Determining Which Elements to Track
Element retrieval from the database is always achieved using a filtered element collector.
Determining which elements to track is just a matter of defining the appropriate filters to apply to the collector.
We already looked at numerous different filtering examples in the past.
In this case, we may be interested in all or just certain sets of elements.
You will need to define that in detail for yourself depending on your exact needs.
Here are some of the existing examples and discussions on this topic:
- [Retrieving all elements](http://thebuildingcoder.typepad.com/blog/about-the-author.html#5.9)
- [Retrieving all model elements](http://thebuildingcoder.typepad.com/blog/2016/01/retrieving-all-model-elements.html)
- [Retrieving MEP elements and connectors](http://thebuildingcoder.typepad.com/blog/2010/06/retrieve-mep-elements-and-connectors.html)
- [Retrieving structural elements](http://thebuildingcoder.typepad.com/blog/2010/07/retrieve-structural-elements.html)
- [Recent thread on model categories](http://forums.autodesk.com/t5/revit-api/all-model-elements-in-project/m-p/5972035)
The current implementation selects all elements that:
- Are not ElementType objects
- Are view independent
- Have a category whose category type is `CategoryType.Model`
- Have a non-null bounding box
- Have non-null geometry
For the current final implementation, please refer to the [GetTrackedElements method implementation](5.4) below.
#### Implementation
I created the [TrackChanges GitHub repository](https://github.com/jeremytammik/TrackChanges) to host this project, including the entire source code, Visual Studio project and add-in manifest.
The version discussed here is [release 2016.0.0.1](https://github.com/jeremytammik/TrackChanges/releases/tag/2016.0.0.1).
If you are seriously interested in taking a deeper look, you will obviously fork and clone that repository for yourself and explore the code directly in the Visual Studio IDE.
Otherwise, for the sake of completeness and to help the Internet search engines find this discussion, let's present and discuss the complete source code for the external command and its various helper methods right here, divided into the following code regions:
- [Geometrical Comparison](#5.1)
- [String formatting](#5.2)
- [Retrieve solid vertices](#5.3)
- [Retrieve elements of interest](#5.4)
- [Store element state](#5.5)
- [Creating a Database State Snapshot](#5.6)
- [Report Differences](#5.7)
- [External Command Mainline Execute Method](#5.8)
#### Geometrical Comparison
Define helper functions and other support for geometrical comparisons:

```
  const double _eps = 1.0e-9;

  public static double Eps
  {
    get
    {
      return _eps;
    }
  }

  public static double MinLineLength
  {
    get
    {
      return _eps;
    }
  }

  public static double TolPointOnPlane
  {
    get
    {
      return _eps;
    }
  }

  public static bool IsZero(
    double a,
    double tolerance )
  {
    return tolerance > Math.Abs( a );
  }

  public static bool IsZero( double a )
  {
    return IsZero( a, _eps );
  }

  public static bool IsEqual( double a, double b )
  {
    return IsZero( b - a );
  }

  public static int Compare( double a, double b )
  {
    return IsEqual( a, b ) ? 0 : ( a < b ? -1 : 1 );
  }

  public static int Compare( XYZ p, XYZ q )
  {
    int d = Compare( p.X, q.X );

    if( 0 == d )
    {
      d = Compare( p.Y, q.Y );

      if( 0 == d )
      {
        d = Compare( p.Z, q.Z );
      }
    }
    return d;
  }
```

#### String Formatting
Define a bunch of helper functions to generate string representations of various objects:

```
  ///
  /// Convert a string to a byte array.
  ///
  static byte[] GetBytes( string str )
  {
    byte[] bytes = new byte[str.Length
      * sizeof( char )];

    System.Buffer.BlockCopy( str.ToCharArray(),
      0, bytes, 0, bytes.Length );

    return bytes;
  }

  ///
  /// Return a string for a real number
  /// formatted to two decimal places.
  ///
  public static string RealString( double a )
  {
    return a.ToString( "0.##" );
  }

  ///
  /// Return a string for an XYZ point
  /// or vector with its coordinates
  /// formatted to two decimal places.
  ///
  public static string PointString( XYZ p )
  {
    return string.Format( "({0},{1},{2})",
      RealString( p.X ),
      RealString( p.Y ),
      RealString( p.Z ) );
  }

  ///
  /// Return a string for this bounding box
  /// with its coordinates formatted to two
  /// decimal places.
  ///
  public static string BoundingBoxString(
    BoundingBoxXYZ bb )
  {
    return string.Format( "({0},{1})",
      PointString( bb.Min ),
      PointString( bb.Max ) );
  }

  ///
  /// Return a string for this point array
  /// with its coordinates formatted to two
  /// decimal places.
  ///
  public static string PointArrayString( IList<XYZ> pts )
  {
    return string.Join( ", ",
      pts.Select<XYZ, string>(
        p => PointString( p ) ) );
  }

  ///
  /// Return a string for this curve with its
  /// tessellated point coordinates formatted
  /// to two decimal places.
  ///
  public static string CurveTessellateString(
    Curve curve )
  {
    return PointArrayString( curve.Tessellate() );
  }

  ///
  /// Return a string for this curve with its
  /// tessellated point coordinates formatted
  /// to two decimal places.
  ///
  public static string LocationString(
    Location location )
  {
    LocationPoint lp = location as LocationPoint;
    LocationCurve lc = ( null == lp )
      ? location as LocationCurve
      : null;

    return null == lp
      ? ( null == lc
        ? null
        : CurveTessellateString( lc.Curve ) )
      : PointString( lp.Point );
  }

  ///
  /// Return a JSON string representing a dictionary
  /// of the given parameter names and values.
  ///
  public static string GetPropertiesJson(
    IList<Parameter> parameters )
  {
    int n = parameters.Count;
    List<string> a = new List<string>( n );
    foreach( Parameter p in parameters )
    {
      a.Add( string.Format( "\"{0}\":\"{1}\"",
        p.Definition.Name, p.AsValueString() ) );
    }
    a.Sort();
    string s = string.Join( ",", a );
    return "{" + s + "}";
  }

  ///
  /// Return a string describing the given element:
  /// .NET type name,
  /// category name,
  /// family and symbol name for a family instance,
  /// element id and element name.
  ///
  public static string ElementDescription(
    Element e )
  {
    if( null == e )
    {
      return "";
    }

    // For a wall, the element name equals the
    // wall type name, which is equivalent to the
    // family name ...

    FamilyInstance fi = e as FamilyInstance;

    string typeName = e.GetType().Name;

    string categoryName = ( null == e.Category )
      ? string.Empty
      : e.Category.Name + " ";

    string familyName = ( null == fi )
      ? string.Empty
      : fi.Symbol.Family.Name + " ";

    string symbolName = ( null == fi
      || e.Name.Equals( fi.Symbol.Name ) )
        ? string.Empty
        : fi.Symbol.Name + " ";

    return string.Format( "{0} {1}{2}{3}<{4} {5}>",
      typeName, categoryName, familyName,
      symbolName, e.Id.IntegerValue, e.Name );
  }

  public static string ElementDescription(
    Document doc,
    int element_id )
  {
    return ElementDescription( doc.GetElement(
      new ElementId( element_id ) ) );
  }
```

#### Retrieve Solid Vertices
Retrieve solid vertices and sort them lexicographically to define an extremely simplified form of a partial canonical element geometry representation.

```
  ///
  /// Define equality between XYZ objects, ensuring
  /// that almost equal points compare equal.
  ///
  class XyzEqualityComparer : IEqualityComparer<XYZ>
  {
    public bool Equals( XYZ p, XYZ q )
    {
      return p.IsAlmostEqualTo( q );
    }

    public int GetHashCode( XYZ p )
    {
      return PointString( p ).GetHashCode();
    }
  }

  ///
  /// Add the vertices of the given solid to
  /// the vertex lookup dictionary.
  ///
  static void AddVertices(
    Dictionary<XYZ, int> vertexLookup,
    Transform t,
    Solid s )
  {
    Debug.Assert( 0 < s.Edges.Size,
      "expected a non-empty solid" );

    foreach( Face f in s.Faces )
    {
      Mesh m = f.Triangulate();

      foreach( XYZ p in m.Vertices )
      {
        XYZ q = t.OfPoint( p );

        if( !vertexLookup.ContainsKey( q ) )
        {
          vertexLookup.Add( q, 1 );
        }
        else
        {
          ++vertexLookup[q];
        }
      }
    }
  }

  ///
  /// Recursively add vertices of all solids found
  /// in the given geometry to the vertex lookup.
  /// Untested!
  ///
  static void AddVertices(
    Dictionary<XYZ, int> vertexLookup,
    Transform t,
    GeometryElement geo )
  {
    if( null == geo )
    {
      Debug.Assert( null != geo, "null GeometryElement" );
      throw new System.ArgumentException( "null GeometryElement" );
    }

    foreach( GeometryObject obj in geo )
    {
      Solid solid = obj as Solid;

      if( null != solid )
      {
        if( 0 < solid.Faces.Size )
        {
          AddVertices( vertexLookup, t, solid );
        }
      }
      else
      {
        GeometryInstance inst = obj as GeometryInstance;

        if( null != inst )
        {
          //GeometryElement geoi = inst.GetInstanceGeometry();
          GeometryElement geos = inst.GetSymbolGeometry();

          //Debug.Assert( null == geoi || null == geos,
          //  "expected either symbol or instance geometry, not both" );

          Debug.Assert( null != inst.Transform,
            "null inst.Transform" );

          //Debug.Assert( null != inst.GetSymbolGeometry(),
          //  "null inst.GetSymbolGeometry" );

          if( null != geos )
          {
            AddVertices( vertexLookup,
              inst.Transform.Multiply( t ),
              geos );
          }
        }
      }
    }
  }

  ///
  /// Return a sorted list of all unique vertices
  /// of all solids in the given element's geometry
  /// in lexicographical order.
  ///
  static List<XYZ> GetCanonicVertices( Element e )
  {
    GeometryElement geo = e.get_Geometry( new Options() );
    Transform t = Transform.Identity;

    Dictionary<XYZ, int> vertexLookup
      = new Dictionary<XYZ, int>(
        new XyzEqualityComparer() );

    AddVertices( vertexLookup, t, geo );

    List<XYZ> keys = new List<XYZ>( vertexLookup.Keys );

    keys.Sort( Compare );

    return keys;
  }
```

#### GetTrackedElements – Retrieve Elements of Interest
Retrieve all the elements of interest:

```
  ///
  /// Retrieve all elements to track.
  /// It is up to you to decide which elements
  /// are of interest to you.
  ///
  static IEnumerable<Element> GetTrackedElements(
    Document doc )
  {
    Categories cats = doc.Settings.Categories;

    List<ElementFilter> a = new List<ElementFilter>();

    foreach( Category c in cats )
    {
      if( CategoryType.Model == c.CategoryType )
      {
        a.Add( new ElementCategoryFilter( c.Id ) );
      }
    }

    ElementFilter isModelCategory
      = new LogicalOrFilter( a );

    Options opt = new Options();

    return new FilteredElementCollector( doc )
      .WhereElementIsNotElementType()
      .WhereElementIsViewIndependent()
      .WherePasses( isModelCategory )
      .Where<Element>( e =>
        ( null != e.get_BoundingBox( null ) )
        && ( null != e.get_Geometry( opt ) ) );
  }
```

#### GetElementState – Store Element State
Determine the state of an element and encode it as a string:

```
  ///
  /// Return a string representing the given element
  /// state. This is the information you wish to track.
  /// It is up to you to ensure that all data you are
  /// interested in really is included in this snapshot.
  /// In this case, we ignore all elements that do not
  /// have a valid bounding box.
  ///
  static string GetElementState( Element e )
  {
    string s = null;

    BoundingBoxXYZ bb = e.get_BoundingBox( null );

    if( null != bb )
    {
      List<string> properties = new List<string>();

      properties.Add( ElementDescription( e )
        + " at " + LocationString( e.Location ) );

      if( !( e is FamilyInstance ) )
      {
        properties.Add( "Box="
          + BoundingBoxString( bb ) );

        properties.Add( "Vertices="
          + PointArrayString( GetCanonicVertices( e ) ) );
      }

      properties.Add( "Parameters="
        + GetPropertiesJson( e.GetOrderedParameters() ) );

      s = string.Join( ", ", properties );

      //Debug.Print( s );
    }
    return s;
  }
```

#### Creating a Database State Snapshot
Retrieve each element's state, encode it as a string and store their resulting hash codes in a dictionary mapping element id to hash code:

```
  ///
  /// Return a dictionary mapping element id values
  /// to hash codes of the element state strings.
  /// This represents a snapshot of the current
  /// database state.
  ///
  static Dictionary<int, string> GetSnapshot(
    IEnumerable<Element> a )
  {
    Dictionary<int, string> d
      = new Dictionary<int, string>();

    SHA256 hasher = SHA256Managed.Create();

    foreach( Element e in a )
    {
      //Debug.Print( e.Id.IntegerValue.ToString()
      //  + " " + e.GetType().Name );

      string s = GetElementState( e );

      if( null != s )
      {
        string hashb64 = Convert.ToBase64String(
          hasher.ComputeHash( GetBytes( s ) ) );

        d.Add( e.Id.IntegerValue, hashb64 );
      }
    }
    return d;
  }
```

#### Report differences
Determine and report the differences between the two states at the start and end of the tracking period:

```
  ///
  /// Compare the start and end states and report the
  /// differences found. In this implementation, we
  /// just store a hash code of the element state.
  /// If you choose to store the full string
  /// representation, you can use that for comparison,
  /// and then report exactly what changed and the
  /// original values as well.
  ///
  static void ReportDifferences(
    Document doc,
    Dictionary<int, string> start_state,
    Dictionary<int, string> end_state )
  {
    int n1 = start_state.Keys.Count;
    int n2 = end_state.Keys.Count;

    List<int> keys = new List<int>( start_state.Keys );

    foreach( int id in end_state.Keys )
    {
      if( !keys.Contains( id ) )
      {
        keys.Add( id );
      }
    }

    keys.Sort();

    int n = keys.Count;

    Debug.Print(
      "{0} elements before, {1} elements after, {2} total",
      n1, n2, n );

    int nAdded = 0;
    int nDeleted = 0;
    int nModified = 0;
    int nIdentical = 0;
    List<string> report = new List<string>();

    foreach( int id in keys )
    {
      if( !start_state.ContainsKey( id ) )
      {
        ++nAdded;
        report.Add( id.ToString() + " added "
          + ElementDescription( doc, id ) );
      }
      else if( !end_state.ContainsKey( id ) )
      {
        ++nDeleted;
        report.Add( id.ToString() + " deleted" );
      }
      else if( start_state[id] != end_state[id] )
      {
        ++nModified;
        report.Add( id.ToString() + " modified "
          + ElementDescription( doc, id ) );
      }
      else
      {
        ++nIdentical;
      }
    }

    string msg = string.Format(
      "Stopped tracking changes now.\r\n"
      + "{0} deleted, {1} added, {2} modified, "
      + "{3} identical elements:",
      nDeleted, nAdded, nModified, nIdentical );

    string s = string.Join( "\r\n", report );

    Debug.Print( msg + "\r\n" + s );
    TaskDialog dlg = new TaskDialog( "Track Changes" );
    dlg.MainInstruction = msg;
    dlg.MainContent = s;
    dlg.Show();
  }
```

#### External Command Mainline Execute Method
Note that this command makes no modifications to the Revit database, so it uses the `ReadOnly` transaction mode.
The static variable `_start_state` stores the initial snapshot when we start tracking changes:

```
  ///
  /// Current snapshot of database state.
  /// You could also store the entire element state
  /// strings here, not just their hash code, to
  /// report their complete original and modified
  /// values.
  ///
  static Dictionary<int, string> _start_state = null;
```

The mainline checks whether `_start_state` has been initialised.
If not, we create a snapshot to start tracking changes and report so to the user.
Otherwise, a new snapshot of the end state is created and the differences are reported:

```
  public Result Execute(
    ExternalCommandData commandData,
    ref string message,
    ElementSet elements )
  {
    UIApplication uiapp = commandData.Application;
    UIDocument uidoc = uiapp.ActiveUIDocument;
    Application app = uiapp.Application;
    Document doc = uidoc.Document;

    IEnumerable<Element> a = GetTrackedElements( doc );

    if( null == _start_state )
    {
      _start_state = GetSnapshot( a );
      TaskDialog.Show( "Track Changes",
        "Started tracking changes now." );
    }
    else
    {
      Dictionary<int, string> end_state = GetSnapshot( a );
      ReportDifferences( doc, _start_state, end_state );
      _start_state = null;
    }
    return Result.Succeeded;
  }
```

#### Sample Runs
Let's run the element modification tracker in a real project, e.g. the Revit sample rac_advanced_sample.rvt:
![Revit rac_advanced_sample BIM](img/track_rac_advanced_sample.png)
Zoom in to some simple wall that we can modify:
![Original wall](img/track_wall_original.png)
Move it a little bit:
![Moved wall](img/track_wall_original.png)
That generates the following report:

```
5603 elements before, 5603 elements after, 5603 total
139803 modified Floor Floors <139803 Hollow Core Plank - Concrete Topping>
139829 modified Floor Floors <139829 Hollow Core Plank - Concrete Topping>
152882 modified Wall Walls <152882 Interior - 138mm Partition (1-hr)>
153000 modified Wall Walls <153000 Interior - 138mm Partition (1-hr)>
153435 modified Wall Walls <153435 Interior - 138mm Partition (1-hr)>
177329 modified Room Rooms <177329 Corridor 107>
0 deleted, 0 added, 6 modified, 5597 identical elements
```

Obviously, as always, just moving one single wall a little bit affected a number of related objects, e.g., neighbouring walls, floors, and rooms.
Slightly more interesting test after adding a dialogue box to display the report:
Again, we'll just move one single wall; this one will affect more elements:
![Original wall](img/track_wall2_original.png)
Move this one down a bit:
![Moved wall](img/track_wall2_moved.png)
That generates the following report:

```
5603 elements before, 5603 elements after, 5603 total
0 deleted, 0 added, 24 modified, 5579 identical elements:
139803 modified Floor Floors <139803 Hollow Core Plank - Concrete Topping>
139829 modified Floor Floors <139829 Hollow Core Plank - Concrete Topping>
152037 modified Wall Walls <152037 Interior - 138mm Partition (1-hr)>
152111 modified Wall Walls <152111 Interior - 138mm Partition (1-hr)>
152271 modified Wall Walls <152271 Interior - 138mm Partition (1-hr)>
152347 modified Wall Walls <152347 Interior - 138mm Partition (1-hr)>
152622 modified FamilyInstance Doors M_Single-Flush <152622 0915 x 2134mm>
152688 modified FamilyInstance Doors M_Single-Flush <152688 0915 x 2134mm>
152882 modified Wall Walls <152882 Interior - 138mm Partition (1-hr)>
153000 modified Wall Walls <153000 Interior - 138mm Partition (1-hr)>
153162 modified Wall Walls <153162 Interior - 138mm Partition (1-hr)>
153242 modified Wall Walls <153242 Interior - 138mm Partition (1-hr)>
156935 modified Opening Shaft Openings <156935 Opening Cut>
168671 modified Ceiling Ceilings <168671 600 x 600mm Grid>
168679 modified Ceiling Ceilings <168679 600 x 600mm Grid>
168687 modified Ceiling Ceilings <168687 600 x 600mm Grid>
168695 modified Ceiling Ceilings <168695 600 x 600mm Grid>
168894 modified Ceiling Ceilings <168894 600 x 600mm Grid>
168902 modified Ceiling Ceilings <168902 600 x 600mm Grid>
177324 modified Room Rooms <177324 Electrical 112>
177325 modified Room Rooms <177325 Lounge 111>
177326 modified Room Rooms <177326 Men 110>
177328 modified Room Rooms <177328 Women 109>
177329 modified Room Rooms <177329 Corridor 107>
```

It is displayed in a task dialogue like this:
![Tracked element report](img/track_wall2_moved_report.png)
For real-world usage, you would obviously implement a more intelligent reporting system, for example a two-tiered one with a top-level summary displayed to the user and a detailed report stored in a log file or somewhere.
Once again: do not forget that the external command TrackChanges acts as a toggle: every second call creates and stores a snapshot, every second one creates a new snapshot, compares it with the stored one and reports the differences.
#### Demo Recording
Here is a [two-and-a-half minute video recording](https://vimeo.com/152442481) showing it in action:

#### Download
This project lives in the [TrackChanges GitHub repository](https://github.com/jeremytammik/TrackChanges) and
the version discussed above is [release 2016.0.0.1](https://github.com/jeremytammik/TrackChanges/releases/tag/2016.0.0.1).