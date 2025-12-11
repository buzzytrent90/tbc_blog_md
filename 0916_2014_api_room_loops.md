---
ai_optimized: true
author: Jeremy Tammik
complexity_score: 8.6
content_type: code_example
optimization_date: '2025-12-11T11:44:14.884085'
original_url: https://thebuildingcoder.typepad.com/blog/0916_2014_api_room_loops.html
post_number: 0916
reading_time_minutes: 9
series: general
slug: 2014_api_room_loops
source_file: 0916_2014_api_room_loops.htm
tags:
- csharp
- elements
- family
- filtering
- geometry
- python
- references
- revit-api
- rooms
- schedules
- selection
- views
- windows
title: Revit 2014 API and Plan View Room Boundary Loops
word_count: 1762
---

### Revit 2014 API and Plan View Room Boundary Loops

I am continuing the research and development for my
[cloud-based round-trip 2D Revit model editing project](http://thebuildingcoder.typepad.com/blog/2013/03/cloud-mobile-extensible-storage-data-use-in-schedules.html#3).

At the same time, Revit 2014 has been announced, and I am sure you are eager to hear more about that, especially from the API point of view, so let's have a look at that first.

#### The Revit 2014 API

It is impossible to cover everything, and I have to start somewhere.

Here are a couple of highlights:

- API access to the project browser and selected elements; API commands and macros are enabled.
- Copy and paste API supports copy within or between documents including view-specific elements.
- Full API support for the new non-rectangular crop regions.
- Schedule API now provides formatting control and read-write access to individual data items.
- Command API enables programmatic command launch including built-in Revit, external add-in and macro.
- Add-in API supports mid-session loading and execution.
- Displaced elements API enables exploded views.
- Join geometry API to create or remove a Boolean join and control join order.
- FreeForm element API enables modification of solid geometry imported from e.g. DWG or SAT.
- Site API enables editing of topography surface points and supports sub-regions.
- MEP calculations are moved to external services and can be replaced by add-ins.
- Structural reinforcement and rebar enhancements.
- Enhanced document open, save and worksharing API.
- Linked model API supports loading, unloading, path manipulation, link identification and creation.
- Linked model interaction enables tag creation for linked rooms, linked element selection, geometric reference conversion, etc.
- Import DXF markup, import and link SAT and SketchUp.
- Export to NavisWorks via add-in, access DWG, IFC and DGN layer, linetype, lineweight, font and pattern tables.
- Direct API access to rendering output pipeline including all geometry and material properties.
- Macro API provides support for Python and Ruby development plus List, create, delete and execute modules, macros and security settings.

How does that sound?

Rather a lot of new stuff, isn't there?

Rather a lot of **really exciting** new stuff, isn't there?

Joe Ye describes a few of these API features in a little more detail on the
[AEC DevBlog](http://adndevblog.typepad.com/aec/2013/03/revit-2014-announced.html).

To round this off, here are the complete materials from the Revit 2014 DevDays presentations:

- [Presentation](file:///a/lib/revit/2014/adn/devdays_online/revit_2014_api_presentation_slides.pdf)
- [Recording](http://thebuildingcoder.typepad.com/revit_2014_api/index.html)  [^](/a/lib/revit/2014/adn/devdays_online/camtasia/web/revit_2014_api/index.html)
- [Sample code](file:///a/lib/revit/2014/adn/devdays_online/revit_2014_api_sample_source_code.zip)

We will have all the time in the world to explore this in more detail anon.

The material provided above should keep you occupied over the Easter weekend, however :-)

Enjoy!

#### Retrieving Plan View Room Boundary Polygon Loops

Returning to the cloud-based 2D model editing project, one of the first required components is an add-in that determines and uploads the room, furniture and equipment family instance boundary polygons to some globally accessible data repository for a simplified 2D plan view rendering on a mobile device.

As a first step in that direction, I revamped my code to retrieve and
[graphically display area boundary loops](http://thebuildingcoder.typepad.com/blog/2012/08/graphically-display-area-boundary-loops.html) and
combined it with the
[integer-based point class](http://thebuildingcoder.typepad.com/blog/2012/06/obj-model-export-considerations.html#7) that
I implemented for the
[OBJ model exporter](http://thebuildingcoder.typepad.com/blog/2012/07/obj-model-exporter-with-transparency-support.html).

The task I want to achieve for the first part of this first step is to retrieve the room boundary and store the 2D loops in a cloud-based data repository.

Since Revit does not support precision below one sixteenth of an inch, I might as well approximate all my data to something in that region.

For performance and efficiency reasons, it is also useful to move my calculations from floating point double numbers to integers.

Since I want to display my model on a mobile device with a limited resolution using SVG, integers also seem pretty appropriate.

Very handily, a millimetre is just a little bit less than a sixteenth of an inch.

That leads me to define the following integer-based 2D point class:

```csharp
/// <summary>
/// An integer-based 2D point class.
/// </summary>
class Point2dInt : IComparable<Point2dInt>
{
  public int X { get; set; }
  public int Y { get; set; }

  const double \_feet\_to\_mm = 25.4 \* 12;

  static int ConvertFeetToMillimetres( double d )
  {
    return (int) ( \_feet\_to\_mm \* d + 0.5 );
  }

  /// <summary>
  /// Convert a 3D Revit XYZ to a 2D millimetre
  /// integer point by discarding the Z coordinate
  /// and scaling from feet to mm.
  /// </summary>
  public Point2dInt( XYZ p )
  {
    X = ConvertFeetToMillimetres( p.X );
    Y = ConvertFeetToMillimetres( p.Y );
  }

  public int CompareTo( Point2dInt a )
  {
    int d = X - a.X;

    if( 0 == d )
    {
      d = Y - a.Y;
    }
    return d;
  }

  public override string ToString()
  {
    return string.Format( "({0},{1})", X, Y );
  }
}
```

A room boundary may include several loops, for instance if a room surrounds some other space such as an elevator, i.e. its outer boundary loop contains some interior loops representing 'holes'.

Therefore, the room GetBoundarySegments method returns a list of loops, and each loop as a list of boundary segments:

```csharp
  IList<IList<BoundarySegment>> loops = room.
    GetBoundarySegments( opt );
```

I therefore define my own integer-based 2D loop and list of loops classes like this:

```python
  class JtLoop : List<Point2dInt>
  {
    public JtLoop( int capacity )
      : base( capacity )
    {
    }

    public override string ToString()
    {
      return string.Join( ", ", this );
    }
  }

  class JtLoops : List<JtLoop>
  {
    public JtLoops( int capacity )
      : base( capacity )
    {
    }
  }
```

The code to retrieve the boundary segments and convert them to my own representation can be implemented as follows:

```python
/// <summary>
/// Retrieve the room plan view boundary
/// polygon loops and convert to 2D integer-based.
/// For optimisation and consistency reasons,
/// convert all coordinates to integer values in
/// millimetres. Revit precision is limited to
/// 1/16 of an inch, which is abaut 1.2 mm, anyway.
/// </summary>
JtLoops GetRoomLoops( Room room )
{
  SpatialElementBoundaryOptions opt
    = new SpatialElementBoundaryOptions();

  opt.SpatialElementBoundaryLocation =
    SpatialElementBoundaryLocation.Center; // loops closed
    //SpatialElementBoundaryLocation.Finish; // loops not closed

  IList<IList<BoundarySegment>> loops = room.
    GetBoundarySegments( opt );

  int nLoops = loops.Count;

  JtLoops jtloops = new JtLoops( nLoops );

  foreach( IList<BoundarySegment> loop in loops )
  {
    int nSegments = loop.Count;

    JtLoop jtloop = new JtLoop( nSegments );

    XYZ p0 = null; // loop start point
    XYZ p; // segment start point
    XYZ q = null; // segment end point

    foreach( BoundarySegment seg in loop )
    {
      p = seg.Curve.get\_EndPoint( 0 );

      jtloop.Add( new Point2dInt( p ) );

      Debug.Assert( null == q || q.IsAlmostEqualTo( p ),
        "expected last endpoint to equal current start point" );

      q = seg.Curve.get\_EndPoint( 1 );

      Debug.Print( "{0} --> {1}",
        Util.PointString( p.ToUv() ),
        Util.PointString( q.ToUv() ) );

      if( null == p0 )
      {
        p0 = p; // save loop start point
      }
    }
    Debug.Assert( q.IsAlmostEqualTo( p0 ),
      "expected last endpoint to equal loop start point" );

    jtloops.Add( jtloop );
  }
  return jtloops;
}
```

My external command mainline Execute method driving this method also implements some fancy pre- and post-selection support and reporting code listing the contents of my 2D integer-based loops in the Visual Studio debug output window:

```python
public Result Execute(
  ExternalCommandData commandData,
  ref string message,
  ElementSet elements )
{
  UIApplication uiapp = commandData.Application;
  UIDocument uidoc = uiapp.ActiveUIDocument;
  Application app = uiapp.Application;
  Document doc = uidoc.Document;

  if( null == doc )
  {
    ErrorMsg( "Please run this command in a valid"
      + " Revit project document." );
    return Result.Failed;
  }

  // Iterate over all pre-selected rooms

  List<ElementId> ids = null;

  Selection sel = uidoc.Selection;

  if( 0 < sel.Elements.Size )
  {
    foreach( Element e in sel.Elements )
    {
      if( !( e is Room ) )
      {
        ErrorMsg( "Please pre-select only room"
          + " elements before running this command." );
        return Result.Failed;
      }

      if( null == ids )
      {
        ids = new List<ElementId>( 1 );
      }

      ids.Add( e.Id );
    }
  }

  // If no rooms were pre-selected,
  // prompt for post-selection

  if( null == ids )
  {
    IList<Reference> refs = null;

    try
    {
      refs = sel.PickObjects( ObjectType.Element,
        new RoomSelectionFilter(),
        "Please select rooms." );
    }
    catch( Autodesk.Revit.Exceptions
      .OperationCanceledException )
    {
      return Result.Cancelled;
    }
    ids = new List<ElementId>(
      refs.Select<Reference, ElementId>(
        r => r.ElementId ) );
  }

  foreach( ElementId id in ids )
  {
    Element e = doc.GetElement( id );

    Debug.Assert( e is Room,
      "expected parts only" );

    JtLoops roomLoops = GetRoomLoops( e as Room );

    int nLoops = roomLoops.Count;

    Debug.Print( "{0} has {1} loop{2}{3}",
      Util.ElementDescription( e ), nLoops,
      Util.PluralSuffix( nLoops ),
      Util.DotOrColon( nLoops ) );

    int i = 0;

    foreach( JtLoop loop in roomLoops )
    {
      Debug.Print( "  {0}: {1}", i++, loop.ToString() );
    }
  }
  return Result.Succeeded;
}
```

I tested this on a simple sample room with one hole:

![Room with a hole](img/room_with_hole.png)

The original start and end points of the boundary segments for this room are reported as follows:

```csharp
(9.03,10.13,0) --> (-14.59,10.13,0)
(-14.59,10.13,0) --> (-14.59,1.93,0)
(-14.59,1.93,0) --> (-2.45,1.93,0)
(-2.45,1.93,0) --> (-2.45,-3.98,0)
(-2.45,-3.98,0) --> (9.03,-3.98,0)
(9.03,-3.98,0) --> (9.03,10.13,0)
(0.98,-0.37,0) --> (0.98,1.93,0)
(0.98,1.93,0) --> (5.57,1.93,0)
(5.57,1.93,0) --> (5.57,-0.37,0)
(5.57,-0.37,0) --> (0.98,-0.37,0)
```

Converting these to my 2D integer-based loop classes and listing those generates the following debug output:

```csharp
Room Rooms <212639 Room 1> has 2 loops:
0: (2753,3087), (-4446,3087), (-4446,587),
(-746,587), (-746,-1212), (2753,-1212)
1: (298,-112), (298,587), (1698,587), (1698,-112)
```

So far, so good.

As far as I can tell, all systems go.

My next step for this add-in is to implement code to determine 2D plan view boundary polygons for the furniture and equipment family instances contained within the selected room.

I am hoping to be able to make use of the ExtrusionAnalyzer class for this.
As I mentioned, it is supplied a solid geometry, a plane, and a direction.
From those, it calculates the outer boundary of the shadow cast by the solid onto the input plane along the extrusion direction.

At the same time, I am continuing to explore options for a
[cloud-based data repository](http://thebuildingcoder.typepad.com/blog/2013/03/relax-simple-free-cloud-based-data-repository-with-nosql-couchdb-and-iriscouch.html).

And I have my day-to-day support tasks to attend to too...

Anyway, here is
[GetRoomLoops.zip](zip/GetRoomLoops.zip) containing
the complete source code, Visual Studio solution and add-in manifest of the current state of this external command.