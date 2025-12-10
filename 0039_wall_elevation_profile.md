---
post_number: "0039"
title: "Wall Elevation Profile"
slug: "wall_elevation_profile"
author: "Jeremy Tammik"
tags: ['csharp', 'elements', 'geometry', 'revit-api', 'selection', 'walls']
source_file: "0039_wall_elevation_profile.htm"
original_url: "https://thebuildingcoder.typepad.com/blog/0039_wall_elevation_profile.html"
---

### Wall Elevation Profile

In this post, we explore how to determine the wall elevation boundary polygons, similarly to the
[floor boundary polygon](http://thebuildingcoder.typepad.com/blog/2008/10/slab-boundary.html)
algorithm.
This functionality is frequently requested, last but not least in a
[comment by Art](http://thebuildingcoder.typepad.com/blog/2008/10/slab-boundary.html#comments).

The approach is similar to the one for the floor boundary polygons, with some small additional twists.
To determine the floor boundary, we simply searched for one of its two horizontal faces and queried it for its edge loops.
For a wall, it is slightly more complicated to decide which face we want to query, since we cannot expect it to be aligned with any of the cardinal coordinate axes.
What we can expect, however, for simple walls, is that the normal vector of the face we are interested in is perpendicular to both the wall location line and the Z axis.
Then, we just have to decide whether we are interested in the face on the interior or exterior wall side.
The preceding post on the
[wall layers](http://thebuildingcoder.typepad.com/blog/2008/11/wall-compound-layers.html)
discussed some analysis concerning the wall location line and its Flipped property to help determine which face is interior and exterior.

The implementation is similar to the floor boundary one and has the following main steps:

- Select wall elements to process.
- Query each wall for its location curve and solid.
- From these, determine its exterior face.
- Query the face for its edge loops and collect the polygon data from them.
- Draw model lines representing the boundary loops, offset from the wall by one foot.

Here is an example of running this algorithm on a couple of walls, showing the original walls and the resulting model lines representing the boundary loops, offset by one foot outwards from their exterior faces, and including the interior hole loops:

![Wall elevation profile boundary loops](img/wall_profiles.png)

Determining the exterior face from the solid and assembling the polygon data from its edge loops is performed by the GetProfile helper method. It also includes some debugging sanity checks to ensure that the loops obtained are in fact closed loops, i.e. that subsequent edges join and the last vertex equals the first:

```csharp
const double \_offset = 1.0;
bool GetProfile(
  List<List<XYZ>> polygons,
  Solid solid,
  XYZ v,
  XYZ w )
{
  double d, dmax = 0;
  PlanarFace outermost = null;
  FaceArray faces = solid.Faces;
  foreach( Face f in faces )
  {
    PlanarFace pf = f as PlanarFace;
    if( null != pf
      && Util.IsVertical( pf )
      && Util.IsZero( v.Dot( pf.Normal ) ) )
    {
      d = pf.Origin.Dot( w );
      if( ( null == outermost )
        || ( dmax < d ) )
      {
        outermost = pf;
        dmax = d;
      }
    }
  }
  if( null != outermost )
  {
    XYZ voffset = \_offset \* w;
    XYZ p, q = XYZ.Zero;
    bool first;
    int i, n;
    EdgeArrayArray loops = outermost.EdgeLoops;
    foreach( EdgeArray loop in loops )
    {
      List<XYZ> vertices = new List<XYZ>();
      first = true;
      foreach( Edge e in loop )
      {
        XYZArray points = e.Tessellate();
        p = points.get\_Item( 0 );
        if( !first )
        {
          Debug.Assert( p.AlmostEqual( q ),
            "expected subsequent start point"
            + " to equal previous end point" );
        }
        n = points.Size;
        q = points.get\_Item( n - 1 );
        for( i = 0; i < n - 1; ++i )
        {
          XYZ a = points.get\_Item( i );
          a += voffset;
          vertices.Add( a );
        }
      }
      q += voffset;
      Debug.Assert( q.AlmostEqual( vertices[0] ),
        "expected last end point to equal"
        + " first start point" );
      polygons.Add( vertices );
    }
  }
  return null != outermost;
}
```

Here is the CmdWallProfile mainline source code, making the call to GetProfile:

```csharp
public CmdResult Execute(
  ExternalCommandData commandData,
  ref string message,
  ElementSet elements )
{
  Application app = commandData.Application;
  Document doc = app.ActiveDocument;

  List<RvtElement> walls = new List<RvtElement>();
  if( !Util.GetSelectedElementsOrAll(
    walls, doc, typeof( Wall ) ) )
  {
    Selection sel = doc.Selection;
    message = ( 0 < sel.Elements.Size )
      ? "Please select some wall elements."
      : "No wall elements found.";
    return CmdResult.Failed;
  }

  XYZ p, q, v, w;
  List<List<XYZ>> polygons = new List<List<XYZ>>();
  Options opt = app.Create.NewGeometryOptions();

  foreach( Wall wall in walls )
  {
    string desc = Util.ElementDescription( wall );

    LocationCurve curve
      = wall.Location as LocationCurve;

    if( null == curve )
    {
      message = desc + ": No wall curve found.";
      return CmdResult.Failed;
    }
    p = curve.Curve.get\_EndPoint( 0 );
    q = curve.Curve.get\_EndPoint( 1 );
    v = q - p;
    v = v.Normalized;
    w = XYZ.BasisZ.Cross( v ).Normalized;
    if( wall.Flipped ) { w = -w; }

    GeoElement geo = wall.get\_Geometry( opt );
    GeometryObjectArray objects = geo.Objects;
    foreach( GeometryObject obj in objects )
    {
      Solid solid = obj as Solid;
      if( solid != null )
      {
        GetProfile( polygons, solid, v, w );
      }
    }
  }

  int n = polygons.Count;

  Debug.WriteLine( string.Format(
    "{0} boundary loop{1} found.",
    n, Util.PluralSuffix( n ) ) );

  Creator creator = new Creator( app );
  creator.DrawPolygons( polygons );

  return CmdResult.Succeeded;
}
```

Here is an updated
[version 1.0.0.13](http://thebuildingcoder.typepad.com/blog/files/bc10013.zip)
of the complete Visual Studio solution,
including the new CmdWallProfile and all other commands discussed so far.