---
ai_optimized: true
author: Jeremy Tammik
complexity_score: 9.4
content_type: code_example
optimization_date: '2025-12-11T11:44:15.043234'
original_url: https://thebuildingcoder.typepad.com/blog/0981_custom_exporter_json.html
post_number: 0981
reading_time_minutes: 18
series: general
slug: custom_exporter_json
source_file: 0981_custom_exporter_json.htm
tags:
- csharp
- elements
- family
- python
- revit-api
- transactions
- views
- walls
- windows
title: ADN Mesh Data Custom Exporter to JSON
word_count: 3548
---

### ADN Mesh Data Custom Exporter to JSON

I mentioned my idea of implementing a
[custom exporter to JSON](http://thebuildingcoder.typepad.com/blog/2013/07/graphics-pipeline-custom-exporter.html#7) to drive
[Philippe Leefsma](http://adndevblog.typepad.com/cloud_and_mobile/philippe-leefsma.html)'s online
[3D WebGL viewer](http://adndevblog.typepad.com/cloud_and_mobile/2013/06/3d-webgl-viewer-with-javascript-and-threejs.html).

Well, here it is.

#### Muttenhorn

Before getting to that, though, let me briefly mention that I went on a nice mountain with my friend Martin last Saturday, over the Gross Muttenhorn on the south side of the Furka pass.

A glacier named Muttgletscher lies over its north-western flanks, and we crossed that on our ascent up to the west ridge.
From the summit we continued down the east ridge to descend between the three lakes next to the Stotzigen Firsten:

![A lake at Stotzigen Firsten by Muttenhorn](file:////j/photo/jeremy/2013/2013-07-06_muttenhorn/p1020836_blue_lake_stotzigen_firsten.jpg)

Here are some
[more pictures](https://www.facebook.com/media/set/?set=a.10200914722843221.1073741825.1019863650&type=1) if
you be so inclined.

#### Gwen and Dave's Getaway

Another little non-Revit pointer is to the
[Gwen and Dave's Getaway](http://area.autodesk.com/contest/winners) animated
short film contest.
I really like the two top prize winners,
[Sky Fishing](http://area.autodesk.com/contest/videos/sky-fishing#content) by Austin Reddington and
[Peace and Quiet](http://area.autodesk.com/contest/videos/peace-and-quiet#content) by
Khye Kading; well worth just a few minutes of your time.

Now, back to business.

#### ADN Mesh Data Custom Exporter

I was originally expecting to be done in just a few hours, but this turned out to be a full one day plus night project that kept me happily busy until four o'clock this morning, touching on numerous topics and challenges both named and unnamed, some of which are:

- [ADN mesh data format](#2)
- [Tetrahedron sample JSON data](#3)
- [Little house and curved wall in JSON](#4)
- [Custom exporter implementation and components](#5)
- [Centroid and volume](#6)
- [Export context implementation](#7)
- [ADN mesh data class](#8)
- [JSON serialisation](#9)

#### ADN Mesh Data Format

Before we can implement our custom exporter, we need a definition of the JSON format to generate.
I analysed the files provided by Philippe and described the result in my
[comment](http://adndevblog.typepad.com/cloud_and_mobile/2013/06/3d-webgl-viewer-with-javascript-and-threejs.html?cid=6a0167607c2431970b0192abb7b8d8970d#comment-6a0167607c2431970b0192abb7b8d8970d) on his post:
I see the FacetCount, VertexCount, VertexCoords, VertexIndices, Normals, NormalIndices, Center, Color and Id properties define in the JSON file.
Exactly how are these defined, and are some of them optional?

Philippe replies: the AdnMeshData class definition specifies the data format:

```csharp
  public class AdnMeshData
  {
    public AdnMeshData()
    {
    }

    public int FacetCount
    {
      get;
      protected set;
    }

    public int VertexCount
    {
      get;
      protected set;
    }

    public double[] VertexCoords
    {
      get;
      protected set;
    }

    public int[] VertexIndices
    {
      get;
      protected set;
    }

    public double[] Normals
    {
      get;
      protected set;
    }

    public int[] NormalIndices
    {
      get;
      protected set;
    }

    public double[] Center
    {
      get;
      protected set;
    }

    public int[] Color
    {
      get;
      protected set;
    }

    public string Id
    {
      get;
      protected set;
    }
  }
```

All remaining issues are clarified by this little Q&A:

- Are only triangles supported?
   –  Yes, only triangles, that’s what Inventor provides.
- Are any of the properties optional?
   –  FacetCount and VertexCount are optional.
- What happens if you simply omit the normals?
  Will they be automatically calculated from the triangle definitions?
   –  As it is now it won’t work.
  Those are the normals at the vertices, not at the faces, this info is important to webgl rendering to compute the lights in a realistic way.
  This is what makes the model looks good even if it has been triangulated.
- What about the facet and vertex count?
  They could be automatically deduced from the lists provided.
   –  Yep, in the webgl viewer those properties aren’t used actually.

That should be enough to get us up and running.

#### Tetrahedron Sample JSON Data

After I completed my very first stab at the implementation and dragged the resulting JSON output file onto Philippe's web viewer, nothing was displayed.

By the way, I am making of this
[offline version](zip/webgl-viewer-5.zip) that
he provided for testing.

The reason turned out to be some misunderstanding about the triangle vertex order in the JSON input file.

To ensure I could understand what was going on and how the vertices need to be sorted, I implemented a little JSON file by hand defining a tetrahedron between the four points (0,0,0), (10,0,0), (0,10,0) and (0,0,10):

```
[
{
 "FacetCount":4,
 "VertexCount":4,
 "VertexCoords":[0,0,0, 10,0,0, 0,10,0, 0,0,10],
 "VertexIndices":[0,2,1, 0,1,3, 0,3,2, 1,2,3],
 "Normals":[0,0,-1, 0,-1,0, -1,0,0, 1,1,1],
 "NormalIndices":[0,0,0, 1,1,1, 2,2,2, 3,3,3],
 "Center":[3,3,3],
 "Color":[-2139062017],
 "Id":"tetrahedron"
}
]
```

Here is what it looks like in the web viewer:

![WebGL viewer showing tetrahedron](img/webgl_tetrahedron.png)

#### Little House and Curved Wall in JSON

Once I had that sorted, I proceeded to debug the display of one single wall, and progressed to the little house generated by the ADN training labs.

In Revit, it looks like this in perspective view:

![Little house in perspective view](img/little_house_perspective_view.png)

The WebGL rendering of the JSON export looks like this:

![Little house in WebGL](img/webgl_little_house.png)

Finally, here is a rather strange sample BIM from Philippe,
[CurvedWall.rvt](zip/CurvedWall.rvt),
its old JSON representation
[CurvedWall.json](zip/CurvedWall.json),
generated using the Revit 2013 generator based on the ElementViewer SDK sample, and the new version
[CurvedWallJt.json](zip/CurvedWallJt.json) generated
by this custom exporter, looking like this in the WebGL viewer:

![Curved wall model in WebGL](img/CurvedWall.png)

#### Custom Exporter Implementation and Components

I ended up reusing a number of components from previous projects for this little endeavour.

Here is
[CustomExporterAdnMeshJson.zip](zip/CustomExporterAdnMeshJson.zip) containing
the complete source code, Visual Studio solution and add-in manifest for the ADN mesh data custom exporter external command add-in.

It consists of the following modules:

- AdnMeshData.cs – The data format specifying one solid for the WebGL viewer, defining its centre, colour, id, triangular facets, their vertex coordinates, indices and normals as discussed above.
- CentroidVolume.cs – Calculate and store the
  [centroid and volume](http://thebuildingcoder.typepad.com/blog/2012/12/solid-centroid-and-volume-calculation.html) from a set of triangular facets.
- Command.cs – ADN mesh data custom exporter external command mainline.
- ExportContextAdnMesh.cs – Custom exporter IExportContext implementation to capture ADN mesh data.
- NormalLookupXyz.cs – A facet normal vector lookup class to avoid duplicate normal vector definitions, similar to the vertex lookup mentioned below.
- PointInt.cs – An integer-based 3D point class, supporting the vertex lookup, storing the vertices in integer number millimetres for the sake of efficiency, readability, and to avoid all rounding issues.
- Util.cs – Utility methods.
- VertexLookupInt.cs – A vertex lookup class to avoid duplicate vertex definitions, reused from the
  [OBJ exporter](http://thebuildingcoder.typepad.com/blog/2013/07/graphics-pipeline-custom-exporter.html#6).

Which ones should we discuss in greater depth?

I will skip the integer-based point and lookup classes, since I have already belaboured them repeatedly in the past.

#### Centroid and Volume

The ADN mesh data format requires a centre point.

The custom exporter is fed faces, and does not have a built-in mechanism to identify solids.

Therefore, the determination of a centre point needs to be based on the facets we receive instead.

Happily, I already discussed how to determine
[centroid and volume](http://thebuildingcoder.typepad.com/blog/2012/12/solid-centroid-and-volume-calculation.html) using
an algorithm that calculates and stores these properties from a set of triangular facets.

I cleaned up the previous implementation to make its use more obvious, and it now looks like this:

```python
class CentroidVolume
{
  XYZ \_centroid;
  double \_volume;

  public CentroidVolume()
  {
    Init();
  }

  public void Init()
  {
    \_centroid = XYZ.Zero;
    \_volume = 0.0;
  }

  public void AddTriangle( XYZ[] p )
  {
    double vol
      = p[0].X \* ( p[1].Y \* p[2].Z - p[2].Y \* p[1].Z )
      + p[0].Y \* ( p[1].Z \* p[2].X - p[2].Z \* p[1].X )
      + p[0].Z \* ( p[1].X \* p[2].Y - p[2].X \* p[1].Y );

    \_centroid += vol \* ( p[0] + p[1] + p[2] );
    \_volume += vol;
  }

  /// <summary>
  /// Set centroid coordinates and volume
  /// to their final values when completed.
  /// </summary>
  public void Complete()
  {
    \_centroid /= 4 \* \_volume;
    \_volume /= 6;
  }

  public XYZ Centroid
  {
    get
    {
      return \_centroid;
    }
  }

  public double Volume
  {
    get
    {
      return \_volume;
    }
  }

  override public string ToString()
  {
    return Util.RealString( \_volume ) + "@"
      + Util.PointString( \_centroid );
  }
}
```

In my current test implementation, I just close the calculation when an element ends.

This should probably be improved to terminate every time the material changes, to handle cases like windows with several different components using different materials.

Actually, you can see that the windows in the screen snapshots above are not perfectly rendered due to this.

#### Export Context Implementation

By far the most complex module is the export context implementation.

The IExportContext interface specifies the following methods:

```csharp
  void Finish();
  bool IsCanceled();
  void OnDaylightPortal( DaylightPortalNode node );
  RenderNodeAction OnElementBegin( ElementId elementId );
  void OnElementEnd( ElementId elementId );
  RenderNodeAction OnFaceBegin( FaceNode node );
  void OnFaceEnd( FaceNode node );
  RenderNodeAction OnInstanceBegin( InstanceNode node );
  void OnInstanceEnd( InstanceNode node );
  void OnLight( LightNode node );
  RenderNodeAction OnLinkBegin( LinkNode node );
  void OnLinkEnd( LinkNode node );
  void OnMaterial( MaterialNode node );
  void OnPolymesh( PolymeshTopology node );
  void OnRPC( RPCNode node );
  RenderNodeAction OnViewBegin( ViewNode node );
  void OnViewEnd( ElementId elementId );
  bool Start();
```

A very few of them can be left unimplemented, at least in my simple test project, but most need attention.
I left the NotImplementedException statements in the unimplemented ones, so that I am notified if they are called.

I implemented some rudimentary logging to see in which order the methods are called in the debug output window.

A number of the methods provide support for cancelling the rendering process and need to return true or RenderNodeAction.Proceed for it to continue.

Family instances and links need to push their transformations onto a stack, and all vertices received need to be transformed appropriately.

Here is the complete implementation of the ADN mesh data export context in its current state:

```python
class ExportContextAdnMesh : IExportContext
{
  Document \_doc;

  /// <summary>
  /// Stack of transformations for
  /// link and instance elements.
  /// </summary>
  Stack<Transform> \_transformationStack
    = new Stack<Transform>();

  /// <summary>
  /// List of triangle vertices.
  /// </summary>
  VertexLookupInt \_vertices = new VertexLookupInt();

  /// <summary>
  /// List of triangles, defined as
  /// triples of vertex indices.
  /// </summary>
  List<int> \_triangles = new List<int>();

  /// <summary>
  /// List of normal vectors, defined by an index
  /// into the normal lookup for each triangle vertex.
  /// </summary>
  List<int> \_normalIndices = new List<int>();

  NormalLookupXyz \_normals = new NormalLookupXyz();

  /// <summary>
  /// Calculate centre of gravity of current element.
  /// </summary>
  CentroidVolume \_centroid\_volume
    = new CentroidVolume();

  Color \_color;
  double \_transparency;
  List<AdnMeshData> \_data;

  public ExportContextAdnMesh( Document doc )
  {
    \_doc = doc;
    \_data = new List<AdnMeshData>();
    \_transformationStack.Push( Transform.Identity );
  }

  public AdnMeshData[] MeshData
  {
    get
    {
      return \_data.ToArray();
    }
  }

  Transform CurrentTransform
  {
    get
    {
      return \_transformationStack.Peek();
    }
  }

  /// <summary>
  /// Store a triangle, adding new vertices for it
  /// to our vertex lookup dictionary if needed and
  /// accumulating its volume and centroid contribution.
  /// </summary>
  void StoreTriangle(
    IList<XYZ> vertices,
    PolymeshFacet triangle,
    XYZ normal )
  {
    // Retrieve the three triangle vertices

    Transform currentTransform = CurrentTransform;

    XYZ[] p = new XYZ[] {
      currentTransform.OfPoint( vertices[triangle.V1] ),
      currentTransform.OfPoint( vertices[triangle.V2] ),
      currentTransform.OfPoint( vertices[triangle.V3] )
    };

    // Ensure the three are ordered counter-clockwise

    XYZ v = p[1] - p[0];
    XYZ w = p[2] - p[0];
    Debug.Assert( Util.IsRightHanded( v, w, normal ),
      "expected counter-clockwise vertex order" );

    // Centroid and volume calculation

    \_centroid\_volume.AddTriangle( p );

    // Store vertex, facet and normals

    for( int i = 0; i < 3; ++i )
    {
      PointInt q = new PointInt( p[i] );

      \_triangles.Add( \_vertices.AddVertex( q ) );

      \_normalIndices.Add( \_normals.AddNormal(
        currentTransform.OfVector( normal ) ) );
    }
  }

  public void Finish()
  {
    Debug.Print( "Finish" );
  }

  public bool IsCanceled()
  {
    return false;
  }

  public void OnDaylightPortal(
    DaylightPortalNode node )
  {
    throw new NotImplementedException();
  }

  public RenderNodeAction OnElementBegin(
    ElementId elementId )
  {
    string s = elementId.IntegerValue.ToString();

    Debug.Print( "ElementBegin id " + s );

    \_vertices.Clear();
    \_triangles.Clear();
    \_normals.Clear();
    \_normalIndices.Clear();
    \_centroid\_volume.Init();

    return RenderNodeAction.Proceed;
  }

  public void OnElementEnd( ElementId elementId )
  {
    Debug.Print( "ElementEnd" );

    // Set centroid coordinates to their final value

    \_centroid\_volume.Complete();

    string metadataId = \_doc.GetElement(
      elementId ).UniqueId;

    AdnMeshData meshData = new AdnMeshData(
      \_vertices, \_triangles, \_normals, \_normalIndices,
      new PointInt( \_centroid\_volume.Centroid ),
      \_color, \_transparency, metadataId );

    \_data.Add( meshData );
  }

  public RenderNodeAction OnFaceBegin( FaceNode node )
  {
    throw new NotImplementedException();
  }

  public void OnFaceEnd( FaceNode node )
  {
    throw new NotImplementedException();
  }

  public RenderNodeAction OnInstanceBegin(
    InstanceNode node )
  {
    FamilySymbol symbol = \_doc.GetElement(
      node.GetSymbolId() ) as FamilySymbol;

    Debug.Assert( null != symbol,
      "expected valid family symbol" );

    Debug.Print( "InstanceBegin "
      + symbol.Category.Name + " : "
      + symbol.Family.Name + " : "
      + symbol.Name );

    \_transformationStack.Push( CurrentTransform
      .Multiply( node.GetTransform() ) );

    return RenderNodeAction.Proceed;
  }

  public void OnInstanceEnd( InstanceNode node )
  {
    Debug.Print( "InstanceEnd" );

    \_transformationStack.Pop();
  }

  public void OnLight( LightNode node )
  {
    throw new NotImplementedException();
  }

  public RenderNodeAction OnLinkBegin( LinkNode node )
  {
    \_transformationStack.Push( CurrentTransform
      .Multiply( node.GetTransform() ) );

    throw new NotImplementedException();
  }

  public void OnLinkEnd( LinkNode node )
  {
    \_transformationStack.Pop();

    throw new NotImplementedException();
  }

  public void OnMaterial( MaterialNode node )
  {
    Color c = node.Color;
    double t = node.Transparency;

    string s = string.Format( "({0},{1},{2})",
      c.Red, c.Green, c.Blue );

    Debug.Print( "Colour " + s + ", transparency "
      + t.ToString( "0.##" ) );

    \_color = c;
    \_transparency = t;
  }

  public void OnPolymesh( PolymeshTopology node )
  {
    int nPts = node.NumberOfPoints;
    int nFacets = node.NumberOfFacets;

    DistributionOfNormals distrib
      = node.DistributionOfNormals;

    Debug.Print( string.Format(
      "Polymesh {0} vertices {1} facets",
      nPts, nFacets ) );

    int iFacet = 0;
    int iPoint = 0;

    IList<XYZ> vertices = node.GetPoints();
    IList<XYZ> normals = node.GetNormals();
    XYZ normal;

    foreach( PolymeshFacet triangle in node.GetFacets() )
    {
      // Just grab one normal per facet; ignore the
      // three normals per point if they differ.

      if( DistributionOfNormals.OnePerFace == distrib )
      {
        normal = node.GetNormal( 0 );
      }
      else if( DistributionOfNormals.OnEachFacet
        == distrib )
      {
        normal = node.GetNormal( iFacet++ );
      }
      else
      {
        Debug.Assert( DistributionOfNormals
          .AtEachPoint == distrib, "what else?" );

        normal = node.GetNormal( iPoint++ )
          + node.GetNormal( iPoint++ )
          + node.GetNormal( iPoint++ );
        normal /= 3.0;
      }

      StoreTriangle( vertices, triangle, normal );
    }
  }

  public void OnRPC( RPCNode node )
  {
    throw new NotImplementedException();
  }

  public RenderNodeAction OnViewBegin( ViewNode node )
  {
    View3D view = \_doc.GetElement( node.ViewId )
      as View3D;

    Debug.Assert( null != view,
      "expected valid 3D view" );

    Debug.Print( "ViewBegin " + view.Name );

    return RenderNodeAction.Proceed;
  }

  public void OnViewEnd( ElementId elementId )
  {
    Debug.Print( "ViewEnd" );
  }

  public bool Start()
  {
    Debug.Print( "Start" );
    return true;
  }
}
```

The rendering generates a list of AdnMeshData instances.

Each one of them contains the data required to render one solid.

#### ADN Mesh Data Class

Here are the methods used to structure and store the ADN mesh data:

```csharp
class AdnMeshData
{
  int FacetCount { get; set; } // optional
  int VertexCount { get; set; } // optional
  int[] VertexCoords { get; set; }
  int[] VertexIndices { get; set; } // triangles
  double[] Normals { get; set; }
  int[] NormalIndices { get; set; } // not optional, one normal per vertex
  int[] Center { get; set; }
  int Color { get; set; }
  string Id { get; set; }

  /// <summary>
  /// Apply this factor to all point data when
  /// saving to JSON to accomodate the expected
  /// scaling.
  /// </summary>
  const double \_export\_factor = 0.002;

  public AdnMeshData(
    VertexLookupInt vertices,
    List<int> vertexIndices,
    NormalLookupXyz normals,
    List<int> normalIndices,
    PointInt center,
    Color color,
    double transparency,
    string id )
  {
    int n = vertexIndices.Count;

    Debug.Assert( 0 == (n % 3),
      "expected triples of 3D point vertex indices" );

    Debug.Assert( normalIndices.Count == n,
      "expected a normal for each vertex" );

    FacetCount = n / 3;

    n = vertices.Count;
    VertexCount = n;
    VertexCoords = new int[n \* 3];
    int i = 0;
    foreach( PointInt p in vertices.Keys )
    {
      VertexCoords[i++] = p.X;
      VertexCoords[i++] = p.Y;
      VertexCoords[i++] = p.Z;
    }
    VertexIndices = vertexIndices.ToArray();

    n = normals.Count;
    Normals = new double[n \* 3];
    i = 0;
    foreach( XYZ v in normals.Keys )
    {
      Normals[i++] = v.X;
      Normals[i++] = v.Y;
      Normals[i++] = v.Z;
    }
    NormalIndices = normalIndices.ToArray();

    Center = new int[3];
    i = 0;
    Center[i++] = center.X;
    Center[i++] = center.Y;
    Center[i] = center.Z;

    byte alpha = (byte) (
      ( 100 - transparency ) \* 2.55555555 );

    Color = ConvertClr(
      color.Red, color.Green, color.Blue, alpha );

    Id = id;
  }

  /// <summary>
  /// Convert colour and transparency to
  /// the required integer format.
  /// </summary>
  static int ConvertClr( byte r, byte g, byte b, byte a )
  {
    return ( r << 24 ) + ( g << 16 ) + ( b << 8 ) + a;
  }
```

That completes the input side of this class.

It is followed by the output:

#### JSON Serialisation

Once the ADN mesh data has been assembled, I want to export it to a JSON file to pass into Philippe's viewer.

I did think of using a JSON serialisation library.
The .NET framework provides two different ones:

- System.Web.Script.Serialization.JavaScriptSerializer
- System.Runtime.Serialization.Json.DataContractJsonSerializer

In addition, numerous other libraries are available.

I read a nice short discussion on their various advantages and disadvantages, resulting in the author rolling his own
[FridayThe13th library](http://procbits.com/2011/08/11/fridaythe13th-the-best-json-parser-for-silverlight-and-net).

That prompted me to simply implement the serialisation myself inline in a member method of the AdnMeshData class like this:

```csharp
public string ToJson()
{
  string s = string.Format
    ( "\n \"FacetCount\":{0},"
    + "\n \"VertexCount\":{1},"
    + "\n \"VertexCoords\":[{2}],"
    + "\n \"VertexIndices\":[{3}],"
    + "\n \"Normals\":[{4}],"
    + "\n \"NormalIndices\":[{5}],"
    + "\n \"Center\":[{6}],"
    + "\n \"Color\":[{7}],"
    + "\n \"Id\":\"{8}\"",
    FacetCount,
    VertexCount,
    string.Join( ",", VertexCoords.Select<int, string>( i => ( \_export\_factor \* i ).ToString( "0.#" ) ).ToArray() ),
    string.Join( ",", VertexIndices.Select<int, string>( i => i.ToString() ).ToArray() ),
    string.Join( ",", Normals.Select<double, string>( a => a.ToString( "0.####" ) ).ToArray() ),
    string.Join( ",", NormalIndices.Select<int, string>( i => i.ToString() ) ),
    string.Join( ",", Center.Select<int, string>( i => ( \_export\_factor \* i ).ToString( "0.#" ) ) ),
    Color,
    Id );

  return "\n{" + s + "\n}";
}
```

Notice the last minute scaling applied via the \_export\_factor to generate a model in a suitable size for Philippe's viewer :-)

#### Driving the Process and Streaming to File

The command mainline checks for a valid 3D view in a valid project document, then drives the exporter and retrieves its mesh data instances to serialise the data to JSON like this:

```python
[Transaction( TransactionMode.ReadOnly )]
public class Command : IExternalCommand
{
  public Result Execute(
    ExternalCommandData commandData,
    ref string message,
    ElementSet elements )
  {
    UIApplication uiapp = commandData.Application;
    UIDocument uidoc = uiapp.ActiveUIDocument;
    Application app = uiapp.Application;
    Document doc = uidoc.Document;

    // This command requires an active document

    if( null == uidoc )
    {
      message = "Please run this command in an active project document.";
      return Result.Failed;
    }

    View3D view = doc.ActiveView as View3D;

    if( null == view )
    {
      message = "Please run this command in a 3D view.";
      return Result.Failed;
    }

    // Instantiate our custom context

    ExportContextAdnMesh context
      = new ExportContextAdnMesh( doc );

    // Instantiate a custom exporter with it

    using( CustomExporter exporter
      = new CustomExporter( doc, context ) )
    {
      // Tell the exporter whether we need face info.
      // If not, it is better to exclude them, since
      // processing faces takes significant time and
      // memory. In any case, tessellated polymeshes
      // can be exported (and will be sent to the
      // context). Excluding faces just excludes the calls,
      // not the actual processing of face tessellation.
      // Meshes of the faces will still be received by
      // the context.

      exporter.IncludeFaces = false;

      exporter.Export( view );
    }

    // Save ADN mesh data in JSON format

    StreamWriter s = new StreamWriter(
      "C:/tmp/test.json" );

    s.Write( "[" );

    int i = 0;

    foreach( AdnMeshData d in context.MeshData )
    {
      if( 0 < i ) { s.Write( ',' ); }

      s.Write( d.ToJson() );

      ++i;
    }

    s.Write( "\n]\n" );
    s.Close();

    return Result.Succeeded;
  }
}
```

The complete code and solution is available for
[download](#5) above.

The only remaining open issue with this right now that I am aware of is how to generate different ADN mesh data instances for different materials, e.g. separate output for a window pane and its frame.

So now we have three different and pretty diverse custom exporters to experiment with.

I am looking forward to hearing what experiences you have with the custom exporter framework.

---

# Cloud and Mobile

### Revit ADN Mesh Data Custom Exporter to JSON

By
[Jeremy](http://adndevblog.typepad.com/cloud_and_mobile/jeremy-tammik.html)
[Tammik](http://thebuildingcoder.typepad.com/blog/about-the-author.html).

I implemented a new Revit 2014
[custom exporter to JSON](http://thebuildingcoder.typepad.com/blog/2013/07/adn-mesh-data-custom-exporter-to-json.html#1) generating the ADN mesh data to drive
[Philippe Leefsma](http://adndevblog.typepad.com/cloud_and_mobile/philippe-leefsma.html)'s online
[3D WebGL viewer](http://adndevblog.typepad.com/cloud_and_mobile/2013/06/3d-webgl-viewer-with-javascript-and-threejs.html).

This is obviously mainly of interest to Revit API aficionados, though some of the topics covered are completely generic cloud and mobile issues as well:

- [ADN mesh data format](http://thebuildingcoder.typepad.com/blog/2013/07/adn-mesh-data-custom-exporter-to-json.html#2)
- [Tetrahedron sample JSON data](http://thebuildingcoder.typepad.com/blog/2013/07/adn-mesh-data-custom-exporter-to-json.html#3)
- [Little house and curved wall in JSON](http://thebuildingcoder.typepad.com/blog/2013/07/adn-mesh-data-custom-exporter-to-json.html#4)
- [Custom exporter implementation and components](http://thebuildingcoder.typepad.com/blog/2013/07/adn-mesh-data-custom-exporter-to-json.html#5)
- [Centroid and volume](http://thebuildingcoder.typepad.com/blog/2013/07/adn-mesh-data-custom-exporter-to-json.html#6)
- [Export context implementation](http://thebuildingcoder.typepad.com/blog/2013/07/adn-mesh-data-custom-exporter-to-json.html#7)
- [ADN mesh data class](http://thebuildingcoder.typepad.com/blog/2013/07/adn-mesh-data-custom-exporter-to-json.html#8)
- [JSON serialisation](http://thebuildingcoder.typepad.com/blog/2013/07/adn-mesh-data-custom-exporter-to-json.html#9)

The post also mentions my most recent mountain hike to the
[Muttenhorn](http://thebuildingcoder.typepad.com/blog/2013/07/adn-mesh-data-custom-exporter-to-json.html#muttenhorn) :-)

![A lake at Stotzigen Firsten by Muttenhorn](file:////j/photo/jeremy/2013/2013-07-06_muttenhorn/p1020836_blue_lake_stotzigen_firsten.jpg)