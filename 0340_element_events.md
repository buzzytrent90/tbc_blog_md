---
ai_optimized: true
author: Jeremy Tammik
complexity_score: 4.2
content_type: news
optimization_date: '2025-12-11T11:44:13.773751'
original_url: https://thebuildingcoder.typepad.com/blog/0340_element_events.html
post_number: '0340'
reading_time_minutes: 8
series: elements
slug: element_events
source_file: 0340_element_events.htm
tags:
- csharp
- elements
- family
- filtering
- geometry
- levels
- parameters
- revit-api
- transactions
- views
- walls
- windows
title: Element Level Events
word_count: 1574
---

### Element Level Events

From my point of view, one of the most important
[enhancements to the Revit API](http://thebuildingcoder.typepad.com/blog/2010/03/revit-2011-is-coming.html) is
the availability of element level events.
This allows an application to react to modifications to the database, including tracking changes to individual elements.
The Revit 2011 API provides not just one but two separate mechanisms to subscribe to element level notification events:

- [Elements changed event](#1)- [Dynamic model update](#2)

#### Elements Changed Event

The new Application.DocumentChanged event is raised after every transaction is committed, undone, or redone.
This is a read-only event, designed to allow an application to keep external data synchronised with the Revit database state.
To update the Revit database in response to changes in elements, use the Dynamic Model Update framework described below.

The event handler receives a DocumentChangedEventArgs instance as an input argument which provides information on the operation performed and the document and elements involved.
Some of its most important properties and methods are therefore the following:

- Operation: the operation associated with this event.- GetDocument: the document associated with this event.- GetAddedElementIds: the set of elements newly added to the document.- GetDeletedElementIds: the set of elements that were deleted.- GetModifiedElementIds: the set of elements that were modified.- GetTransactionNames: the names of the transactions associated with this event.

#### ChangesMonitor SDK Sample

The use of the elements changed event is demonstrated by the ChangesMonitor Revit SDK sample.
It implements a data table to log all the changes made to the database, and a modeless dialogue ChangesInformationForm to display its contents.
The main modules of the sample project are ChangesMoniotor.cs and ChangesInfoForm.cs:

- ChangesMoniotor.cs:
  This file contains a class ExternalApplication which implements the IExternalApplication interface and the sample entry point.
  It creates a modeless dialog to track the changed elements' information.
  This file also defines an external command class Command which inherits from the IExternalCommmand interface.
  Its function is to redisplay the information dialogue if it has been closed.- ChangesInfoForm.cs:
    This file defines a ChangesInformationForm form class which displays the changed element information in a DataGridView in a modeless dialogue.
    The information includes the change type (Added, Deleted, and Modified), and the modified element id, name, category name, and document owning it.

The external application initialises these components and registers to the Application.DocumentChanged event in its OnStartup method like this:
```csharp
public Result OnStartup(
  UIControlledApplication application )
{
  // initialize member variables.

  m\_CtrlApp = application.ControlledApplication;
  m\_ChangesInfoTable = CreateChangeInfoTable();

  m\_InfoForm = new ChangesInformationForm(
    ChangesInfoTable );

  // register the DocumentChanged event

  m\_CtrlApp.DocumentChanged += new EventHandler<
    Autodesk.Revit.DB.Events.DocumentChangedEventArgs>(
    CtrlApp\_DocumentChanged );

  // show dialog

  m\_InfoForm.Show();

  return Result.Succeeded;
}
```

Here is an example of the information that it collects and presents:

![ChangesMonitor modeless dialogue](img/SdkChangesMonitor.png)

ChangesMonitor also implements an external command to redisplay the modeless dialogue, in case you closed it and want to reopen it again.
By default, it is opened automatically on start-up.

#### Dynamic Model Update

As mentioned above, the Application.DocumentChanged event is a read-only event, so it does not support modifications to the Revit database as a reaction to the changes that triggered it.
For that, you can make use of the much more powerful dynamic model update mechanism.
This is what the What's New section in the Revit API help file has to say about the dynamic model update:

Dynamic model update offers the ability for a Revit API application to modify the Revit model as a reaction to changes happening in the model.
This facility is offered through implementation of updaters.
The updater interface offers the ability to implement a method that is informed of the scope of changes that triggered the update.

In order to 'subscribe' an updater to the right set of changes, the updater should be assigned one or more update triggers.
Update triggers are combinations of 'Change Scope' and 'Change Type'. Change Scope may be either an explicit list of element ids in a document, or an implicit list of elements communicated via an ElementFilter. Change Type represents one of an available list of possible changes, including element addition, deletion, and modification of geometry, parameters, or any property of the element.

More detailed information on the dynamic model update capabilities are provided in the Developer Guide chapter 25.

One important thing to note is that this facility allows the application to perform its changes to react to the notification within the same transaction as the one that triggered the changes.

Since many different updaters may try to modify things which will trigger new updaters, the execution sequence is important.
A facility for defining your execution priority is included in the system.

The use of the dynamic model update is demonstrated by two different SDK samples, DynamicModelUpdate and DistanceToSurfaces.
The latter also demonstrates the new analysis visualization framework and is therefore located in the AnalysisVisualizationFramework subfolder.
Both of these are implemented as external applications which set up their updater and trigger during the OnStartup or DocumentOpened events.

#### DynamicModelUpdate SDK Sample

The use of the dynamic model update is demonstrated by the DynamicModelUpdate Revit SDK sample.
It is also implemented as an external application and designed to work in a specific included sample model AssociativeSection.rvt.
It shows how you can use this mechanism to maintain the relative position between elements, in this case a window located in a wall and a section view positioned to display the cut through the window and the wall.
This sample uses the hard-coded element id of the section view marker to update its position:

![Associative section view sample model plan view](img/SdkDynamicModelUpdate1.png)

Here is the result of the section view displaying the window in the wall:

![Associative section view](img/SdkDynamicModelUpdate2.png)

These are the detailed steps of the sample implementation:

1. During Revit start-up:
   1. Register the SectionUpdater class.- Add a trigger so that the SectionUpdater class will be invoked for any geometry change to a window.- When the geometry of a window changes:
     1. Move the section marker (using the hard-coded element id for the sample model) to a new location based on the distance that the window moved in the X, Y, and Z directions, which are stored as parameters in the window family.- Update the values of the window's X, Y, and Z parameters based on the location point of the window.

If the window is moved, the section view is updated to move with it, so that it continues to display the correct cut.
Obviously, if the wall moves, the window will move with it, too, and that will again update the section location.

#### DistanceToSurfaces SDK Sample

The use of the dynamic model update is also demonstrated by the DistanceToSurfaces SDK sample, although it is primarily intended to highlight the new analysis visualization framework, which allows an application to display analysis results graphically by painting the surfaces of Revit elements.
Once again, it is written to work in a specific sample model.
The model includes a sphere object represented by a family instance, i.e. an insertion of the sphere.rfa family.
It calculates and displays the distance from the sphere to all surfaces of all walls and mass elements in the model.

On start-up, the external application registers a DocumentOpened event handler.
This in turn creates a trigger that will execute when walls, masses, and family instances in the project change.
When this occurs, the application calculates the distance from the sphere to several points on each face.
These distances are used as values for the analysis visualization display.
Here are the individual steps in more detail:

1. On start-up, register a DocumentOpened event. When a document is opened:
   1. Check for the presence of the sphere family.- Check for the presence of a 3D view named 'AVF'.- Create a GetChangeTypeGeometry trigger that will activate when a wall, mass, or family instance changes.- When this trigger occurs:
     1. Find the XYZ location of the sphere.- Get the existing spatial field manager object, or create one if one does not already exist.- Create a face array containing every face of a wall or mass in the project, including in-place geometry.- Calculate the distance between the sphere's origin and several points on each face.- Display these distances as analysis results on the faces by updating the spatial field primitive.

This is what the initial DistanceToSurfaces.rvt sample model looks like:

![DistanceToSurfaces sample model](img/SdkDistanceToSurfaces1.png)

If any one of the relevant elements is moved, the trigger reacts and the updater is called, which uses a filter to retrieve all walls and mass elements and the SpatialFieldManager to paint their faces depending on the distance to the sphere element:

![DistanceToSurfaces painting the distances](img/SdkDistanceToSurfaces2.png)

Using Manage > Additional Settings > Analysis Display Styles, the analysis result colour mapping can either be set up to use ranges, as shown above, or a smooth gradient like this:

![DistanceToSurfaces distance gradiant](img/SdkDistanceToSurfaces3.png)

Developers have been clamouring for element level events ever since the first events were introduced in the Revit API, and even more so when the events were
[enhanced for the Revit 2010 API](http://thebuildingcoder.typepad.com/blog/2009/04/new-2010-events.html).
In previous releases, only application and document level events were available.
It will be very exciting to see what powerful new applications can be created to make use of this!