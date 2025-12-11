---
ai_optimized: true
author: Jeremy Tammik
complexity_score: 6.9
content_type: code_example
optimization_date: '2025-12-11T11:44:13.478159'
original_url: https://thebuildingcoder.typepad.com/blog/0169_space_adjacency.html
post_number: 0169
reading_time_minutes: 8
series: general
slug: space_adjacency
source_file: 0169_space_adjacency.htm
tags:
- csharp
- doors
- elements
- python
- revit-api
- rooms
- selection
- walls
title: Space Adjacency
word_count: 1570
---

### Space Adjacency

We looked at the topic of
[room and wall adjacency](http://thebuildingcoder.typepad.com/blog/2009/01/room-and-wall-adjacency.html)
a while back.
Martin Schmid now implemented a nice little related utility to check adjacencies between spaces.
It demonstrates a beautiful use of the new Space.IsPointInSpace method.
Here is an example of the kind of situation he is interested in:

![Adjacent spaces](img/adjacent_spaces.png)

The aim is to obtain a list of all adjacent spaces from this model.

Martin added a new external command CmdSpaceAdjacency to The Building Coder sample application to analyse and report the space adjacency relationships in such a model.
Here are some more implementation notes on this from Martin:

Basically, this command starts by collecting all the segments from all spaces, tessellating any curved segments.
It then iterates each space's segments over all the other spaces' segments to find the segment 'closest' to it.
This is done by FindClosestSegments( segmentPairs, segments );

However, just because a pair of segments from two different spaces are 'close' to one another, doesn't necessarily mean they are to be considered adjacent.
There could be an outlying 'space', e.g., a storage shed or garage, and one surface of the space would be considered 'closest' to the main building itself, but in terms of being 'adjacent' for analysis, there is actually a considerable 'air space' between them.
Therefore, we need to limit how far apart spaces can be to be considered adjacent.

To achieve this, DetermineAdjacencies( spaceAdjacencies, segmentPairs ) checks whether the midpoints between each pair are closer than a specified tolerance.
It also calculates a test point that should be within the adjacent space and uses the new Space.IsPointInSpace method to check whether this is true.

This work s on the test models so far.
I have not yet tested on a larger dataset.
The implementation is probably not optimal, but is a start.

After speaking with a customer about this, I learned that they needed to limit 'adjacency' to spaces that share a common door between them.
Implementing this was actually a little more straightforward:
each door knows its 'to room' and 'from room', and this can be queried from the model by inspecting the 'linked' model data.
Each space knows what room encloses it, and this info is available in the 'host' model.
Thus, I was able to establish a relationship between doors and Spaces to figure out the adjacencies.
So that is yet another different method for a different analysis scenario.

It would be nice if there was actually a relationship in Revit between spaces and the walls that enclose it.
Also, this is all more cumbersome when working with linked files.
There may be another route, for instance if the ray intersection algorithm could be used to intelligently to seek for room or space boundaries and wall surfaces only.
However, I found that in linked models, it seems that there is no way to determine exactly what object is being hit by the ray.
Apparently, it only tells you that it hit a 'linked model', but not what particular object or type within that linked model.
There are probably some API requirements to eke out of these example scenarios, but I'll leave that for later.

So much for Martin's explanation of this algorithm and other related work of his.
Here are some more comments of mine on this command implementation based on reverse engineering:

The CmdSpaceAdjacency command works with the following data items:

- Segment: a helper class to manage a space boundary segment, managing start and end point and the associated space and including some methods to obtain and compare slope and distance to other segments.- List<Segment> segments: a list of all spaces' boundary segments.- Dictionary<Segment, Segment> segmentPairs: a dictionary mapping each segment to the closest other segment in the set.- Dictionary<Space, List<Space>> spaceAdjacencies: a dictionary mapping each space to a list of all other spaces directly adjacent to it.

This data is generated and processed step by step by the following methods:

GetBoundaries: determine all boundary segments for a given space. This method is applied in a loop to all selected spaces, or all spaces in the project if none were manually preselected.- FindClosestSegments: iterate over the list of segments and determine the closest other segment for each one. This is not fool-proof, since it simply calculates the distance between the midpoints of each segment.- DetermineAdjacencies: determine the space adjacencies from the pairs of closest boundary segments using the Space.IsPointInSpace method.- ReportAdjacencies: print the results to the Visual Studio debug output console.

Here is the Segment class implementation:

```python
class Segment
{
  XYZ \_sp;
  XYZ \_ep;
  Space \_space;

  public XYZ StartPoint
  {
    get { return \_sp; }
  }

  public XYZ EndPoint
  {
    get { return \_ep; }
  }

  public Space Space
  {
    get { return \_space; }
    set { \_space = value; }
  }

  public Segment( XYZ sp, XYZ ep, Space space )
  {
    \_sp = sp;
    \_ep = ep;
    \_space = space;
  }

  public double Slope
  {
    get
    {
      double deltaX = \_sp.X - \_ep.X;
      double deltaY = \_sp.Y - \_ep.Y;
      if( deltaX != 0 )
      {
        return deltaY / deltaX;
      }
      return 0;
    }
  }

  public bool IsHorizontal
  {
    get
    {
      return \_sp.Y == \_ep.Y;
    }
  }

  public bool IsVertical
  {
    get
    {
      return \_sp.X == \_ep.X;
    }
  }

  public new string ToString()
  {
    return string.Format( "{0} {1}",
      Util.PointString( \_sp ),
      Util.PointString( \_ep ) );
  }

  public XYZ MidPoint
  {
    get
    {
      return \_sp + 0.5 \* ( \_ep - \_sp );
    }
  }

  public XYZ DirectionTo( Segment a )
  {
    XYZ v = a.MidPoint - MidPoint;
    return v.IsZero ? v : v.Normalized;
  }

  public double Distance( Segment a )
  {
    return MidPoint.Distance( a.MidPoint );
  }

  public bool Parallel( Segment a )
  {
    return ( IsVertical && a.IsVertical )
      || ( IsHorizontal && a.IsHorizontal )
      || Util.IsEqual( Slope, a.Slope );
  }
}
```

GetBoundaries simply asks a space for its boundary curves, tessellates them, and generates Segment instances for the result:

```csharp
private void GetBoundaries(
  List<Segment> segments,
  Space space )
{
  BoundarySegmentArrayArray boundaries
    = space.Boundary;

  foreach( BoundarySegmentArray b in boundaries )
  {
    foreach( BoundarySegment s in b )
    {
      Curve curve = s.Curve;
      XYZArray a = curve.Tessellate();
      for( int i = 1; i < a.Size; i++ )
      {
        Segment segment = new Segment(
          a.get\_Item( i - 1 ),
          a.get\_Item( i ), space );

        segments.Add( segment );
      }
    }
  }
}
```

FindClosestSegments iterates over the list of segments and determines the closest other segment for each one by comparing the distance between their midpoints, returning The resulting closest pairs:

```csharp
private void FindClosestSegments(
  Dictionary<Segment, Segment> segmentPairs,
  List<Segment> segments )
{
  foreach( Segment segOuter in segments )
  {
    bool first = true;
    double dist = 0;
    Segment closest = null;

    foreach( Segment segInner in segments )
    {
      if( segOuter == segInner )
        continue;

      if( segInner.Space == segOuter.Space )
        continue;

      double d = segOuter.Distance(
        segInner );

      if( first || d < dist )
      {
        dist = d;
        first = false;
        closest = segInner;
      }
    }

    segmentPairs.Add( segOuter, closest );
  }
}
```

DetermineAdjacencies determines the space adjacencies from the pairs of closest boundary segments using the Space.IsPointInSpace method.
It analyses the relationship between the two closest segments s and t.
If their distance exceeds the maximum wall thickness, the spaces are not considered adjacent.
Otherwise, a test point two millimetres away from s in the direction of t is calculated and the Space.IsPointInSpace method applied to it to test whether it really lies within the candidate neighbouring space:

```csharp
private void DetermineAdjacencies(
  Dictionary<Space, List<Space>> a,
  Dictionary<Segment, Segment> segmentPairs )
{
  foreach( Segment s in segmentPairs.Keys )
  {
    Segment t = segmentPairs[s];
    double d = s.Distance( t );
    if( d < MaxWallThickness )
    {
      XYZ direction = s.DirectionTo( t );
      XYZ startPt = t.MidPoint;
      XYZ testPoint = startPt + direction \* D2mm;
      if( t.Space.IsPointInSpace( testPoint ) )
      {
        if( !a.ContainsKey( s.Space ) )
        {
          a.Add( s.Space, new List<Space>() );
        }
        if( !a[s.Space].Contains( t.Space ) )
        {
          a[s.Space].Add( t.Space );
        }
      }
    }
  }
}
```

Finally, ReportAdjacencies prints the results to the Visual Studio debug output console:

```csharp
private void PrintSpaceInfo(
  string indent,
  Space space )
{
  Debug.Print( "{0}{1} {2}", indent,
    space.Name, space.Number );
}

private void ReportAdjacencies(
  Dictionary<Space, List<Space>> spaceAdjacencies )
{
  Debug.WriteLine( "\nReport Space Adjacencies:" );
  foreach( Space space in spaceAdjacencies.Keys )
  {
    PrintSpaceInfo( "", space );
    foreach( Space adj in spaceAdjacencies[space] )
    {
      PrintSpaceInfo( "  ", adj );
    }
  }
}
```

Here is the code of the Execute method which performs these steps in sequence:

```csharp
Application app = commandData.Application;
Document doc = app.ActiveDocument;

List<Element> spaces = new List<Element>();
if( !Util.GetSelectedElementsOrAll(
  spaces, doc, typeof( Space ) ) )
{
  Selection sel = doc.Selection;
  message = (0 < sel.Elements.Size)
    ? "Please select some space elements."
    : "No space elements found.";
  return CmdResult.Failed;
}

List<Segment> segments = new List<Segment>();

foreach( Space space in spaces )
{
  GetBoundaries( segments, space );
}

Dictionary<Segment, Segment> segmentPairs
  = new Dictionary<Segment, Segment>();

FindClosestSegments( segmentPairs, segments );

Dictionary<Space, List<Space>> spaceAdjacencies
  = new Dictionary<Space, List<Space>>();

DetermineAdjacencies(
  spaceAdjacencies, segmentPairs );

ReportAdjacencies( spaceAdjacencies );

return CmdResult.Failed;
```

Here is the result of running the new command CmdSpaceAdjacency on the sample shown above:

```
Report Space Adjacencies:

Space 1 1
  Space 2 2
Space 2 2
  Space 4 4
  Space 3 3
  Space 1 1
Space 3 3
  Space 2 2
  Space 4 4
Space 4 4
  Space 2 2
  Space 5 5
  Space 3 3
Space 5 5
  Space 4 4
```

Here is
[version 1.1.0.38](zip/bc11038.zip)
of the complete Visual Studio solution with the new command.

As Martin pointed out above, this sample provides a solution for one specific case.
It may not be totally reliable under all circumstances, and as Martin already discovered, many other approaches and different requirements for space adjacency analysis may occur.
It does however provide a wonderful example of what can be achieved with relatively little effort.
And as said, it also shows a really nice use of the new Space.IsPointInSpace method.

Very many thanks to Martin for providing this interesting sample!