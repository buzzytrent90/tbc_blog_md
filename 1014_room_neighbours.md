---
ai_optimized: true
author: Jeremy Tammik
complexity_score: 7.5
content_type: code_example
optimization_date: '2025-12-11T11:44:15.122012'
original_url: https://thebuildingcoder.typepad.com/blog/1014_room_neighbours.html
post_number: '1014'
reading_time_minutes: 11
series: general
slug: room_neighbours
source_file: 1014_room_neighbours.htm
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
- rooms
- selection
- transactions
- walls
- windows
title: Room Neighbours
word_count: 2189
---

### Room Neighbours

Let's say hello to our neighbours.

This issue was raised and also solved by Erik Eriksson of
[White Arkitketer AB](http://www.white.se),
who wishes to determine the neighbouring rooms for any given room.

It is an extension of the very early discussion of
[room and wall adjacency](http://thebuildingcoder.typepad.com/blog/2009/01/room-and-wall-adjacency.html) from
January 2009 that just looks at determining the neighbouring walls for a given room.

The Revit API now provides much more powerful tools for this kind of analysis, enabling us to easily go the one step further and find out what the neighbouring room is on the other side of each wall.

In the sample code presented below, this analysis is only performed in the wall midpoint.
It could easily be extended, though, to determine multiple neighbouring rooms on the other side, splitting the wall location line into a separate interval for each.

**Question:** I'm trying to find rooms adjacent to other rooms.

Basically the pseudocode would be something like this:

- Get all the rooms in the document.
- For each room, find all adjacent rooms (i.e. the room on the other side of the wall) and the wall separating them.

Is there any method for finding adjacent rooms?
First I was thinking about ReferenceIntersector, but that doesn't work on rooms, right?

Then I was thinking about using the Room.IsPointInRoom method and providing it with a point that I project out from the wall, but this means iterating over all the rooms for each of the walls for each room.
That feels like it's going to take a while for a large project; for instance, given 1000 rooms with 6 walls each would require 6 000 000 iterations.

**Answer:** The API does not provide any direct method for obtaining neighbouring rooms.

What you suggest sounds like a good starting point for further analysis to me.

What you need is basically a sort of graph structure.

You could partition the entire 2D plane on any given level into polygons that fall into one of three classes: within a room, within a wall, or outside the building.
If you then shrink away the walls, all the neighbouring room polygons of any given room are its adjacent rooms.

If you had a 2D geometry library which automatically generated a graph of neighbouring polygons for you, that might be an attractive and reliable way to go.

How to obtain this data most efficiently without going through the 2D geometrical analysis is not obvious to me either.

The Room.IsPointInRoom method that you suggest is definitely not a very good candidate, though, since, as you say, it requires you to know which room you are looking for in advance, or test all rooms, as you suggest, a really bad idea.
You could also limit the rooms to test to the immediate vicinity of the target room, which would improve performance tremendously, of course.

A much more efficient algorithm could be based on the Document.GetRoomAtPoint method instead, using points projected out of the target room through the bounding walls a certain small distance into the neighbouring space, because that just requires the input point, not a room, and gives you the neighbouring room.

One open question is how to decide the number of attempts to make to find different neighbouring rooms on the other side of any one given wall.

You always know that your rooms are larger than a certain minimum size, because you can simply iterate over them all and find out what the smallest width or height of any room is.

Therefore, you know that you only need to perform the 'project point through wall and determine neighbour from GetRoomAtPoint' at points at a certain given distance, e.g. every 50 cm or so.

This is assuming that you do not have really crazily shaped rooms, such as lots of wedges which all meet at a central small circular room, which then might have a large number of neighbours at arbitrarily small distances around its one and only circular wall.

Here are some related discussions:

- [Room and wall adjacency](http://thebuildingcoder.typepad.com/blog/2009/01/room-and-wall-adjacency.html)
- [Room and wall adjacent area](http://thebuildingcoder.typepad.com/blog/2009/01/room-and-wall-adjacent-area.html)
- [Space adjacency for heat load calculation](http://thebuildingcoder.typepad.com/blog/2013/07/football-and-space-adjacency-for-heat-load-calculation.html)

**Response:** The get room at point method worked great!

I go through all the boundary segments of all the rooms and see if there is a room on the other side.

Here is the method that I use to find the room on the other side at the midpoint of a boundary segment:

```python
  /// <summary>
  /// Return the neighbouring room to the given one
  /// on the other side of the midpoint of the given
  /// boundary segment.
  /// </summary>
  Room GetRoomNeighbourAt(
    BoundarySegment bs,
    Room r )
  {
    Document doc = r.Document;

    Wall w = bs.Element as Wall;

    double wallThickness = w.Width;

    double wallLength = ( w.Location as
      LocationCurve ).Curve.Length;

    Transform derivatives = bs.Curve
      .ComputeDerivatives(  0.5, true );

    XYZ midPoint = derivatives.Origin;

    Debug.Assert(
      midPoint.IsAlmostEqualTo(
        bs.Curve.Evaluate( 0.5, true ) ),
      "expected same result from Evaluate and derivatives" );

    XYZ tangent = derivatives.BasisX.Normalize();

    XYZ normal = new XYZ( tangent.Y,
      tangent.X \* ( -1 ), tangent.Z );

    XYZ p = midPoint + wallThickness \* normal;

    Room otherRoom = doc.GetRoomAtPoint( p );

    if( null != otherRoom )
    {
      if( otherRoom.Id == r.Id )
      {
        normal = new XYZ( tangent.Y \* ( -1 ),
          tangent.X, tangent.Z );

        p = midPoint + wallThickness \* normal;

        otherRoom = doc.GetRoomAtPoint( p );

        Debug.Assert( null == otherRoom
            || otherRoom.Id != r.Id,
          "expected different room on other side" );
      }
    }
    return otherRoom;
  }
```

The if-statement makes sure that I'm looking in the correct direction with regard to the wall direction.
I reverse the vector if the first attempt returns the same room as the one I already have.

Many thanks to Erik for the discussion and solution to this question!

#### Room Neighbour External Command

I added this code to The Building Coder samples as a new command CmdRoomNeighbours.

It reports the neighbouring rooms at the midpoints of all the boundary segments of a user-selected set of rooms.

#### JtSelectorMulti – a Generic Multiple Pre- and Post-Selector Class

The selection process supports both pre- and post-selection.
You can optionally select the rooms before launching the command.
If you do not, you will be prompted to do so interactively by the command itself.

Since I repeatedly mentioned that quite a number of lines of code are required to support pre- and post-selection, I decided to package that into a separate class and implemented a generic templated JtSelectorMulti class for the selection of multiple elements.

It is templated by the type of element to select.
You can also specify the required built-in category (or null for none) and a freely definable filtering method.

Using that, the pre- and post-selection support in the external command Execute method mainline now just requires these three statements:

```csharp
  // Interactively select elements of type Room,
  // either via pre-selection before launching the
  // command, or interactively via post-selection.

  JtSelectorMulti<Room> selector
    = new JtSelectorMulti<Room>(
      uidoc, BuiltInCategory.OST\_Rooms, "room",
      e => e is Room );

  if( selector.IsEmpty )
  {
    return selector.ShowResult();
  }

  IList<Room> rooms = selector.Selected;
```

Here is the class implementation including some comments:

```python
/// <summary>
/// Select multiple elements of the same type using
/// either pre-selection, before launching the
/// command, or post-selection, afterwards.
/// The element type is determined by the template
/// parameter. A filtering method must be provided
/// and is reused for both testing the pre-selection
/// and defining allowable elements for the post-
/// selection.
/// </summary>
class JtSelectorMulti<T> where T : Element
{
  /// <summary>
  /// Error message in case of invalid pre-selection.
  /// </summary>
  const string \_usage\_error = "Please pre-select "
    + "only {0}s before launching this command.";

  /// <summary>
  /// Determine whether the given element is a valid
  /// selectable object. The method passed in is
  /// reused for both the interactive selection
  /// filter and the pre-selection validation.
  /// See below for a sample method.
  /// </summary>
  public delegate bool IsSelectable( Element e );

  #region Sample common filtering helper method
  /// <summary>
  /// Determine whether the given element is valid.
  /// This specific implementation requires a family
  /// instance element of the furniture category
  /// belonging to the named family.
  /// </summary>
  static public bool IsTable( Element e )
  {
    bool rc = false;

    Category cat = e.Category;

    if( null != cat )
    {
      if( cat.Id.IntegerValue.Equals(
        (int) BuiltInCategory.OST\_Furniture ) )
      {
        FamilyInstance fi = e as FamilyInstance;

        if( null != fi )
        {
          string fname = fi.Symbol.Family.Name;

          rc = fname.Equals( "SampleTableFamilyName" );
        }
      }
    }
    return rc;
  }
  #endregion // Common filtering helper method

  #region JtSelectionFilter
  class JtSelectionFilter : ISelectionFilter
  {
    Type \_t;
    BuiltInCategory? \_bic;
    IsSelectable \_f;

    public JtSelectionFilter(
      Type t,
      BuiltInCategory? bic,
      IsSelectable f )
    {
      \_t = t;
      \_bic = bic;
      \_f = f;
    }

    bool HasBic( Element e )
    {
      return null == \_bic
        || (null != e.Category
          && e.Category.Id.IntegerValue.Equals(
            (int) \_bic ));
    }

    public bool AllowElement( Element e )
    {
      return e is T
        && HasBic( e )
        && \_f( e );
    }

    public bool AllowReference( Reference r, XYZ p )
    {
      return true;
    }
  }
  #endregion // JtSelectionFilter

  List<T> \_selected;
  string \_msg;
  Result \_result;

  /// <summary>
  /// Instantiate and run element selector.
  /// </summary>
  /// <param name="uidoc">UIDocument.</param>
  /// <param name="bic">Built-in category or null.</param>
  /// <param name="description">Description of the elements to select.</param>
  /// <param name="f">Validation method.</param>
  public JtSelectorMulti(
    UIDocument uidoc,
    BuiltInCategory? bic,
    string description,
    IsSelectable f )
  {
    \_selected = null;
    \_msg = null;

    Document doc = uidoc.Document;

    if( null == doc )
    {
      \_msg = "Please run this command in a valid"
        + " Revit project document.";
      \_result = Result.Failed;
    }

    // Check for pre-selected elements

    Selection sel = uidoc.Selection;

    int n = sel.Elements.Size;

    if( 0 < n )
    {
      foreach( Element e in sel.Elements )
      {
        if( !f( e ) )
        {
          \_msg = string.Format(
            \_usage\_error, description );

          \_result = Result.Failed;
        }

        if( null == \_selected )
        {
          \_selected = new List<T>( n );
        }

        \_selected.Add( e as T );
      }
    }

    // If no elements were pre-selected,
    // prompt for post-selection

    if( null == \_selected
      || 0 == \_selected.Count )
    {
      IList<Reference> refs = null;

      try
      {
        refs = sel.PickObjects(
          ObjectType.Element,
          new JtSelectionFilter( typeof( T ), bic, f ),
          string.Format(
            "Please select {0}s.",
            description ) );
      }
      catch( Autodesk.Revit.Exceptions
        .OperationCanceledException )
      {
        \_result = Result.Cancelled;
      }

      if( null != refs && 0 < refs.Count )
      {
        \_selected = new List<T>(
          refs.Select<Reference, T>(
            r => doc.GetElement( r.ElementId )
              as T ) );
      }
    }

    Debug.Assert(
      null == \_selected || 0 < \_selected.Count,
      "ensure we return only non-empty collections" );

    \_result = ( null == \_selected )
      ? Result.Cancelled
      : Result.Succeeded;
  }

  /// <summary>
  /// Return true if nothing was selected.
  /// </summary>
  public bool IsEmpty
  {
    get
    {
      return null == \_selected
        || 0 == \_selected.Count;
    }
  }

  /// <summary>
  /// Return the cancellation or failure code
  /// to Revit and display a message if there
  /// is anything to say.
  /// </summary>
  public Result ShowResult()
  {
    if( Result.Failed == \_result )
    {
      Debug.Assert( 0 < \_msg.Length,
        "expected a non-empty error message" );

      Util.ErrorMsg( \_msg );
    }
    return \_result;
  }

  /// <summary>
  /// Return selected elements or null.
  /// </summary>
  public IList<T> Selected
  {
    get
    {
      return \_selected;
    }
  }
}
```

#### CmdRoomNeighbours External Command Mainline

Making use of the [JtSelectorMulti](#4) pre- and post-selection and the [GetRoomNeighbourAt](#2) method to list the results, here is the complete code of the external command mainline.

```csharp
  UIApplication uiapp = commandData.Application;
  UIDocument uidoc = uiapp.ActiveUIDocument;
  Document doc = uidoc.Document;

  // Interactively select elements of type Room,
  // either via pre-selection before launching the
  // command, or interactively via post-selection.

  JtSelectorMulti<Room> selector
    = new JtSelectorMulti<Room>(
      uidoc, BuiltInCategory.OST\_Rooms, "room",
      e => e is Room );

  if( selector.IsEmpty )
  {
    return selector.ShowResult();
  }

  IList<Room> rooms = selector.Selected;

  List<string> msg = new List<string>();

  int n = rooms.Count;

  msg.Add( string.Format(
    "{0} room{1} selected{2}\r\n",
    n, Util.PluralSuffix( n ),
    Util.DotOrColon( n ) ) );

  SpatialElementBoundaryOptions opt
    = new SpatialElementBoundaryOptions();

  IList<IList<BoundarySegment>> loops;

  Room neighbour;
  int i = 0, j, k;

  foreach( Room room in rooms )
  {
    ++i;

    loops = room.GetBoundarySegments( opt );

    n = loops.Count;

    msg.Add( string.Format(
      "{0}. {1} has {2} loop{3}{4}",
      i, Util.ElementDescription( room ),
      n, Util.PluralSuffix( n ),
      Util.DotOrColon( n ) ) );

    j = 0;

    foreach( IList<BoundarySegment> loop in loops )
    {
      ++j;

      n = loop.Count;

      msg.Add( string.Format(
        "  {0}. Loop has {1} boundary segment{2}{3}",
        j, n, Util.PluralSuffix( n ),
        Util.DotOrColon( n ) ) );

      k = 0;

      foreach( BoundarySegment seg in loop )
      {
        ++k;

        neighbour = GetRoomNeighbourAt( seg, room );

        msg.Add( string.Format(
          "    {0}. Boundary segment has neighbour {1}",
          k,
          (null==neighbour
            ? "<nil>"
            : Util.ElementDescription( neighbour )) ) );
      }
    }
  }

  Util.InfoMsg2( "Room Neighbours",
    string.Join( "\n", msg.ToArray() ) );

  return Result.Succeeded;
```

We use read-only transaction mode for this command, since no database modifications are required.

#### Sample Run and Download

I tested the command selecting room 5 in the following simplistic model:

![Sample model with neighbouring rooms](img/ee_room_neighbours_1.png)

Its neighbouring rooms are listed in a message box and in the Visual Studio debug output window, which reports:

```
Room Neighbours

1 room selected:

1. Room Rooms <197109 Room 5> has 1 loop:
  1. Loop has 4 boundary segments:
    1. Boundary segment has neighbour <nil>
    2. Boundary segment has neighbour Room Rooms <197103 Room 3>
    3. Boundary segment has neighbour Room Rooms <197112 Room 6>
    4. Boundary segment has neighbour Room Rooms <197115 Room 7>
```

Here is
[version 2014.0.103.0](zip/bc_14_103_0.zip) of
The Building Coder samples source code, Visual Studio solution and RvtSamples include file including the new CmdRoomNeighbours command.

I hope you find this useful and thank Erik again for raising this issue.