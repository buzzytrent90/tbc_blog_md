---
ai_optimized: true
author: Jeremy Tammik
complexity_score: 7.6
content_type: code_example
optimization_date: '2025-12-11T11:44:14.602048'
original_url: https://thebuildingcoder.typepad.com/blog/0792_obj_export_v1.html
post_number: 0792
reading_time_minutes: 13
series: general
slug: obj_export_v1
source_file: 0792_obj_export_v1.htm
tags:
- csharp
- elements
- family
- filtering
- geometry
- python
- revit-api
- selection
- views
- walls
- windows
title: OBJ Model Exporter Take One
word_count: 2567
---

### OBJ Model Exporter Take One

Before continuing on the OBJ model exporter, I would like to mention I was finally up in the alps again for the first time in ages last weekend, to climb the
[Vrenelisgärtli](http://en.wikipedia.org/wiki/Gl%C3%A4rnisch) mountain in
[Glarus](http://en.wikipedia.org/wiki/Canton_of_Glarus), "Verena's Little Garden".

If you would like to skip straight to the meaty stuff, though, here is a table of contents:

- [AEC DevCamp 2012 material finalised](#2)- [Disk full](#3)- [Model up to cloud and down to mobile](#4)- [Face emitter interface](#5)- [OBJ exporter implementation](#6)- [Retrieving a solid from an element](#7)- [Exporting an element](#8)- [Exporting all collected elements](#9)- [Exporter mainline](#10)- [Export results](#11)

The mountain name is shrouded in legend and presumably originated in the cooling down of the Little Ice Age.
In the summer of 2003, the firn fields, which can be seen from far away, became snow-free for the first time again since then.
Right now they are covered again :-)

The weather was fantastic on the summit, although it looked as if all the rest of the country including all other mountains were covered in clouds.
Here is a view of the descent afterwards back down over the Glrnischfirn glacier
([more photos](https://plus.google.com/photos/104316998805199988071/albums/5757979017881867041?authkey=CJjvrdzk5LCbwgE)):

![Jeremy descending Glärnisch glacier from Vrenelisgärtli](file:////j/photo/jeremy/2012/2012-06-23_vrenelisgaertli/dscf5493_jeremy_walking_down_glacier_over_clouds.jpg)

#### AEC DevCamp 2012 Material Finalised

As Mikako Harada just
[pointed out](http://adndevblog.typepad.com/aec/2012/06/materials-from-aec-devcamp-2012.html),
the final version of the AEC DevCamp 2012 material is now available and has been relocated to its two final archival sites:

- For ADN members:
  [ADN extranet](http://adn.autodesk.com) >
  [Events](http://adn.autodesk.com/adn/servlet/index?siteID=4814862&id=5105692) >
  [Autodesk DevCamps 2012](http://adn.autodesk.com/adn/servlet/item?siteID=4814862&id=19908969)
- Public:
  [ADN Archive](http://www.adskconsulting.com/adn/cs/api_course_webcast_archive.php) >
  [AEC DevCamp - June 6, 2012](http://download.autodesk.com/media/adn/Autodesk_AEC_DevCamp_2012.zip)

For more information on the content, please refer to

- The [overview of the five tracks and the classes in each](http://thebuildingcoder.typepad.com/blog/2012/06/aec-devcamp-2012-material.html)- [Day one](http://thebuildingcoder.typepad.com/blog/2012/06/devcamp-day-one.html) of the Revit advanced track- Most of [day two](http://thebuildingcoder.typepad.com/blog/2012/06/devcamp-day-two.html) of the Revit advanced track- The [complete list of classes and course descriptions](http://adndevblog.typepad.com/files/aec-devcamp-2012---list-of-classes-final2-1.pdf)

I hope you will find this material useful.

Please note that the temporary DevCamp 2012 project site on Buzzsaw which was used during the event and up until now will be closed on July 2nd.

Once again, many thanks to all participants!
It was great to have you there and meet!

#### Disk Full

Unfortunately, I spent much less time than expected on fun stuff, since almost all my time yesterday and this morning went to shuffling around gigabytes of files on my overfilled PC and external hard disk.

I was still using the Revit Quasar release preview, which suddenly stopped working.
No big surprise, of course.

To
[obtain a one-box version](http://thebuildingcoder.typepad.com/blog/2012/04/migrate-building-coder-samples-to-revit-2013.html),
I went for the ultimate design suite, which requires downloading seven 4 GB files for a grand total of 27.3 GB.
After finally creating enough space for them and freeing up as much as possible for installation, I launched the installer.
It took an hour or longer to just copy temporary files and then aborted with a disk full error before even starting the installation proper.

I just went out and bought a new and bigger external hard disk, and am still working at freeing up more space.
Very time consuming business, this.

I later found two other helpful items which gave me another couple of gig:

- [Delete the Windows installer cache](http://www.ehow.com/how_5891038_delete-windows-installer-cache.html):
  1. Open Disk Cleanup: Start > Run > cleanmgr > OK. This will launch the Disk Cleanup utility.- Select 'Temporary Files', the Windows Installer cache directory.- Click OK.- [Clear the CCM cache](http://www.ehow.com/how_12157017_clear-ccm-cache.html):
    1. Select Start > Settings > Control Panel.- Double click 'Configuration Manager' and open the 'Advanced' tab.- Change the location of future cached files, if needed, by clicking the 'Change Location' button and choosing a new location on your hard drive. The default location is '%windir%\system32\CCM\Cache.'- Delete all cached files in CCM by clicking the 'Delete Files' button. Press 'Yes' if asked to confirm.- Click 'OK' to save changes and exit the Advanced window.

I'm up to 44 GB free space now...

Meanwhile, I can talk about what I did for the OBJ exporter before uninstalling Quasar RP.

#### Model up to Cloud and Down to Mobile

As I explained yesterday in the basic
[Revit model export considerations](http://thebuildingcoder.typepad.com/blog/2012/06/getting-going-with-the-cloud.html),
the purpose of this exercise is to push Revit model data up to the cloud and provide access to it from mobile devices.

Adam Nagy is also currently discussing a similar project on the AEC DevBlog.
After presenting his
[Revit add-in to upload geometry data to a storage service](http://adndevblog.typepad.com/aec/2012/06/revit-model-viewer-for-ios-part-1.html),
the second instalment discusses an
[iOS application to download and display the data using OpenGL](http://adndevblog.typepad.com/aec/2012/06/revit-model-viewer-for-ios-part-2.html).

One thing that I am thinking of doing is to define a flexible exporter architecture so that different export targets can easily be plugged in, for instance to switch between different file formats and disk-based versus cloud-based repositories.

#### Face Emitter Interface

For the moment, I am working with the following pretty trivial interface definition which only specifies one single output method to export a Revit geometry face:
```csharp
interface IJtFaceEmitter
{
  /// <summary>
  /// Emit a face with a specified colour.
  /// </summary>
  int EmitFace( Face face, Color color );

  /// <summary>
  /// Return the final triangle count
  /// after processing all faces.
  /// </summary>
  int GetFaceCount();

  /// <summary>
  /// Return the final triangle count
  /// after processing all faces.
  /// </summary>
  int GetTriangleCount();

  /// <summary>
  /// Return the final vertex count
  /// after processing all faces.
  /// </summary>
  int GetVertexCount();
}
```

It also provides methods to query the number of exported faces and the resulting triangles and vertices when finished.
I would have liked to define the count methods as simple properties instead, but that apparently cannot be done in an interface specification, unfortunately.

#### OBJ Exporter Implementation

The OBJ exporter class implementing this interface makes use of the vertex lookup dictionaries that I discussed yesterday to
[eliminate duplicate vertices](http://thebuildingcoder.typepad.com/blog/2012/06/obj-model-export-considerations.html#5).
Initially I used the
[XYZ-based lookup class](http://thebuildingcoder.typepad.com/blog/2012/06/obj-model-export-considerations.html#6), and
later I switched to the
[integer-based one](http://thebuildingcoder.typepad.com/blog/2012/06/obj-model-export-considerations.html#7) instead.

The exported implements the four methods required by the interface definition plus a fifth method to save the resulting file:

- EmitFace: export a Revit geometry Face object.- GetFaceCount, GetTriangleCount, GetVertexCount: return object counts.- ExportTo: save the resulting OBJ file.

The helper method StoreTriangle is called by EmitFace to populate the vertex lookup dictionary and triangle list.

The helper methods EmitVertex and EmitFacet are called by ExportTo to write out the OBJ vertex and face records.
```python
class ObjExporter : IJtFaceEmitter
{
  //VertexLookupXyz \_vertices;
  VertexLookupInt \_vertices;

  /// <summary>
  /// List of triangles, defined as
  /// triples of vertex indices.
  /// </summary>
  List<int> \_triangles;

  /// <summary>
  /// Keep track of the number of faces processed.
  /// </summary>
  int \_faceCount;

  /// <summary>
  /// Keep track of the number of triangles processed.
  /// </summary>
  int \_triangleCount;

  public ObjExporter()
  {
    \_faceCount = 0;
    \_triangleCount = 0;
    \_vertices = new VertexLookupInt();
    \_triangles = new List<int>();
  }

  /// <summary>
  /// Add the vertices of the given triangle to our
  /// vertex lookup dictionary and emit a triangle.
  /// </summary>
  void StoreTriangle( MeshTriangle triangle )
  {
    for( int i = 0; i < 3; ++i )
    {
      XYZ p = triangle.get\_Vertex( i );
      PointInt q = new PointInt( p );
      \_triangles.Add( \_vertices.AddVertex( q ) );
    }
  }

  /// <summary>
  /// Emit a Revit geometry Face object and
  /// return the number of resulting triangles.
  /// </summary>
  public int EmitFace( Face face, Color color )
  {
    ++\_faceCount;

    Mesh mesh = face.Triangulate();

    int n = mesh.NumTriangles;

    Debug.Print( " {0} mesh triangles", n );

    for( int i = 0; i < n; ++i )
    {
      ++\_triangleCount;

      MeshTriangle t = mesh.get\_Triangle( i );

      StoreTriangle( t );
    }
    return n;
  }

  public int GetFaceCount()
  {
    return \_faceCount;
  }

  /// <summary>
  /// Return the number of triangles processed.
  /// </summary>
  public int GetTriangleCount()
  {
    int n = \_triangles.Count;

    Debug.Assert( 0 == n % 3,
      "expected a multiple of 3" );

    Debug.Assert( \_triangleCount.Equals( n / 3 ),
      "expected equal triangle count" );
    return \_triangleCount;
  }

  public int GetVertexCount()
  {
    return \_vertices.Count;
  }

  #region ExportTo: output the OBJ file
  /// <summary>
  /// Obsolete: emit an XYZ vertex.
  /// </summary>
  static void EmitVertex( StreamWriter s, XYZ p )
  {
    s.WriteLine( "v {0} {1} {2}",
      Util.RealString( p.X ),
      Util.RealString( p.Y ),
      Util.RealString( p.Z ) );
  }

  /// <summary>
  /// Emit a vertex to OBJ. The first vertex listed
  /// in the file has index 1, and subsequent ones
  /// are numbered sequentially.
  /// </summary>
  static void EmitVertex(
    StreamWriter s,
    PointInt p )
  {
    s.WriteLine( "v {0} {1} {2}", p.X, p.Y, p.Z );
  }

  /// <summary>
  /// Emit an OBJ triangular face.
  /// </summary>
  static void EmitFacet(
    StreamWriter s,
    int i,
    int j,
    int k )
  {
    s.WriteLine( "f {0} {1} {2}",
      i + 1, j + 1, k + 1 );
  }

  public void ExportTo( string path )
  {
    using( StreamWriter s = new StreamWriter( path ) )
    {
      foreach( PointInt key in \_vertices.Keys )
      {
        EmitVertex( s, key );
      }

      int i = 0;
      int n = \_triangles.Count;

      while( i < n )
      {
        int i1 = \_triangles[i++];
        int i2 = \_triangles[i++];
        int i3 = \_triangles[i++];

        EmitFacet( s, i1, i2, i3 );
      }
    }
  }
  #endregion // ExportTo: output the OBJ file
}
```

The command mainline makes use of three other helper methods:

- GetSolid: Retrieve the first non-empty solid found for a given element.- ExportElement: Export an individual element.- ExportElements: Export all elements retrieved by a filtered element collector.

#### Retrieving a Solid from an Element

The GetSolid helper method retrieves the first non-empty solid found for a given element.

In case it is a family instance, it may have its own non-empty solid, in which case we use that.
Otherwise we search the symbol geometry.
If we use the symbol geometry, we might have to keep track of the instance transform to map it to the actual instance project location.
Instead, we ask for transformed geometry to be returned, so the resulting solid is already in place:
```csharp
Solid GetSolid( Element e, Options opt )
{
  Solid solid = null;

  GeometryElement geo = e.get\_Geometry( opt );

  if( null != geo )
  {
    if( e is FamilyInstance )
    {
      geo = geo.GetTransformed(
        Transform.Identity );
    }

    GeometryInstance inst = null;
    //Transform t = Transform.Identity;

    foreach( GeometryObject obj in geo )
    {
      solid = obj as Solid;

      if( null != solid
        && 0 < solid.Faces.Size )
      {
        break;
      }

      inst = obj as GeometryInstance;
    }

    if( null == solid && null != inst )
    {
      geo = inst.GetSymbolGeometry();
      //t = inst.Transform;

      foreach( GeometryObject obj in geo )
      {
        solid = obj as Solid;

        if( null != solid
          && 0 < solid.Faces.Size )
        {
          break;
        }
      }
    }
  }
  return solid;
}
```

#### Exporting an Element

The ExportElement helper method exports a given element and returns the number of solids found and exported from it.

If the element is a group, this method is called recursively on the group members, so the return value may be greater than one:
```python
int ExportElement(
  IJtFaceEmitter emitter,
  Element e,
  Options opt )
{
  Group group = e as Group;

  if( null != group )
  {
    int n = 0;

    foreach( ElementId id
      in group.GetMemberIds() )
    {
      Element e2 = e.Document.GetElement(
        id );

      n += ExportElement( emitter, e2, opt );
    }
    return n;
  }

  string desc = Util.ElementDescription( e );

  if( null == e.Category )
  {
    Debug.Print( "Element '{0}' has no "
      + "category.", desc );

    return 0;
  }

  Solid solid = GetSolid( e, opt );

  if( null == solid )
  {
    Debug.Print( "Unable to access "
      + "solid for element {0}.", desc );

    return 0;
  }

  Material material;
  Color color;

  foreach( Face face in solid.Faces )
  {
    material = e.Document.GetElement(
      face.MaterialElementId ) as Material;

    color = ( null == material )
      ? null
      : material.Color;

    emitter.EmitFace( face, color );
  }
  return 1;
}
```

#### Exporting all Collected Elements

The ExportElements helper method exports all elements returned by a filtered element collector and reports the results:
```csharp
void ExportElements(
  IJtFaceEmitter emitter,
  FilteredElementCollector collector,
  Options opt )
{
  int nElements = 0;
  int nSolids = 0;

  foreach( Element e in collector )
  {
    ++nElements;

    nSolids += ExportElement( emitter, e, opt );
  }

  int nFaces = emitter.GetFaceCount();
  int nTriangles = emitter.GetTriangleCount();
  int nVertices = emitter.GetVertexCount();

  string msg = string.Format(
    "{0} element{1} with {2} solid{3}, "
    + "{4} face{5}, {6} triangle{7} and "
    + "{8} vertice{9} exported.",
    nElements, Util.PluralSuffix( nElements ),
    nSolids, Util.PluralSuffix( nSolids ),
    nFaces, Util.PluralSuffix( nFaces ),
    nTriangles, Util.PluralSuffix( nTriangles ),
    nVertices, Util.PluralSuffix( nVertices ) );

  InfoMsg( msg );
}
```

#### Exporter Mainline

The external command mainline sets up a filtered element collector to define all elements to be exported.

It supports pre-selecting specific elements, in which case only those are used.

Even when limiting the filtered element collector to a predefined set, at least one filter must still be applied, or the filtered element collector will throw an exception.

I chose to apply the WhereElementIsNotElementType and WhereElementIsViewIndependent in all cases.

The user is prompted to select an output file, the exporter is instantiated, and the job is done:
```csharp
public Result Execute(
  ExternalCommandData commandData,
  ref string message,
  ElementSet elements )
{
  UIApplication uiapp = commandData.Application;
  UIDocument uidoc = uiapp.ActiveUIDocument;
  Application app = uiapp.Application;
  Document doc = uidoc.Document;

  // Determine elements to export

  FilteredElementCollector collector = null;

  // Access current selection

  SelElementSet set = uidoc.Selection.Elements;

  int n = set.Size;

  if( 0 < n )
  {
    // If any elements were preselected,
    // export those to OBJ

    ICollection<ElementId> ids = set
      .Cast<Element>()
      .Select<Element, ElementId>( e => e.Id )
      .ToArray<ElementId>();

    collector = new FilteredElementCollector( doc, ids );
  }
  else
  {
    // If nothing was preselected, export
    // all model elements to OBJ

    collector = new FilteredElementCollector( doc );
  }

  collector.WhereElementIsNotElementType()
      .WhereElementIsViewIndependent();

  if( null == \_export\_folder\_name )
  {
    \_export\_folder\_name = Path.GetTempPath();
  }

  string filename = null;

  if( !FileSelect( \_export\_folder\_name,
    out filename ) )
  {
    return Result.Cancelled;
  }

  \_export\_folder\_name
    = Path.GetDirectoryName( filename );

  ObjExporter exporter = new ObjExporter();

  Options opt = app.Create.NewGeometryOptions();

  ExportElements( exporter, collector, opt );

  exporter.ExportTo( filename );

  return Result.Succeeded;
}
```

#### Export Results

Here is the result of exporting a single wall.
It is actually one of the end walls in the standard Revit sample project rac\_advanced\_sample\_project.rvt:

![A single wall in OBJ](img/obj_export_complex_wall.png)

Here are the resulting object counts of resulting elements, faces, triangles, etc.:

![Single wall object counts](img/obj_export_complex_wall_msg.png)

Running the same command without preselecting anything exports the entire model.
Again, using rac\_advanced\_sample\_project.rvt, the result looks like this:

![Entire model in OBJ](img/obj_export_entire_model.png)

The following object counts are reported in this case:

![Entire model object counts](img/obj_export_entire_model_msg.png)

The resulting OBJ file [sample\_model.obj](src/ObjExport/test/sample_model.obj) is 5 MB in size, which is pretty tolerable considering all faces are completely triangulated.

As you can see in both of these cases, the number of faces is significantly higher than the number of vertices.
Just imagine what the number of vertices and resulting file size would be like if I had not gone to the effort of eliminating duplicate vertices.
The vertex count would simply have been three times the triangle count in both cases, i.e. 82 \* 3 = 246 instead of 43, a factor of 5.72, and 163964 \* 3 = 491892 instead of 86527, a factor of 5.68.