---
post_number: "1008"
title: "Open MEP Connector Warning"
slug: "loose_connector"
author: "Jeremy Tammik"
tags: ['csharp', 'elements', 'parameters', 'revit-api', 'rooms', 'schedules', 'selection', 'sheets', 'views', 'windows']
source_file: "1008_loose_connector.htm"
original_url: "https://thebuildingcoder.typepad.com/blog/1008_loose_connector.html"
---

### Open MEP Connector Warning

I presented a little utility add-in yesterday that
[determines the maximal flow](http://thebuildingcoder.typepad.com/blog/2013/08/determining-maximal-flow-in-hvac-duct-connectors.html) in
HVAC duct connectors and populates a shared parameter with the result.

It received positive feedback, and triggered an additional request for new functionality:

**Response:** Looks great!
This is a huge step forward.
The routine runs as expected, and actually uncovered a disconnect in my sample model.
I was also then able to run it successfully on other, bigger models with no issues.

One situation where we may improve the functionality might be where duct has an open end. Here are two examples of this:

![Open connector](img/duct_max_flow_open_end_1.png)

![Open connector](img/duct_max_flow_open_end_2.png)

In these scenarios, flow calculations are incorrect because the duct is open.
In the other instances you will see an endcap in this position.
It is best practice in Revit (and life) to close these systems.

**Answer:** Thank you for testing and confirming that it does the job.

Regarding the loose connectors, I actually implemented a quite serious application for checking those a while back, mainly as one of the early samples demonstrating use of the Idling event in Revit 2011 and 2012:

- [Modeless loose connectors](http://thebuildingcoder.typepad.com/blog/2010/07/modeless-loose-connectors.html)
- [Modeless loose connector navigator update](http://thebuildingcoder.typepad.com/blog/2011/08/modeless-loose-connector-navigator-update.html)
- [Yet another modeless update](http://thebuildingcoder.typepad.com/blog/2011/09/yet-another-modeless-update.html)

The modeless loose connector navigator is a complete tool in its own right, though, enabling you to navigate through a modeless list of all unconnected connecters and manipulate the model at the same time, jumping back and forth as you like.

I have not updated it for 2014, though, and actually not even for 2013 :-)

So putting some minimal functionality into the new sample to check for open connectors instead makes good sense.

Each Connector instance provides a property IsConnected that can be used to check whether it is open or not.

It is therefore very easy to use LINQ to determine the number of open connectors, i.e. whose IsConnected property returns false, e.g. like this:

```csharp
  ConnectorSet connectors
    = GetConnectorManager( e ).Connectors;

  int nUnconnected = connectors
    .Cast<Connector>()
    .Count<Connector>( c => !c.IsConnected );

  bool hasOpenConnectors = 0 < nUnconnected;

  string s = 0 == nUnconnected
    ? string.Empty
    : string.Format( " with {0} open connector{1}",
      nUnconnected, Util.PluralSuffix( nUnconnected ) );
```

I added that code to the
[SetMaxFlowOnElement](http://thebuildingcoder.typepad.com/blog/2013/08/determining-maximal-flow-in-hvac-duct-connectors.html#4) method
that I presented yesterday, and added some support for reporting back if any open connectors at all are found in the model.

This leads to an additional clause in the result message displayed at the end stating how many elements have open connectors:

![Duct iteration result message](img/duct_max_flow_result_msg.png)

The additional clause is omitted if the number is zero, as it normally should be.

The detailed information on the offending elements is provided by the log in the Visual Studio debug output console window (copy and paste to an editor or view source to see the truncated lines in full):

```
Duct <630420 Mitered Elbows> has 2 connectors and max flow 22.08.
. . .
Duct <630695 Short Radius> has 2 connectors and max flow 4.58.
Duct <630991 Mitered Elbows> has 2 connectors and max flow 0.
Duct <630992 Mitered Elbows> has 2 connectors and max flow 0 with 1 open connector.
Duct <630995 Mitered Elbows> has 2 connectors and max flow 4.58.
. . .
Duct <631007 Short Radius> has 2 connectors and max flow 4.58.
Duct <631074 Mitered Elbows> has 2 connectors and max flow 0.
Duct <631075 Mitered Elbows> has 6 connectors and max flow 8.33 with 1 open connector.
Duct <631077 Mitered Elbows> has 2 connectors and max flow 4.58.
. . .
Duct <632298 Mitered Elbows> has 2 connectors and max flow 8.33.

Set MEP Duct Flow Parameter:
  Set max flow parameter on 43 ducts,
  2 of which have open connectors.
```

Here is
[FlowParam03.zip](zip/FlowParam03.zip) containing
the complete source code with the updated method implementations using LINQ as described above, the Visual Studio solution and the add-in manifest for this add-in.

#### Autodesk University Class Catalog Preview

The
[AU 2013 class catalog preview](http://au.typepad.com/files/au-2013-class-catalog-preview-082613.xls) (2.5 Mb xls)
listing the classes available at Autodesk University in Las Vegas in December is now available for download.

I verified that mine are all present and intact:

- **DV1736** – Cloud-Based, Real-Time, Round-Trip, 2D Revit Model Editing on Any Mobile Device – This presentation demonstrates real-time, round-trip editing of a simplified 2D rendering of an Autodesk Revit intelligent model on any mobile device with no need to install any additional software whatsoever beyond a web browser. How can this be achieved? A Revit software add-in exports polygon renderings of room boundaries and other elements such as furniture and equipment to a cloud-based repository that is implemented using an Apache CouchDB NoSQL database. On the mobile device, the repository is queried and the data rendered in a standard browser using server-side generated JavaScript and SVG. The rendering supports graphical editing, specifically translation and rotation of the furniture and equipment. Modified transformations are saved back to the cloud database. The Revit add-in picks up these changes and updates the Revit intelligent model in real-time. All of the components used are completely open source, except for Revit itself. This is an advanced class for experienced programmers.
- **DV1914** – Revit API Expert Roundtable: Open House on the Factory Floor – Interact with a panel of Autodesk Revit API experts from Autodesk to get answers to your questions and discuss all relevant topics of your choice. If you are writing add-ins for Revit software, then this is the perfect forum to get to know better the people who shape the APIs you work with and to explain your views, ideas, and problems directly face-to-face. Note that prior .NET programming and Revit programming experience is required and that this class is not suitable for beginners.
- **DV2010** – Advanced Revit 2014 API Features and Samples – This class focuses on some of the major new Autodesk Revit 2014 API features. We look at API access to the project browser, dockable panels, copy and paste, command launching, the graphics pipeline, schedule formatting, and additions to the view API including demonstration and discussion of sample code. We also provide an overview of all the new Revit 2014 SDK samples. Note that prior .NET and Revit programming experience is required and that this class is not suitable for beginners.

Since popular classes often fill up quite quickly, you can use this spreadsheet to get started on your selections and help ensure that you can register for the classes you want when registration opens on September 12.

#### Songbird Dies, Long Live Nightingale

Completely unrelated to the Revit API, you may have noticed that I dabble with music now and then and was using Songbird a my main playing and tagging tool on Mac.

Well,
[Songbird shut down](http://techcrunch.com/2013/06/14/songbird-sings-its-last-tune-as-music-service-runs-out-of-money-and-plans-to-shut-down-june-28) as
a company in June, and I now moved on to
[Nightingale](http://getnightingale.com)
that is touted as its
[worthy](http://www.makeuseof.com/tag/nightingale-a-faster-cleaner-cross-platform-fork-of-songbird-music-player)
[successor](http://www.linuxjournal.com/node/1096940)
and looking good to me so far.

I am also using
[VirtualDJ](http://www.virtualdj.com/) when I want to cross-fade nicely from one song to the next in a playlist.