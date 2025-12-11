---
ai_optimized: true
author: Jeremy Tammik
complexity_score: 4.0
content_type: qa
optimization_date: '2025-12-11T11:44:15.748000'
original_url: https://thebuildingcoder.typepad.com/blog/1313_3dwc_twgl.html
post_number: '1313'
reading_time_minutes: 6
series: general
slug: 3dwc_twgl
source_file: 1313_3dwc_twgl.htm
tags:
- csharp
- elements
- family
- geometry
- levels
- revit-api
- views
- windows
title: Live Rendering of 3D Revit Element Geometry in a Remote WebGL Viewer
word_count: 1210
---

### Live Rendering of 3D Revit Element Geometry in a Remote WebGL Viewer

Today, I'll implement live real-time export of 3D geometry from a Revit add-in to a web-hosted WebGL viewer.

This is an enhancement to the initial version
[exporting 3D element geometry to a WebGL viewer](http://thebuildingcoder.typepad.com/blog/2015/04/exporting-3d-element-geometry-to-a-webgl-viewer.html),
which just generated data that I copied and pasted to hard-code it into the
[NodeWebGL viewer](https://github.com/jeremytammik/NodeWebGL) as
a proof of concept.

Meanwhile, I enhanced the WebGL viewer in various ways to prepare it for this real-time live connection, mainly by adding a REST API and support for HTTP POST:

- [Implementing a Node REST API server](http://the3dwebcoder.typepad.com/blog/2015/04/implementing-a-node-rest-api-server.html)
- [Implementing HTTP POST](http://the3dwebcoder.typepad.com/blog/2015/04/implementing-node-server-http-post-get-vs-post.html)
- [Adding a view engine](http://the3dwebcoder.typepad.com/blog/2015/04/post-data-and-swig-driven-webgl-viewer-node-server.html)
- [POST geometry data to the viewer](http://the3dwebcoder.typepad.com/blog/2015/04/node-server-rendering-using-either-swig-and-handlebars.html)
- [Cleaning up the Node server REST API implementation](http://the3dwebcoder.typepad.com/blog/2015/04/cleaner-node-server-rest-api-implementation.html)

Now that the viewer sports a REST API accepting POST data, the time has come to drive it directly from the Revit add-in instead of copy and paste of hard-coded geometry data into the viewer.

Here are the steps I took to achieve this:

- [Package geometry data into JSON](#2)
- [Send an HTTP POST Request and Display Result in Browser](#3)
- [Install Node.js on Windows](#4)
- [Demo](#5)
- [Download](#6)
- [Next steps](#7)

#### Package Geometry Data into JSON

I retrieve the geometry to be rendered by analysing the element geometry of a selected Revit element and generating lists of face vertices, normal vectors and indices to represent it.

In the initial implementation, I just printed this data to the debug console for manual copy and paste to the viewer.

Now I package that data into a JSON string for rendering via the [DisplayWgl](#3) method like this:

```csharp
  List<int> faceIndices = new List<int>();
  List<int> faceVertices = new List<int>();
  List<double> faceNormals = new List<double>();
  . . .
  // Scale the vertices to a [-1,1] cube
  // centered around the origin. Translation
  // to the origin was already performed above.

  double scale = 2.0 / FootToMm( MaxCoord( vsize ) );

  string sposition = string.Join( ", ",
    faceVertices.ConvertAll<string>(
      i => ( i \* scale ).ToString( "0.##" ) ) );

  string snormal = string.Join( ", ",
    faceNormals.ConvertAll<string>(
      f => f.ToString( "0.##" ) ) );

  string sindices = string.Join( ", ",
    faceIndices.ConvertAll<string>(
      i => i.ToString() ) );

  Debug.Print( "position: [{0}],", sposition );
  Debug.Print( "normal: [{0}],", snormal );
  Debug.Print( "indices: [{0}],", sindices );

  string json\_geometry\_data =
    "{ \"position\": [" + sposition
    + "],\n\"normal\": [" + snormal
    + "],\n\"indices\": [" + sindices
    + "] }";

  Debug.Print( "json: " + json\_geometry\_data );

  DisplayWgl( json\_geometry\_data );
```

With that in hand, it is time for the exciting stuff.

#### Send an HTTP POST Request and Display Result in Browser

The DisplayWgl implements the following tasks:

- Define the base URL and REST API route on the WebGL rendering server
- Support switching between a locally hosted or remote Heroku-hosted server
- Receive a JSON package defining the geometry data to be rendered
- Package the JSON string into a POST data package
- Sends it via a HTTP POST request to the server
- Retrieve the HTTP result
- Store it in a local file
- Display it in the browser

The implementation looks like this:

```csharp
  /// <summary>
  /// Toggle between a local server and
  /// a remote Heroku-hosted one.
  /// </summary>
  static public bool UseLocalServer = false;
. . .
  /// <summary>
  /// Invoke the node.js WebGL viewer web server.
  /// Use a local or global base URL and an HTTP POST
  /// request passing the 3D geometry data as body.
  /// </summary>
  void DisplayWgl( string json\_geometry\_data )
  {
    string base\_url = UseLocalServer
      ? "http://127.0.0.1:5000"
      : "https://nameless-harbor-7576.herokuapp.com";

    string api\_route = "api/v2";

    string uri = base\_url + "/" + api\_route;

    HttpWebRequest req = WebRequest.Create( uri ) as HttpWebRequest;

    req.KeepAlive = false;
    req.Method = WebRequestMethods.Http.Post;

    // Turn our request string into a byte stream.

    byte[] postBytes = Encoding.UTF8.GetBytes( json\_geometry\_data );

    req.ContentLength = postBytes.Length;

    // Specify content type.

    req.ContentType = "application/json; charset=UTF-8"; // or just "text/json"?
    req.Accept = "application/json";
    req.ContentLength = postBytes.Length;

    Stream requestStream = req.GetRequestStream();
    requestStream.Write( postBytes, 0, postBytes.Length );
    requestStream.Close();

    HttpWebResponse res = req.GetResponse() as HttpWebResponse;

    string result;

    using( StreamReader reader = new StreamReader(
      res.GetResponseStream() ) )
    {
      result = reader.ReadToEnd();
    }

    string filename = Path.GetTempFileName();
    filename = Path.ChangeExtension( filename, "html" );

    using( StreamWriter writer = File.CreateText( filename ) )
    {
      writer.Write( result );
      writer.Close();
    }

    System.Diagnostics.Process.Start( filename );
  }
```

I wonder whether there might be an easier or more efficient way to transfer the HTTP result to the browser than saving it to a local file.

There are probably more efficient methods, but this approach is hard to beat for easy of use.

#### Install Node.js on Windows

For local testing, I initially thought I would run the node.js server on the Mac and access that from the Windows virtual machine.

However, I was unable to access the Mac localhost running my local node server from Parallels.

Happily, it took one single click to install node.js on the Windows virtual machine and then run the unmodified code inside the virtual box:

Just go to [nodejs.org](https://nodejs.org) and click the big green Install button.

After doing so, I could immediately run the existing server locally inside the box:

![NodeWebGL running locally on Windows](img/twgl_node_js_windows.png)

It worked fine locally.

Next, I switched to the remote viewer.

That worked as well with no problem.

#### Demo

Here is a quick [one-minute video](https://youtu.be/qBjbb_Wv6Qk) showing the TwglExport Revit add-in and the Heroku-hosted node.js WebGL server NodeWebGL in concerted action:

#### Download

The entire TwglExport source code, Visual Studio solution and add-in manifest are provided in the
[TwglExport GitHub repository](https://github.com/jeremytammik/TwglExport),
and the version presented here is
[release 2015.0.0.3](https://github.com/jeremytammik/TwglExport/releases/tag/2015.0.0.3).

The complete node server implementation is available from the
[NodeWebGL GitHub repo](https://github.com/jeremytammik/NodeWebGL),
and the version discussed here is
[0.2.7](https://github.com/jeremytammik/NodeWebGL/releases/tag/0.2.7).

#### Next Steps

The Revit element traversal is currently totally simplistic.

It grabs the first non-empty solid it can find and renders that with no questions asked.

This will not work for a family instance, for instance – please pardon the pun – to access any solid contained in its geometry, you have to navigate through the geometry instance level first.

I could obviously enhance the geometry traversal, as we already did for numerous other add-ins, e.g. the
[OBJ exporter](http://thebuildingcoder.typepad.com/blog/2012/07/obj-model-exporter-with-multiple-solid-support.html).

That would be silly, though.

It is much easier to implement a
[custom exporter](http://thebuildingcoder.typepad.com/blog/about-the-author.html#5.1) and grab the geometry from that.

No more worries about elements, transformations, instances and all that stuff.

Stay tuned and have fun!