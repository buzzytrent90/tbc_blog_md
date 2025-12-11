---
ai_optimized: true
author: Jeremy Tammik
complexity_score: 4.1
content_type: qa
optimization_date: '2025-12-11T11:44:14.671417'
original_url: https://thebuildingcoder.typepad.com/blog/0823_filter_touching_beam.html
post_number: 0823
reading_time_minutes: 6
series: filtering
slug: filter_touching_beam
source_file: 0823_filter_touching_beam.htm
tags:
- csharp
- elements
- filtering
- geometry
- references
- revit-api
- rooms
- selection
- walls
title: Filter for Touching Beams Using Solid Intersection
word_count: 1265
---

### Filter for Touching Beams Using Solid Intersection

I discussed how I created my first
[sphere in the Revit API](http://thebuildingcoder.typepad.com/blog/2012/09/sphere-creation-for-avf-and-filtering.html) and
displayed it using the analysis visualisation framework
[AVF](http://thebuildingcoder.typepad.com/blog/avf).
Now let's make more serious use of it.

As I mentioned, the transient solids created by the geometry creation utility class can be used to set up geometrical proximity filters.

This exploration was prompted by the following query:

**Question:** I would like to programmatically retrieve all touching beams, regardless of whether they are officially connected or not.

The user should pick one single beam, and all other beams touching it should be selected, recursively.

How can I achieve this, please?

**Answer:** First let's look at the case you are not interested in :-)

If the beams are properly connected, you can use the LocationCurve ElementsAtJoin property to determine the connected beams at each end of the first, manually selected one, and iterate through the connection elements as demonstrated by the TraverseSystem SDK sample for duct systems using the MEP connector manager.

If the beams are not properly connected, but just touching, as you say, you can use the ElementIntersectsSolidFilter instead.
Such as filter detects elements within a given geometrical space defined by solid, which may come from the BIM model or be transient, i.e. generated in memory on the fly.

To recursively iterate through the beam system requires creating a new solid and a new filter for each beam you find, since each solid has its own location in space.
If you are interested in beams touching anywhere, you could create an extruded shape matching the beam shape and detect all other beams that intersect it or come close to it anywhere at all along the length of the beam.
If you are only interested in beams touching at the end points, you could set up a simple sphere at each end of the beam.

This process could be repeated recursively for all touching beams encountered.
Of course you would have to skip beams that you have already found, or you would run into an infinite loop.

Here is the algorithm I am thinking of:

- Maintain three lists of beams:
  - The beams already visited.- The current ones being analysed.- The neighbours of the current ones.- Select a starting beam, making it the current one.- While the set of current beams is non-empty, repeat the following:
      - Add the current to the list of visited beams.- Clear the list of neighbours.- For each of the current beams, add all beams touching it to the list of neighbours.- Make the neighbours found in the previous step the new set of current beams.

Finding all touching neighbours for a given beam is implement in the method AddConnectedElements, which sets up a sphere at each end of the beam, creates a proximity detector based on it using an ElementIntersectsSolidFilter, and adds all beams retrieved by that, excluding the beams already visited and neighbours already found.

I already presented the sphere creation method
[CreateSphereAt](http://thebuildingcoder.typepad.com/blog/2012/09/sphere-creation-for-avf-and-filtering.html#3).

Here is the AddConnectedElements method and its helper method AddElementsIntersectingSphereAt to detect touching elements:
```csharp
  /// <summary>
  /// Determine all neighbouring elements connected
  /// to the current element 'e', skipping all
  /// previously visited ones.
  /// </summary>
  void AddElementsIntersectingSphereAt(
    List<ElementId> neighbours,
    XYZ p,
    List<ElementId> visited,
    Document doc )
  {
    Solid sphere = CreateSphereAt(
      doc.Application.Create, p, \_sphere\_radius );

    ElementIntersectsSolidFilter intersectSphere
      = new ElementIntersectsSolidFilter( sphere );

    FilteredElementCollector collector
      = new FilteredElementCollector( doc )
        .WhereElementIsCurveDriven() // we work with the location curve
        .OfCategory( \_bic )
        .Excluding( visited.Union<ElementId>(
          neighbours ).ToList<ElementId>() )
        .WherePasses( intersectSphere );

    neighbours.AddRange( collector.ToElementIds() );
  }

  /// <summary>
  /// Determine all neighbouring elements close to
  /// the two ends of the current element 'e',
  /// skipping all previously visited ones.
  /// </summary>
  void AddConnectedElements(
    List<ElementId> neighbours,
    Element e,
    List<ElementId> visited )
  {
    Location loc = e.Location;

    Debug.Print( string.Format(
      "current element {0} has location {1}",
      ElementDescription( e ),
      null == loc ? "<null>" : loc.GetType().Name ) );

    LocationCurve lc = loc as LocationCurve;

    if( null != lc )
    {
      Document doc = e.Document;

      Curve c = lc.Curve;

      XYZ p = c.get\_EndPoint( 0 );
      XYZ q = c.get\_EndPoint( 1 );

      AddElementsIntersectingSphereAt(
        neighbours, p, visited, doc );

      AddElementsIntersectingSphereAt(
        neighbours, q, visited, doc );
    }
  }
```

The lists of neighbours and visited elements are implemented as collections of ElementId instead of Element instances, since I do not trust the .NET comparison methods to work properly on Elements.
Also, this aligns nicely with the Exclude method on the filtered element collector, which takes a collection of element ids as an argument.

I initially tried calling Exclude twice with separate calls for the lists of already visited elements and neighbours, both of which should be eliminated.
In the first call, though, the list of neighbours is always empty. Providing an empty list to the Exclude method throws an exception.
I could have called the Exclude method conditionally, checking first that the list of neighbours is non-empty.
However, I found it more succinct to combine the two lists into a single one for the call.

With the AddConnectedElements method in place, the algorithm described above translates to the following C# implementation of the external command Execute mainline method:
```csharp
public Result Execute(
  ExternalCommandData commandData,
  ref string message,
  ElementSet elements )
{
  UIApplication uiapp = commandData.Application;
  UIDocument uidoc = uiapp.ActiveUIDocument;
  Application app = uiapp.Application;
  CreationApp creapp = app.Create;
  Document doc = uidoc.Document;
  Selection sel = uidoc.Selection;
  Reference r = null;

  try
  {
    r = sel.PickObject(
      ObjectType.Element,
      "Please select a beam" );
  }
  catch( RvtOperationCanceledException )
  {
    return Result.Cancelled;
  }

  // Starting element

  Element start = doc.GetElement( r );

  // The current elements whose neighbours
  // we are seeking

  List<ElementId> current = new List<ElementId>();

  current.Add( start.Id );

  // List of elements already visited

  List<ElementId> visited = new List<ElementId>();

  // Continue as long as new connected
  // elements are found

  List<ElementId> neighbours = new List<ElementId>();

  while( 0 < current.Count )
  {
    // Remember where we have been, add this to
    // the result so far, and do not revisit these

    visited.AddRange( current );

    // We found no new neighbours yet

    neighbours.Clear();

    // Search current elements for new connected
    // elements not already visited

    foreach( ElementId id in current )
    {
      Element e = doc.GetElement( id );

      AddConnectedElements(
        neighbours, e, visited );
    }

    // Done with the current elements, and the
    // newly found become the next current ones

    current.Clear();
    current.AddRange( neighbours );
  }

  foreach( ElementId id in visited )
  {
    uidoc.Selection.Elements.Add(
      doc.GetElement( id ) );
  }
  return Result.Succeeded;
}
```

Pretty neat, huh?

And pretty readable, I think?

Here is a sample model with several sets of touching but not officially connected beams: two sets of two beams each on the right-hand side of the room in the middle, and a bunch of more beams all touching along the left, top and bottom. Some of them are longer than others and stretch across several columns:

![System of non-connected but touching beams](img/touching_beams_3.png)

After running the command and picking any single one of the beams on the left, the connected elements are all selected and therefore highlighted in blue, whereas the unconnected ones on the right remain unhighlighted:

![Touching beams detected and selected](img/touching_beams_4.png)

Note that this approach is useful for any curve based element, so it could also be adapted to find all touching walls, ducts, pipes, and many other object types.

Here is
[SelectTouchingBeams.zip](zip/SelectTouchingBeams.zip) containing
the source code, Visual Studio solution and add-in manifest of the 'Select Touching Beams' command.
It also includes the
[Display Sphere](http://thebuildingcoder.typepad.com/blog/2012/09/sphere-creation-for-avf-and-filtering.html#4) command that I discussed in the previous post.