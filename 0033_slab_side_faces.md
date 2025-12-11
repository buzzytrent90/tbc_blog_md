---
ai_optimized: true
author: Jeremy Tammik
complexity_score: 4.1
content_type: qa
optimization_date: '2025-12-11T11:44:13.263750'
original_url: https://thebuildingcoder.typepad.com/blog/0033_slab_side_faces.html
post_number: '0033'
reading_time_minutes: 5
series: general
slug: slab_side_faces
source_file: 0033_slab_side_faces.htm
tags:
- csharp
- elements
- geometry
- revit-api
- selection
title: Slab Side Faces
word_count: 952
---

### Slab Side Faces

In the
[preceding post](http://thebuildingcoder.typepad.com/blog/2008/11/model-line-creation.html),
we discussed the creation of model lines to graphically display certain polygons in space for debugging purposes.
The polygons we display are
[slab boundaries](http://thebuildingcoder.typepad.com/blog/2008/10/slab-boundary.html)
calculated by iterating over all the horizontal faces of the slab and choosing the lowest one to extract its edge loops from, which represent the polygonal slab boundary loops.
In this post, we demonstrate another use of this model line creator.
We expand it to calculate and display the normal vectors of all the triangles of a face mesh triangulation.
This enables us to better understand the faces that we are intersted in.
We make use of it to explore a slightly different approach for determining the slab boundary. This time around, we identify all vertical faces of the slab solid instead of the horizontal ones. They represent the two-dimensional 'side' faces. We highlight them by triangulating each one and adding little model lines representing the face normals of each triangle. This is accomplished by a new method DrawFaceTriangleNormals() added the Creator class.

I have implemented an external command CmdSlabSides realising this algorithm. It analyses the solid of a floor slab, just like CmdSlabBoundary, and extracts a list of all vertical faces, i.e. all side faces of the horizontal slab. CmdSlabBoundary only analyses planar faces. If we did so here as well, the face of the cylindrical opening would be ignored, since its face not planar. Therefore, we implemented a new overloaded version of IsVertical() in the Util class for cylindrical faces. Whereas the version for planar faces checks the normal vector of the face, the test for cylindrical faces makes use of the cylinder axis instead. If you need support for other types, you will have to add that yourself.

After the vertical side faces have been determined, we go through them all, triangulate each one, and draw a normal vector in the centre of each triangle. The normal vectors are calculated and added to the Revit model as model lines by the new method DrawFaceTriangleNormals( Face f ) added to the Creator class.

Here is the code for the mainline of the command.
The user can either select a set of specific floors to process, or all floors found in the database are used.
The code to create this set of floors, originally presented in CmdSlabBoundary, is now packaged in the GetSelectedElementsOrAll() utility method.
For each floor, the geometry is extracted and the solid is passed into the GetSideFaces() method for analysis:

```csharp
public CmdResult Execute(
  ExternalCommandData commandData,
  ref string message,
  ElementSet elements )
{
  Application app = commandData.Application;
  Document doc = app.ActiveDocument;

  List<RvtElement> floors = new List<RvtElement>();
  if( !Util.GetSelectedElementsOrAll(
    floors, doc, typeof( Floor ) ) )
  {
    Selection sel = doc.Selection;
    message = ( 0 < sel.Elements.Size )
      ? "Please select some floor elements."
      : "No floor elements found.";
    return CmdResult.Failed;
  }

  List<Face> faces = new List<Face>();
  Options opt = app.Create.NewGeometryOptions();

  foreach( Floor floor in floors )
  {
    GeoElement geo = floor.get\_Geometry( opt );
    GeometryObjectArray objects = geo.Objects;
    foreach( GeometryObject obj in objects )
    {
      Solid solid = obj as Solid;
      if( solid != null )
      {
        GetSideFaces( faces, solid );
      }
    }
  }

  int n = faces.Count;

  Debug.WriteLine( string.Format(
    "{0} side face{1} found.",
    n, Util.PluralSuffix( n ) ) );

  Creator creator = new Creator( app );
  foreach( Face f in faces )
  {
    creator.DrawFaceTriangleNormals( f );
  }
  return CmdResult.Succeeded;
}
```

Here is the straight-forward implementation of GetSideFaces(); please note that it currently only supports planar and cylindrical faces:

```csharp
void GetSideFaces(
  List<Face> verticalFaces,
  Solid solid )
{
  FaceArray faces = solid.Faces;
  foreach( Face f in faces )
  {
    if( f is PlanarFace )
    {
      if( Util.IsVertical( f as PlanarFace ) )
      {
        verticalFaces.Add( f );
      }
    }
    if( f is CylindricalFace )
    {
      if( Util.IsVertical( f as CylindricalFace ) )
      {
        verticalFaces.Add( f );
      }
    }
  }
}
```

And finally, here is the new method added of the Creator class:

```csharp
public void DrawFaceTriangleNormals( Face f )
{
  Mesh mesh = f.Triangulate();
  int n = mesh.NumTriangles;

  string s = "{0} face triangulation returns "
    + "mesh triangle{1} and normal vector{1}:";

  Debug.WriteLine( string.Format(
    s, n, Util.PluralSuffix( n ) ) );

  for( int i = 0; i < n; ++i )
  {
    MeshTriangle t = mesh.get\_Triangle( i );

    XYZ p = ( t.get\_Vertex( 0 )
      + t.get\_Vertex( 1 )
      + t.get\_Vertex( 2 ) ) / 3;

    XYZ v = t.get\_Vertex( 1 )
      - t.get\_Vertex( 0 );

    XYZ w = t.get\_Vertex( 2 )
      - t.get\_Vertex( 0 );

    XYZ normal = v.Cross( w ).Normalized;

    Debug.WriteLine( string.Format(
      "{0} {1} --> {2}", i,
      Util.PointString( p ),
      Util.PointString( normal ) ) );

    CreateModelLine( p, p + normal );
  }
}
```

A given face is triangulated to obtain a mesh. For each triangle in the mesh, the centre point p and two edge vectors v and w are determined. From these, the normal vector can be calculated, and a model line pointing out of the face in the middle of the triangle is generated.

Here is an example floor before processing:

![Original Slab](img/slab_boundary_4.png)

And here it is with the resulting model lines added after processing:

![Slab side face normals](img/slab_boundary_sides.png)

The little green lines are the face normals. Each rectangular face has two of them, since its triangulation returns two mesh triangles. Note that the circular hole is also handled successfully by this algorithm, although its face is not planar. Its mesh approximation obviously has many triangles. Also note that the triangle vertices are consistently oriented, so that the normal vector is always pointing out of the face, i.e. out of the solid that the face belongs to.

I am adding a new version 1.0.0.10 of the complete Visual Studio solution
[here](http://thebuildingcoder.typepad.com/blog/files/bc10010.zip),
including the new CmdSlabSides and updated Creator classes, as well as all other commands discussed so far.