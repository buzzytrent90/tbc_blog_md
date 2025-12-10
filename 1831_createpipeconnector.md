---
post_number: "1831"
title: "Createpipeconnector"
slug: "createpipeconnector"
author: "Jeremy Tammik"
tags: ['elements', 'family', 'geometry', 'parameters', 'references', 'revit-api', 'sheets', 'windows']
source_file: "1831_createpipeconnector.md"
original_url: "https://thebuildingcoder.typepad.com/blog/1831_createpipeconnector.html"
---

### Autorouting and Reference for CreatePipeConnector
Today, let's pick up two recent MEP related discussions, on creating a new pipe connector for a hydraulic fitting family and on automatic pipe system routing, and a couple of other odd items of interest:
- [The names they are a-changin](#2)
- [Getting a `ReferencePlane` for `CreatePipeConnector`](#3)
- [Auto-routing a pipe system between plumbing fixtures](#4)
- [Handling dialogue and failure messages](#5)
- [Retrieving a geometry reference](#6)
#### The Names They are A-Changin
In these times of accelerating change, the name of my team has changed as well.
From now on, I now work in the Autodesk DAS team:
- RDP Registered Developer Program 1987-1995
- ADN Autodesk Developer Network 1995-2015
- FPD Forge Partner Development 2015-2020
- DAS Developer Advocacy and Support 2020+
![Developer Advocacy and Support](img/fpd_das.png "Developer Advocacy and Support")
#### Getting a ReferencePlane for CreatePipeConnector
I had an interesting and fruitful conversation with Edouard Vnuk in
the [Revit API discussion forum](http://forums.autodesk.com/t5/revit-api-forum/bd-p/160) thread
on [CreatePipeConnector from ReferencePlane](https://forums.autodesk.com/t5/revit-api-forum/createpipeconnector-from-referenceplane/m-p/9396013):
\*\*Question:\*\* I would like to create a connector from a reference plane.
The `CreatePipeConnector` function requires a `PlanarFace`.
Is there another function, or how can I transform a `ReferencePlane` into `PlanarFace`?
\*\*Answer:\*\* The [ReferencePlane documentation sample code](https://www.revitapidocs.com/2020/e7003ec7-1dbe-50a2-fb3d-a83a5a3b5b9f.htm) shows
how to call GetPlane to retrieve the DB Plane.
More to the point, the CreateAirHandler SDK sample to how to use the `CreatePipeConnector` method.
When in doubt about how to call a Revit API method, one of the first places to always consult is the collection of Revit SDK samples.
That step often helps and may save time and effort for you and others.
\*\*Response:\*\* I tried several codes without success.
First code:

```
  Autodesk.Revit.DB.Plane plane
    = Reference_plane.GetPlane();

  ConnectorElement connector = ConnectorElement
    .CreatePipeConnector( family_document,
      PipeSystemType.Global, plane );
```

However, I can't compile this code, because the parameter must be a reference of planar face, and I don't know how to get it from the plane.
Second code:

```
  ConnectorElement connector = ConnectorElement
    .CreatePipeConnector( family_document,
      PipeSystemType.Global,
      Reference_plane.GetReference() );
```

This throws an exception during execution: The reference is not a planar face. Parameter name: planarFace.
These are the user interface steps I would like to complete programmatically:
![Pipe connector placement user interface](img/pipe_connector_ui_1.jpg "Pipe connector placement user interface")

![Pipe connector placement user interface](img/pipe_connector_ui_2.jpg "Pipe connector placement user interface")

![Pipe connector placement user interface](img/pipe_connector_ui_3.jpg "Pipe connector placement user interface")

![Pipe connector placement user interface](img/pipe_connector_ui_4.jpg "Pipe connector placement user interface")
I looked the CreateAirHandler SDK sample example.
However, if I understood correctly, it creates the connectors from an extruded volume that provides the required planar faces.
\*\*Answer:\*\* Good. I see your point.
Such situations arise regularly.
You do have a real existing surface somewhere in the model that you can mount your connector on, don't you?
Otherwise, you are modelling something that cannot be built.
The existing surface is part of some BIM element geometry.
You can identify the appropriate element and retrieve its geometry from the document by calling its Geometry property and providing an Options object with ComputeReferences set to true.
Then, iterate through all its surfaces to identify the one you need.
Et voila, that surface is equipped with a reference that you can use to create the connector.
\*\*Response:\*\* What you told me inspired me to think about it.
I am creating a converter.
And so, I create automatically families from basic geometric shapes.
To add my connectors that do not rest on an existing face, I artificially added a cylinder that I masked where I wanted a connector.
Here's what I get :
![Pipe connectors in family](img/pipe_connector_in_family.jpg "Pipe connectors in family")

Pipe connectors in family

![Pipe connectors in project](img/pipe_connector_in_project.jpg "Pipe connectors in project")

Pipe connectors in project

Thank you so much.
Many thanks to Edouard for raising this, the in-depth discussion and sharing his clean solution!
#### Auto-routing a Pipe System Between Plumbing Fixtures
The other item I would like to pick up here is
the [Revit API discussion forum](http://forums.autodesk.com/t5/revit-api-forum/bd-p/160) thread
on [auto routing a pipe system between plumbing fixtures](https://forums.autodesk.com/t5/revit-api-forum/auto-route-a-pipe-system-between-plumbing-fixtures/m-p/9387502):
\*\*Question:\*\* I'm wondering if there is something within the API that will allow me to place plumbing fixture families and then generate a plumbing layout like you can in Revit by using the "Create Plumbing System --> Generate Layout" or is this something I have to do one pipe, one connector at a time?
![Select elements and click create system](img/auto_route_pipe_system_1.png "Select elements and click create system")

Select elements and click create system

![Generate layout button](img/auto_route_pipe_system_generate_placeholder_button.png "Generate layout button")

Generate layout button

![Auto generated pipes](img/auto_route_pipe_system_generated_pipes.png "Auto generated pipes")

Auto generated pipes

\*\*Answer:\*\* You
can [create a placeholder piping system programmatically](https://thebuildingcoder.typepad.com/blog/2011/07/mep-placeholders.html) as well.
Or you can create a piping system right away.
Here is a series of samples that create a minimal bunch of pipes for
a [rolling offset in lots of different ways](http://thebuildingcoder.typepad.com/blog/2014/01/final-rolling-offset-using-pipecreate.html).
\*\*Response:\*\* I am assuming from your response that if I want to connect from say, a water closet to a water heater, with pipes, I will have to explicitly call PipeCreate for each pipe, find connectors, insert each fitting... and then that process over and over until they connect?
Is there a better work flow for what I'm trying to accomplish?
Something similar to what is in the Revit user interface?
\*\*Answer:\*\* The discussions I pointed out already explain everything and cover your question in full.
Actually, here are some additional relevant discussions of this topic:
- [Connector, neighbour, conduit, transition](https://thebuildingcoder.typepad.com/blog/2018/03/connector-neighbour-conduit-transition.html)
- [MEP ductwork and changing pipe direction](https://thebuildingcoder.typepad.com/blog/2019/08/mep-ductwork-and-changing-pipe-direction.html)
The short answer is simply:
- You can go any way you like.
- You can just insert the pipes, no fittings. Then, if you connect the pipe connectors, Revit will automatically select and place and connect the appropriate fittings according to your routing preferences and adjust the pipe endpoints so they fit.
- You can just insert all the fittings, no pipes. Then, if you connect the fitting connectors, Revit will automatically place and connect the appropriate pipes between them.
That is the gist of what I learned researching and implementing the rolling offset.
#### Handling Dialogue and Failure Messages
A recurring question on how to handle dialogue and failure messages keeps popping up when driving Revit programmatically, e.g.,
on [saving families out via Revit API](https://stackoverflow.com/questions/60831658/saving-families-out-vie-revit-api):
\*\*Question:\*\* I am collecting all the families in a project and saving them out via the API using

```
  familyDocument.SaveAs(fileName);
```

Is there a way to catch the following dialogue box and perform an action?
![Failure message](img/failure_message.png "Failure message")
For instance, record the warning and close the dialogue box?
\*\*Answer:\*\* The Revit API offers two different mechanisms to react to and handle dialogue and failure messages:
- The [DialogBoxShowing event](https://www.revitapidocs.com/2020/cb46ea4c-2b80-0ec2-063f-dda6f662948a.htm) and
- The [Failure API](https://www.revitapidocs.com/2020/d0795bd6-f092-90f2-5c2c-3876e616454c.htm).
If all of these fail, a third mechanism is provided by the Windows API, which enables hooking into and reacting to almost any system event, including a dialogue showing.
All three approaches are discussed and compared by The Building Coder in the topic group
on [detecting and handling dialogues and failures](https://thebuildingcoder.typepad.com/blog/about-the-author.html#5.32).
#### Retrieving a Geometry Reference
Let's round off with a quick question
on [getting a reference to the upper face on rebar 51](https://forums.autodesk.com/t5/revit-api-forum/get-reference-upper-on-rebar-51/m-p/9396627):
\*\*Question:\*\* This line of code works on some rebar '51' to retrieve a reference to the upper face, but not all:

```
var srep = $"{rebar51.UniqueId}:2000000:{1002000+typ}:LINEAR";

var refr = Reference.ParseFromStableRepresentation(
  rebar51.Document, srep );
```

Is there a simpler method to get a `Reference` to the upper top face of the rebar form '51'?
\*\*Answer:\*\* You can ask the rebar for its geometry using
the [Element.Geometry property](https://www.revitapidocs.com/2020/d8a55a5b-2a69-d5ab-3e1f-6cf1ee43c8ec.htm).
It takes an `Options` argument in which you can specify `ComputeReferences` = `true`.
Iterate over the geometry solids and their faces, pick the face that you want, e.g., based on its normal vector (pointing upwards) and vertex locations (maximal Z values), and use the reference that it comes equipped with.