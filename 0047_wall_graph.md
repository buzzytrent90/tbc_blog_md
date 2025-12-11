---
ai_optimized: true
author: Jeremy Tammik
complexity_score: 3.7
content_type: qa
optimization_date: '2025-12-11T11:44:13.283286'
original_url: https://thebuildingcoder.typepad.com/blog/0047_wall_graph.html
post_number: '0047'
reading_time_minutes: 4
series: general
slug: wall_graph
source_file: 0047_wall_graph.htm
tags:
- csharp
- elements
- filtering
- levels
- revit-api
- selection
- walls
title: Wall Graph
word_count: 762
---

### Wall Graph

How can I retrieve the wall graph from the Revit API?

I am trying to retrieve the wall connections or wall graph from a Revit drawing using the API.
In AutoCAD Architecture, this is exposed through the wall graph class.
Is there any corresponding object that I can use in Revit to give me this information?

The LocationCurve ElementsAtJoin property returns all the connected walls that join a wall at its two end points.
You can retrieve the wall's location curve though its Location property.
Here is some sample code implementing a new external command class CmdWallNeighbours that demonstrates this:

```csharp
public CmdResult Execute(
  ExternalCommandData commandData,
  ref string message,
  ElementSet elements )
{
  Application app = commandData.Application;
  Document doc = app.ActiveDocument;

  List<Element> walls = new List<Element>();
  if( !Util.GetSelectedElementsOrAll(
    walls, doc, typeof( Wall ) ) )
  {
    Selection sel = doc.Selection;
    message = ( 0 < sel.Elements.Size )
      ? "Please select some wall elements."
      : "No wall elements found.";
    return CmdResult.Failed;
  }

  int i, n;
  string desc, s = null;
  List<Element> neighbours;

  foreach( Wall wall in walls )
  {
    desc = Util.ElementDescription( wall );

    LocationCurve c
      = wall.Location as LocationCurve;

    if( null == c )
    {
      s = desc + ": No wall curve found.";
    }
    else
    {
      s = string.Empty;

      for( i = 0; i < 2; ++i )
      {
        neighbours = c.get\_ElementsAtJoin( i );
        n = neighbours.Count;

        s += string.Format(
          "\n\n{0} {1} point has {2} neighbour{3}{4}",
          desc,
          (0 == i ? "start" : "end"),
          n,
          Util.PluralSuffix( n ),
          Util.DotOrColon( n ) );

        foreach( Wall nb in neighbours )
        {
          s += "\n  " +
            Util.ElementDescription( nb );
        }
      }
    }
    Util.InfoMsg( s );
  }
  return CmdResult.Failed;
}
```

Unfortunately, if two walls intersect in the middle of one of them, there is no API call to retrieve the connected wall.

Thanks to Chris Arps of
[Ameri-CAD Inc.](http://www.americad.com)
for raising this issue, and Joe Ye of Autodesk for providing the answer.

Note that if a wall has at least one neighbour at one end, then the list of joined elements will include the wall itself as well, so that you get either zero neighbours or two or more, and one of them is the wall itself, which obviously can be filtered away from the list.

Here is the result of running this command on a small rectangular house with four walls:

```
Walls <127248 Jt> start point has 2 neighbours:
  Walls <127248 Jt>
  Walls <127251 Jt>

Walls <127248 Jt> end point has 2 neighbours:
  Walls <127248 Jt>
  Walls <127249 Jt>

Walls <127249 Jt> start point has 2 neighbours:
  Walls <127248 Jt>
  Walls <127249 Jt>

Walls <127249 Jt> end point has 2 neighbours:
  Walls <127249 Jt>
  Walls <127250 Jt>

Walls <127250 Jt> start point has 2 neighbours:
  Walls <127249 Jt>
  Walls <127250 Jt>

Walls <127250 Jt> end point has 2 neighbours:
  Walls <127250 Jt>
  Walls <127251 Jt>

Walls <127251 Jt> start point has 2 neighbours:
  Walls <127250 Jt>
  Walls <127251 Jt>

Walls <127251 Jt> end point has 2 neighbours:
  Walls <127248 Jt>
  Walls <127251 Jt>
```

Here is the result of running it on a set of only two walls joined at their common end point:

```
Walls <127876 Jt> start point has 0 neighbours.

Walls <127876 Jt> end point has 2 neighbours:
  Walls <127876 Jt>
  Walls <127933 Jt>

Walls <127933 Jt> start point has 2 neighbours:
  Walls <127876 Jt>
  Walls <127933 Jt>

Walls <127933 Jt> end point has 0 neighbours.
```

Here is an updated
[version 1.0.0.14](http://thebuildingcoder.typepad.com/blog/files/bc10014.zip)
of the complete Visual Studio solution,
including the new CmdWallNeighbours and all other commands discussed so far.

Note that this algorithm only reports the wall endpoint connections. If midpoint connections are required as well, some other algorithm will have to be used. The first idea that comes to mind is to retrieve all the wall center lines and treat is as a segment intersection problem in the plane: given a set S of n closed segments in the plane, report all intersection points among the segments in S. This doesn't seem like a challenging problem: one can take each pair of segments, compute whether they intersect, and, if so, report their intersection point. This brute force algorithm requires O(n^2) time. This is not a practical algorithm in real life. Wei Qian describes a more efficient
[sweep line algorithm for segment intersection](http://www.lems.brown.edu/~wq/projects/cs252.html)
which is O((n + i) log n), where i is the number of intersection points of segments in S. Obviously, one would have to sort the various wall center lines into different buckets for separate buildings and levels and possibly other relevant groupings before feeding them into such a algorithm.