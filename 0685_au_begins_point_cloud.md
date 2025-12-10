---
post_number: "0685"
title: "AU Begins and Point Cloud Overview"
slug: "au_begins_point_cloud"
author: "Jeremy Tammik"
tags: ['csharp', 'elements', 'filtering', 'revit-api', 'selection', 'views', 'walls']
source_file: "0685_au_begins_point_cloud.htm"
original_url: "https://thebuildingcoder.typepad.com/blog/0685_au_begins_point_cloud.html"
---

### AU Begins and Point Cloud Overview

Once again I left the desert and the climbing in Red Rock and entered the air conditioned world of conference land.
This year I was pretty leisurely about climbing, although I did at least a few climbs every day.
I slept well and long, hiked in to Red Rock from the highway, read a book in the desert sunshine, and finally met up with people in the late morning.

A memorable experience on my birthday on Saturday was meeting another Jeremy!
We did some climbs together on The Magic Bus: Electric Koolaid 5.9+, Neon Sunset, Zipperhead and Technicolor Sunrise, 5.8.
I never previously had so much direct personal interaction with another Jeremy, so that was very special.
On Sunday I spent time with Christine, David and Matt, and we did one single climb on Circus Wall, High Wire 5.10-.

Then it was farewell to nature, hello hotel.
I really enjoyed meeting my ADN DevTech colleagues for dinner last night.
And now to get ready for the conference.
It is five o'clock Monday morning, and I am still waking up early.
I think I am well prepared, actually, for a change.

As I
[already mentioned](http://thebuildingcoder.typepad.com/blog/2011/09/revit-and-aec-api-classes-at-autodesk-university.html),
I am presenting a virtual class on the Revit MEP API and a lecture and a hands-on lab on the extensible storage API, besides all the meetings, other classes, and DevLab that I need to attend to.
Anyway, time for me to get going now.

For you people not attending Autodesk University, here is some other interesting information to keep you busy in the meantime, an overview of the Point Cloud Engine API by my colleague Katsuaki Takamizawa:

#### Autodesk Revit Point Cloud Engine API

One of the new features of Revit 2012 is the point cloud functionality and the API for making use of it programmatically.

You can now insert a point cloud file into a Revit project, display them in model views, and select/move/rotate/copy them. Laser scanners and point cloud technology has revolutionized the way we can sample model information, for example, when a project involves an existing building. The application area of this technology is expected to grow.

There are three types of API related to Point Cloud feature:

- Client API – create and manipulate point cloud instance within Revit.- Custom Point Cloud Engine API – supply points in a point cloud to Revit.- Point Cloud Indexer API – convert data from custom format to indexed (.pcg) format used by Revit and other Autodesk products.

The first, Client API is to handle an instance of point cloud. It also offers tools to aid interacting with the users, such as selection filters and highlighting and isolating a portion of point cloud.

Revit software comes with .pcg Point Cloud Engine, which is the same as AutoCAD software.
However, any other format can be supported in two ways, using either the second or the third API.
The second API, Point Cloud Engine API, provides a plug-in to offer a different engine instead of built-in pcg engine.

The third, Point Cloud Indexer API, is for converting data with a different format into pcg format.
Revit software offers other formats, such as .las, .xyb, .xyz, .pts, .fls and .fws, in this method.
Indexer API by itself can be a topic for a separate article and we are not going to cover it here.
You can find a Point Cloud Indexer Sample code written by Adam Nagy on the ADN extranet.
This is C/C++ API and a completely separate executable that Revit software will call as an out of process.

In the following, we are going to focus on the second, the Point Cloud Engine API.

#### Custom Engine Implementation

The Revit Point Cloud Engine API provides a set of interfaces which allow you to create custom point cloud engines to feed your point cloud data to Revit software for users to work on or to process further by plug-ins.
The SDK sample PointCloudEngine implements simple custom point cloud engines to demonstrate the API usage. We are going to look at this sample to give you a big picture of what point cloud engine API is about and how it can be defined.

In order to create a custom point cloud engine using Revit API, you must implement the following interfaces:

- IPointCloudEngine – controls the link between Revit software and a custom Point Cloud engine.- IPointCloudAccess – provides access to an individual point cloud.- IPointSetIterator – called when iterating through a set of points on the engine.

A custom point cloud engine must implement IPointCloudEngine interface. An instance of this interface must be registered using PointCloudEnginesRegistry. This interface is like an entry point for Revit software.

Revit gets the instance of IPointCloudAccess interface from the IPointCloudEngine.CreatePointCloudAccess method.
The IPointCloudAccess.ReadPoints method gets a single set of points specified in a buffer.
The IPointCloudAccess.CreatePointSetIterator method returns an instance of IPointSetIterator interface.
IPointSetIterator.ReadPoints iterates over blocks of the point cloud.
Revit reads points using these methods for display, selection, snapping and other operations with the point cloud.

#### Registering an Engine

In the sample, three custom point cloud engines are registered using the PointCloudEngineRegistry class RegisterPointCloudEngine method in the application OnStartup method:
```csharp
  // Predefined engine (non-randomized)

  IPointCloudEngine engine
    = new BasicPointCloudEngine(
      PointCloudEngineType.Predefined );

  PointCloudEngineRegistry.RegisterPointCloudEngine(
    "apipc", engine, false );

  // Predefined engine with randomized points at the cell borders

  engine = new BasicPointCloudEngine(
    PointCloudEngineType.RandomizedPoints );

  PointCloudEngineRegistry.RegisterPointCloudEngine(
    "apipcr", engine, false );

  // XML-based point cloud definition

  engine = new BasicPointCloudEngine(
    PointCloudEngineType.FileBased );

  PointCloudEngineRegistry.RegisterPointCloudEngine(
    "xml", engine, true );
```

The first two engines are non-file based engines registered with arbitrary identifies (which you can decide). The third one is a file-based engine registered with a file extension it supports. Let's look at the engine types next.

#### File-Based Point Cloud Engine

A file-based point cloud engine processes the points defined in an external file.
In this sample, points are defined in a custom format in a XML file.
Once a file-based point cloud engine is created and registered with Revit with a file extension, the file can be inserted from the UI Link Point Cloud dialog, which you can access from: [Insert] tab > [Link] panel > [Point Cloud] button.
File-based point cloud engines can extend types of point cloud files Revit software can insert:

![Point cloud file formats with a custom engine registered](img/kt_point_cloud_18.jpg)

In this specific sample, points are grouped and presented as cells in the Tower.xml file provided.
A cell represents a rectangular 3D box defined by two points at the lower left and upper right.
Points are created along with these edges of the box.
A colour is assigned to a cell so that points for the cell have the same colour.
Tower.xml file defines three cells in a custom file format as shown below.
The first cell represents the largest rectangular box and two others represent the smaller boxes:
```csharp
<?xml version="1.0" encoding="utf-8"?>
<PointCloud>
  <Scale value="2.5"/>
  <Cell>
    <LowerLeft X="-30" Y="-30" Z="0" />
    <UpperRight X="30" Y="30" Z="200" />
    <Color value="#000000" />
    <Randomize value="True" />
  </Cell>
  ...
</PointCloud>
```

Here is the point cloud created in a Revit model by inserting Tower.xml.
Note: for the purpose of demonstration, it is using the Randomize property in Tower.xml so that the custom file-based point cloud engine adds some random offset to the points' location to mimic point cloud.

![File-based point cloud imported](img/kt_point_cloud_19.jpg)

#### Non-File Based Point Cloud Engine

The source of point information is flexible for non-file based point cloud engines. Non-file based point cloud engines allow the direct integration of point clouds to Revit software. In this sample, non-file based point cloud engines are designed to use points from hard-coded cells. However, it can be much more creative.
For example, non-file based point cloud engines could be designed to access a server to retrieve point information obtained by a laser scanning.
It can offer another powerful tool as you can imagine.

Non-file based point access is only provided through Revit API. There are three commands demonstrating this feature in the sample:

- Add predefined instance - creates points along with edges of hardcoded cells.- Add randomized instance - adds random offset for points.- Add randomized instance at transform - applies a rotation to the cells.

Here is a sample image from non-file based point cloud command:

![Non-file-based point cloud imported](img/kt_point_cloud_20.jpg)

#### Exporting a Point Cloud

The sample also demonstrates export with non-file based point cloud engines.
The engine simply writes out points to an xml file.
You can run this feature from Serialize point cloud (utility) in Add-in tab.
The following is the excerpt from the main functionality:
```csharp
  PredefinedPointCloud cloud
    = new PredefinedPointCloud( "dummy" );

  XDocument doc = new XDocument();
  XElement root = new XElement( "PointCloud" );
  cloud.SerializeObjectData( root );
  doc.Add( root );

  TextWriter writer = new StreamWriter(
    @"c:\serializedPC.xml" );

  doc.WriteTo( new XmlTextWriter( writer ) );

  writer.Close();
```

Where XDocument represents an XML document and XElement an XML element.
They are a part of System.Xml.Linq.XDocument namespace.

I hope it gives you a good starting point for your exploration with the Point Cloud Engine API.

Many thanks to Katsu-san for this helpful introduction and overview!