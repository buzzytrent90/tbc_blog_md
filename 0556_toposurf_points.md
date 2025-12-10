---
post_number: "0556"
title: "Toposurface Interior and Boundary Points"
slug: "toposurf_points"
author: "Jeremy Tammik"
tags: ['csharp', 'elements', 'python', 'revit-api', 'windows']
source_file: "0556_toposurf_points.htm"
original_url: "https://thebuildingcoder.typepad.com/blog/0556_toposurf_points.html"
---

### Toposurface Interior and Boundary Points

I arrived here in
[Jeddah](http://en.wikipedia.org/wiki/Jeddah) last night.
I have not seen anything of it at all yet except through the hotel window, but I will go out exploring today.
The Revit API training I am here to give begins on Sunday.
The weekend here is on Thursday and Friday, so today is the holiest day of the week.

Meanwhile, here is a recent question that gave me a very welcome chance to try my hand at implementing a little algorithm which requires at least a little bit of geometrical and logical analysis, something I love but unfortunately don't get to do as much as I would like.

**Question:** I can access the points of the toposurface.
How can I determine which of them are inner and which are boundary points?

Here is the lower left corner boundary point of a surface:

![Toposurface boundary point](img/toposurface_boundary_point.png)

Here is its middle interior point:

![Toposurface interior point](img/toposurface_interior_point.png)

I tried to enumerate all the mesh triangles and determine whether a point is interior or boundary depending on how many triangles it belongs to, assuming if it just belongs to one or two it would be exterior and three interior, but that seems to be totally wrong.
Could you explain why, please?

Here is part of my testing code, but, as said, it does not work at all:
```csharp
/// <summary>
/// For each point of the given mesh,
/// determine how many triangles it belongs to.
/// </summary>
void DetermineTriangleCountForPoints( Mesh mesh )
{
  int np = mesh.Vertices.Count;
  int nt = mesh.NumTriangles;

  Debug.Print( "Mesh has {0} point{1} and {2} triangle{3}.",
    np, PluralSuffix( np ), nt, PluralSuffix( nt ) );

  MapVertexToTriangleIndices map
    = new MapVertexToTriangleIndices();

  for( int i = 0; i < nt; ++i )
  {
    MeshTriangle t = mesh.get\_Triangle( i );

    for( int j = 0; j < 3; ++j )
    {
      map.AddVertex( t.get\_Vertex( j ), i );
    }
  }
  List<XYZ> pts = new List<XYZ>( map.Keys );
  pts.Sort( Compare );

  foreach( XYZ p in pts )
  {
    int n = map[p].Count;

    Debug.Print( "  vertex {0} belongs to {1} triangle{2}",
      PointString( p ), n, PluralSuffix( n ) );
  }
}
```

Here is the result of running it on the surface above before I discovered that the interior versus boundary classification is completely wrong:
```csharp
TopographySurface 128689:
Mesh has 6 points and 5 triangles.
(-35.952648163, -68.290878296, 0.000000000)
belongs to 3 triangles and is therefore interior
(-34.969902039, -57.134258270, 0.000000000)
belongs to 1 triangle and is therefore exterior
(-24.212404251, -76.898384094, 0.000000000)
belongs to 2 triangles and is therefore exterior
(-23.932262421, -60.340045929, 0.000000000)
belongs to 3 triangles and is therefore interior
(-23.100381851, -69.003776550, 0.000000000)
belongs to 4 triangles and is therefore exterior
(-14.513459206, -65.842926025, 0.000000000)
belongs to 2 triangles and is therefore exterior
```

Could you please suggest a working method to distinguish boundary and interior toposurface mesh points?

As Revit tells me whether a point is boundary or interior in the user interface, where does it get this info from?

By the way, is there any way to display the mesh triangles and possibly even their indices in the Revit UI?

**Answer:** There is currently no built-in Revit API functionality that will provide this info.
We are aware of a few gaps in the current toposurface API.
Here is a host of public
[convex hull algorithms](http://en.wikipedia.org/wiki/Convex_hull_algorithms) that
you could try, though.

Remember, Z should be ignorable, if you are only looking for boundaries it's really about XY location
for a toposurface.

Revit's internal functions to determine the boundary of a mesh are based on the following algorithm idea:

In order to identify boundary points, one has to first identify boundary edges.
A boundary edge is an edge belonging to only one triangle.
All other (interior) edges are shared by two triangles.
Determining if an edge is shared by one or two triangles is a bit tricky.

I implemented an algorithm based on this idea, and it seems to work fine.
I worked on it through several stages.

In a first step, I determine the triangle count for each mesh vertex just as you did above in the implementation of DetermineTriangleCountForPoints.
As you discovered, this information is insufficient to classify the point into boundary or interior.

In a second step, I determine whether an edge is an interior edge or not, implemented in the method DetermineTriangleCountForEdges.

This works reliably, and just needs one further step to classify the points.

#### Identifying Points

At this point, I changed something else as well, though.
I was initially identifying the points and edges using their coordinate values.
Before proceeding to the third stage, I decided to identify them by the vertex indices of the points in the list of mesh vertices instead.
It is obviously much more reliable and efficient to identify the points by one single integer value rather than a fuzzy comparison of three real values.

For example, DetermineTriangleCountForEdges is based on a helper class to represent the triangle edges
which stores the complete XYZ start and end point data of each edge:
```csharp
public class MeshTriangleEdge
{
  public XYZ A { get; set; }
  public XYZ B { get; set; }

  public MeshTriangleEdge( XYZ a, XYZ b )
  {
    int d = Compare( a, b );

    Debug.Assert( 0 != d, "expected non-equal edge vertices" );

    A = ( 0 < d ) ? a : b;
    B = ( 0 < d ) ? b : a;
  }
}
```

After deciding that a method storing the edges based on the mesh vertex index of their start and end points instead of the XYZ coordinates is more efficient and reliable, I switched to that representation in the third step using the following class to represent the edges:
```csharp
/// <summary>
/// Manage a mesh triangle edge by storing the
/// index of the mesh vertex corresponing to
/// the edge start and end point.
/// For reliable comparison purposes, the
/// lower index is always stored in A and
/// the higher in B.
/// </summary>
public class JtEdge
{
  public int A { get; set; }
  public int B { get; set; }

  public JtEdge( int a, int b )
  {
    Debug.Assert( a != b, "expected non-equal edge vertices" );

    A = ( a < b ) ? a : b;
    B = ( a < b ) ? b : a;
  }
}
```

#### Classifying Points

The third step consists in determining whether a point is interior or boundary, implemented in the method ClassifyPoints. A point is interior if all of the edges it belongs to also are. If one single edge is a boundary, then so is the point.

I make use of a couple of helper classes implementing dictionaries to simplify the mainline of the code.
One is MapEdgeToTriangles, which maps a mesh triangle edge to a list of the indices of all the triangles it belongs to:
```csharp
/// <summary>
/// Map mesh triangle edges to a list of the
/// indices of all the triangles they belong to.
/// </summary>
class MapEdgeToTriangles
  : Dictionary<JtEdge, List<int>>
{
  public MapEdgeToTriangles()
    : base( new JtEdgeEqualityComparer() )
  {
  }

  /// <summary>
  /// Add a new edge.
  /// If it is already known, append the triangle
  /// index of the current triangle. Otherwise,
  /// generate a new key for it.
  /// </summary>
  public void AddEdge(
    JtEdge e,
    int triangleIndex )
  {
    if( !ContainsKey( e ) )
    {
      Add( e, new List<int>( 2 ) );
    }
    ( this )[e].Add( triangleIndex );
  }
}
```

Another is MapVertexToEdges, mapping a mesh vertex index to a list of the edges the vertex belongs to.
```csharp
/// <summary>
/// Map mesh vertex index to a list of the edges
/// the vertex belongs to.
/// </summary>
class MapVertexToEdges
  : Dictionary<int, List<JtEdge>>
{
  public MapVertexToEdges( int capacity )
    : base( capacity )
  {
  }

  /// <summary>
  /// Append a new edge for a given vertex.
  /// If the vertex is new, generate a new key for it.
  /// </summary>
  public void AddVertexEdge(
    int vertexIndex,
    JtEdge e )
  {
    if( !ContainsKey( vertexIndex ) )
    {
      Add( vertexIndex, new List<JtEdge>( 2 ) );
    }
    ( this )[vertexIndex].Add( e );
  }
}
```

Both of these basically just provide an enhanced method for adding a new element to the dictionary.

Here is the implementation of the main algorithm:
```python
/// <summary>
/// For each point of the given mesh,
/// determine whether it is interior or boundary.
/// The algorithm goes like this:
/// Every triangle edge belongs to either one or two triangles,
/// depending on whether it it boundary or interior.
/// For each edge, determine whether it is boundary
/// or interior. A point is interior if all of the edges it
/// belongs to are interior.
/// </summary>
/// <returns>A dictionary mapping each mesh vertex index
/// to a Boolean which is true if the corresponding point
/// is interior and false if it is on the boundary</returns>
Dictionary<int, bool> ClassifyPoints( Mesh mesh )
{
  int nv = mesh.Vertices.Count;
  int nt = mesh.NumTriangles;
  int i, n;

  Debug.Print( "\nClassifyPoints: mesh has {0} point{1} and {2} triangle{3}:",
    nv, PluralSuffix( nv ), nt, PluralSuffix( nt ) );

  // set up a map to determine the vertex
  // index of a given triangle vertex;
  // this is needed because the
  // MeshTriangle.get\_Vertex method
  // returns the XYZ but not the index,
  // and we base our edges on the index:

  Dictionary<XYZ, int> vertexIndex
    = new Dictionary<XYZ, int>( nv, new XyzEqualityComparer() );

  for( i = 0; i < nv; ++i )
  {
    XYZ p = mesh.Vertices[i];
    Debug.Print( "  mesh vertex {0}: {1}", i, PointString( p ) );
    vertexIndex[p] = i;
  }

  // set up a map to determine which
  // edges a given vertex belongs to:

  MapVertexToEdges vertexEdges
    = new MapVertexToEdges( nv );

  // set up a map to determine which
  // triangles a given edge belongs to;
  // this is used to determine the edge's
  // interior or boundary status:

  MapEdgeToTriangles map
    = new MapEdgeToTriangles();

  for( i = 0; i < nt; ++i )
  {
    MeshTriangle t = mesh.get\_Triangle( i );

    for( int j = 0; j < 3; ++j )
    {
      // get the start and end vertex
      // of the current triangle edge:

      int a = vertexIndex[t.get\_Vertex( 0 == j ? 2 : j - 1 )];
      int b = vertexIndex[t.get\_Vertex( j )];

      JtEdge e = new JtEdge( a, b );

      map.AddEdge( e, i );

      vertexEdges.AddVertexEdge( a, e );
      vertexEdges.AddVertexEdge( b, e );
    }
  }

  int nBoundaryEdges;
  int nInteriorPoints = 0;

  Dictionary<int, bool> dict = new Dictionary<int,bool>( nv );

  Debug.Print( "Classify the {0} point{1}:", nv, PluralSuffix( nv ) );

  for( i = 0; i < nv; ++i )
  {
    nBoundaryEdges = 0;
    n = vertexEdges[i].Count;

    nBoundaryEdges = vertexEdges[i].Count<JtEdge>(
      e => 1 == map[e].Count );

    dict[i] = ( 0 == nBoundaryEdges );

    XYZ p = mesh.Vertices[i];

    if( 0 == nBoundaryEdges )
    {
      ++nInteriorPoints;
    }

    Debug.Print( "  point {0} {1} belongs to {2} edge{3}, "
      + "{4} interior and {5} boundary and is therefore {6}",
      i, PointString( p ), n, PluralSuffix( n ),
      n - nBoundaryEdges, nBoundaryEdges,
      ( 0 == nBoundaryEdges ? "interior" : "boundary" ) );
  }
  Debug.Print( "{0} boundary and {1} interior points detected.",
    nv - nInteriorPoints, nInteriorPoints );

  return dict;
}
```

I tested this on the following topo surfaces of increasing complexity:

- a triangle,- a quadrilateral with four boundary points,- another quadrilateral with an additional interior point, and- a larger surface with 26 points

They look like this in Revit:

![Toposurfaces](img/toposurfaces.png)

Here are the results (please copy to a text editor to see the full untruncated lines):
```csharp
TopographySurface 128701:
ClassifyPoints: mesh has 3 points and 1 triangle:
mesh vertex 0: (-14.39,11.53,0)
mesh vertex 1: (2.81,12.55,0)
mesh vertex 2: (-7.4,22.76,0)
triangle 0 vertex 0: (-14.39,11.53,0)
triangle 0 vertex 1: (2.81,12.55,0)
triangle 0 vertex 2: (-7.4,22.76,0)
Classify the 3 edges:
edge 0->1 belongs to 1 triangle and is therefore boundary
edge 0->2 belongs to 1 triangle and is therefore boundary
edge 1->2 belongs to 1 triangle and is therefore boundary
Classify the 3 points:
point 0 (-14.39,11.53,0) belongs to 2 edges, 0 interior and 2 boundary and is therefore boundary
point 1 (2.81,12.55,0) belongs to 2 edges, 0 interior and 2 boundary and is therefore boundary
point 2 (-7.4,22.76,0) belongs to 2 edges, 0 interior and 2 boundary and is therefore boundary
3 boundary and 0 interior points detected.
TopographySurface 128712:
ClassifyPoints: mesh has 4 points and 2 triangles:
mesh vertex 0: (25.45,10.77,0)
mesh vertex 1: (25.01,23.65,0)
mesh vertex 2: (38.32,11.22,0)
mesh vertex 3: (38.77,24.09,0)
triangle 0 vertex 0: (25.01,23.65,0)
triangle 0 vertex 1: (25.45,10.77,0)
triangle 0 vertex 2: (38.32,11.22,0)
triangle 1 vertex 0: (38.32,11.22,0)
triangle 1 vertex 1: (38.77,24.09,0)
triangle 1 vertex 2: (25.01,23.65,0)
Classify the 5 edges:
edge 0->1 belongs to 1 triangle and is therefore boundary
edge 0->2 belongs to 1 triangle and is therefore boundary
edge 1->2 belongs to 2 triangles and is therefore interior
edge 1->3 belongs to 1 triangle and is therefore boundary
edge 2->3 belongs to 1 triangle and is therefore boundary
Classify the 4 points:
point 0 (25.45,10.77,0) belongs to 2 edges, 0 interior and 2 boundary and is therefore boundary
point 1 (25.01,23.65,0) belongs to 4 edges, 2 interior and 2 boundary and is therefore boundary
point 2 (38.32,11.22,0) belongs to 4 edges, 2 interior and 2 boundary and is therefore boundary
point 3 (38.77,24.09,0) belongs to 2 edges, 0 interior and 2 boundary and is therefore boundary
4 boundary and 0 interior points detected.
TopographySurface 128718:
ClassifyPoints: mesh has 5 points and 4 triangles:
mesh vertex 0: (80.05,28.53,0)
mesh vertex 1: (63.18,39.63,0)
mesh vertex 2: (84.49,41.85,0)
mesh vertex 3: (75.61,52.06,0)
mesh vertex 4: (98.7,45.84,0)
triangle 0 vertex 0: (75.61,52.06,0)
triangle 0 vertex 1: (63.18,39.63,0)
triangle 0 vertex 2: (84.49,41.85,0)
triangle 1 vertex 0: (80.05,28.53,0)
triangle 1 vertex 1: (98.7,45.84,0)
triangle 1 vertex 2: (84.49,41.85,0)
triangle 2 vertex 0: (84.49,41.85,0)
triangle 2 vertex 1: (98.7,45.84,0)
triangle 2 vertex 2: (75.61,52.06,0)
triangle 3 vertex 0: (80.05,28.53,0)
triangle 3 vertex 1: (84.49,41.85,0)
triangle 3 vertex 2: (63.18,39.63,0)
Classify the 8 edges:
edge 0->1 belongs to 1 triangle and is therefore boundary
edge 0->2 belongs to 2 triangles and is therefore interior
edge 0->4 belongs to 1 triangle and is therefore boundary
edge 1->2 belongs to 2 triangles and is therefore interior
edge 1->3 belongs to 1 triangle and is therefore boundary
edge 2->3 belongs to 2 triangles and is therefore interior
edge 2->4 belongs to 2 triangles and is therefore interior
edge 3->4 belongs to 1 triangle and is therefore boundary
Classify the 5 points:
point 0 (80.05,28.53,0) belongs to 4 edges, 2 interior and 2 boundary and is therefore boundary
point 1 (63.18,39.63,0) belongs to 4 edges, 2 interior and 2 boundary and is therefore boundary
point 2 (84.49,41.85,0) belongs to 8 edges, 8 interior and 0 boundary and is therefore interior
point 3 (75.61,52.06,0) belongs to 4 edges, 2 interior and 2 boundary and is therefore boundary
point 4 (98.7,45.84,0) belongs to 4 edges, 2 interior and 2 boundary and is therefore boundary
4 boundary and 1 interior points detected.
TopographySurface 128725:
ClassifyPoints: mesh has 26 points and 39 triangles:
mesh vertex 0: (-87.01,-44.72,0)
mesh vertex 1: (-75.47,-26.96,0)
. . .
mesh vertex 25: (-83.46,-24.3,0)
triangle 0 vertex 0: (-87.01,-44.72,0)
triangle 0 vertex 1: (-77.69,-55.82,0)
. . .
triangle 38 vertex 2: (-75.47,-26.96,0)
Classify the 64 edges:
edge 0->8 belongs to 1 triangle and is therefore boundary
edge 0->11 belongs to 1 triangle and is therefore boundary
. . .
edge 24->25 belongs to 1 triangle and is therefore boundary
Classify the 26 points:
point 0 (-87.01,-44.72,0) belongs to 6 edges, 4 interior and 2 boundary and is therefore boundary
point 1 (-75.47,-26.96,0) belongs to 10 edges, 10 interior and 0 boundary and is therefore interior
point 2 (-55.49,-19.86,0) belongs to 10 edges, 10 interior and 0 boundary and is therefore interior
point 3 (-53.71,-33.18,0) belongs to 12 edges, 12 interior and 0 boundary and is therefore interior
point 4 (-43.06,-26.96,0) belongs to 8 edges, 6 interior and 2 boundary and is therefore boundary
point 5 (-43.06,-42.5,0) belongs to 10 edges, 10 interior and 0 boundary and is therefore interior
point 6 (-65.26,-35.4,0) belongs to 12 edges, 12 interior and 0 boundary and is therefore interior
point 7 (-60.37,-53.15,0) belongs to 12 edges, 12 interior and 0 boundary and is therefore interior
point 8 (-77.69,-55.82,0) belongs to 8 edges, 6 interior and 2 boundary and is therefore boundary
point 9 (-68.36,-43.83,0) belongs to 14 edges, 14 interior and 0 boundary and is therefore interior
point 10 (-41.28,-54.49,0) belongs to 8 edges, 6 interior and 2 boundary and is therefore boundary
point 11 (-88.34,-33.62,0) belongs to 4 edges, 2 interior and 2 boundary and is therefore boundary
point 12 (-66.14,-61.14,0) belongs to 6 edges, 4 interior and 2 boundary and is therefore boundary
point 13 (-79.02,-44.28,0) belongs to 10 edges, 10 interior and 0 boundary and is therefore interior
point 14 (-81.68,-32.73,0) belongs to 14 edges, 14 interior and 0 boundary and is therefore interior
point 15 (-63.93,-26.96,0) belongs to 14 edges, 14 interior and 0 boundary and is therefore interior
point 16 (-55.05,-43.39,0) belongs to 12 edges, 12 interior and 0 boundary and is therefore interior
point 17 (-47.5,-50.93,0) belongs to 8 edges, 8 interior and 0 boundary and is therefore interior
point 18 (-74.58,-50.93,0) belongs to 8 edges, 8 interior and 0 boundary and is therefore interior
point 19 (-68.81,-54.93,0) belongs to 10 edges, 10 interior and 0 boundary and is therefore interior
point 20 (-53.71,-26.07,0) belongs to 10 edges, 10 interior and 0 boundary and is therefore interior
point 21 (-47.06,-18.97,0) belongs to 6 edges, 4 interior and 2 boundary and is therefore boundary
point 22 (-49.72,-14.98,0) belongs to 4 edges, 2 interior and 2 boundary and is therefore boundary
point 23 (-67.03,-17.64,0) belongs to 6 edges, 4 interior and 2 boundary and is therefore boundary
point 24 (-74.58,-19.42,0) belongs to 6 edges, 4 interior and 2 boundary and is therefore boundary
point 25 (-83.46,-24.3,0) belongs to 6 edges, 4 interior and 2 boundary and is therefore boundary
11 boundary and 15 interior points detected.
```

As far as I can tell, this works efficiently and reliably.

Please let me know what experiences you have with this and whether it solves the problem for you. Thank you!

Here is
[TopoSurfacePointClassify.zip](zip/TopoSurfacePointClassify.zip) containing
the entire source code and Visual Studio solution implementing this add-in.