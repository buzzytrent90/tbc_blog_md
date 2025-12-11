---
ai_optimized: true
author: Jeremy Tammik
complexity_score: 7.9
content_type: tutorial
optimization_date: '2025-12-11T11:44:13.571699'
original_url: https://thebuildingcoder.typepad.com/blog/0219_mep_api.html
post_number: 0219
reading_time_minutes: 11
series: mep
slug: mep_api
source_file: 0219_mep_api.htm
tags:
- elements
- family
- filtering
- geometry
- levels
- parameters
- references
- revit-api
- rooms
- schedules
- views
- mep
title: The Revit MEP API
word_count: 2137
---

### The Revit MEP API

By the time you read this, I will be away happily on a one-week vacation in Italy.
I hope you and the blog manage yourselves and behave well until I return.
I scheduled some items to be published in my absence, though.

In previous posts, we already had a look at the
[new MEP-specific API features in Revit 2010](http://thebuildingcoder.typepad.com/blog/2009/06/revit-mep-api.html) and
the classes involved in defining and representing
[MEP connectors](http://thebuildingcoder.typepad.com/blog/2009/04/mep-connectors.html) in
particular.
On August 27th 2009, we held the first
[webcast](http://thebuildingcoder.typepad.com/blog/2009/08/revit-mep-api-webcast.html) dedicated
entirely to Revit MEP programming.
This post provides an abbreviated overview of the complete Revit MEP and the webcast contents with links to available related materials, including the following topics:

- [The Revit MEP API- [MEP model analysis- [Hierarchical systems and connectors- [Electrical systems- [HVAC and plumbing systems- [Sample applications- [Materials and further reading](#7)](#6)](#5)](#4)](#3)](#2)](#1)

#### The Revit MEP API

Revit MEP is a flavour of Revit for work in the mechanical, electrical and plumbing domains. HVAC, i.e. heating, ventilation and air conditioning, is considered part of the mechanical domain. Efficient work in these areas requires strong model analysis tools in addition to read and write access to the components and data used to model the electrical, HVAC and piping systems in Revit MEP. Some of the required MEP specific functionality includes access to the MEP project information, Green Building XML, spaces and zones, the electrical, duct and pipe systems, their components, properties and parameters, methods for element creation and modification, and system traversal and analysis.

Most of the Revit API is generic and applies to all three flavours of the Revit product, i.e. Architecture, MEP and Structure. All three flavours of Revit also include the same .NET assembly named RevitAPI.dll providing API access to third-party applications. However, some specific additional features exist for each of the flavours as well, for instance some room-related functionality in Revit Architecture and access to the analytical and MEP models in Revit Structure and MEP respectively. If API functionality not supported by the currently running flavour is accessed, the call will simply do nothing or return null.

In Revit 2008, no MEP-specific functionality was provided by the API. It was still possible to analyse and manipulate an HVAC system using the generic Revit API methods, for instance by accessing ducts as generic Revit elements and reading and writing their properties using the generic parameter access. We used this to create a sample application for HVAC air terminal analysis and sizing, presented at Autodesk University 2007. Revit 2009 introduced the first MEP-specific API support, including classes for electrical and mechanical equipment, lighting device and fixture, connector, electrical system, space and zone, the MEP model property on family instances, and the ability to create and modify MEP elements. In Revit 2010, the MEP-specific API was a focal point of the API development and greatly enhanced, especially the support for HVAC and piping systems.

#### MEP Model Analysis

Access to the MEP Project Info is provided through the gbXMLParamElem element and its properties, which is accessed through the ProjectInfo.gbXMLSettings property. The available data includes PostalCode, ProjectLocation, BuildingService, BuildingConstruction, GroundPlane, ShadingSurfaces, and ProjectPhase.

Green Building XML or gbXML export is initiated by a call to the Document.Export method:

```
Document.Export( string folder, string name, GBXMLExportOptions );
```

Using spaces instead of rooms, and zones to group spaces, enables a subdivision of the architectural rooms into subspaces required for MEP analysis. The AddSpaceAndZone SDK sample demonstrates the programmatic creation and management of spaces and zones. The FamilyInstance class already had a Room property to determine in which room an instance is located, and now an analogous Space property has been added for MEP usage.

Several model inspection utility methods added in Revit 2010 are especially useful in the MEP domain. One group helps determine the relationships between and contents of volumes, rooms and spaces:

- Room.IsPointInRoom determines if a 3D point is in the volume of a room.- Space.IsPointInSpace determines if a 3D point is in the volume of a space.- GetRoomAtPoint and GetSpaceAtPoint return the room or space containing a given 3D point.

The new ray intersection method can be used to determine proximity of elements:

- FindReferencesByDirection returns an array of elements, faces, and references found when moving through the model from a specified point in a specified direction.

One use of FindReferencesByDirection is demonstrated by the RayTraceBounce SDK sample.

#### Hierarchical Systems and Connectors

Many of the building blocks of an MEP system are represented in Revit by family instances. The generic family instance definition in the Revit API has been enhanced to provide access to additional MEP-specific properties and data, and provide classes to represent systems and connections between system components.

The objects dealt with in the MEP domain are organised into hierarchical structures. The top level node in these structures is an MEP system, represented by the MEPSystem class. This is the base class for the derived classes ElectricalSystem, MechanicalSystem and PipingSystem. The system has properties to access the base equipment or panel, the terminal elements, and the connector manager. The derived classes have many additional domain specific properties.

The Revit 2009 API introduced the MEPModel and Connector classes as fundamentals for programmatically representing and accessing the MEP model. It is accessed through the FamilyInstance.MEPModel property, and one of its main purposes is to provide access to the connector manager.

We have logical and physical connections between MEP components. Logical connections are used in the electrical domain, where wires and cables are not necessarily modelled individually. Physical connections are used in the mechanical and plumbing domains, where the connectors define and transmit sizing and flow information from one part to the next.

A connector is part of an MEP element such as a duct, pipe, fitting, or piece of equipment. The Connector class is used to represent connectors in the project environment. Individual stand-alone connector elements need to be created inside a family to define the component and are represented there by the DuctConnector, PipeConnector and ElectricalConnector classes.

The connector defines geometry, e.g. its oval, round or rectangular shape, and dimensions such as width, height and radius. It also maintains a coordinate system and system data such as domain, system type, flow, direction.

#### Electrical Systems

Electrical systems are organised hierarchically in Revit MEP. The root elements are panels. Each panel is connected to a number of systems, also known as circuits. Each system or circuit consists of a number of circuit elements, some of which may be further panels, and so on, recursively. Cables and wires need not be modelled individually, and the connections between the electrical system components are defined by logical connectors rather than physical ones. The Revit 2009 API introduced the classes ElectricalEquipment, LightingDevice, LightingFixture, ElectricalSystem, specialisations of the base MEPModel and MEPSystem classes, to represent electrical components and systems. These elements all have a connector manager, whose Connectors property returns logical as well as physical type connectors:

```
ElectricalSystem sys;
ConnectorSet connectors = sys.ConnectorManager.Connectors;
```

This makes it very simple to analyse and traverse the electrical system hierarchies similarly to the approach demonstrated by the TraverseSystem SDK sample for duct and piping ones.

#### HVAC and Plumbing Systems

New support for HVAC and plumbing was provided with Revit 2010, which introduced the Autodesk.Revit.MEP namespace containing a wealth of new classes and functionality, as well as moving existing MEP-related classes to the MEP namespace. The main components in HVAC and piping hierarchy are the top level systems, ducts, pipes, fittings and connectors:

- Systems to manage the top level system properties.- Ducts and pipes to define the main flow elements.- Fittings to implement bends and branches in the system.- Connectors to hook up the ducts, pipes and fittings.

The classes MechanicalSystem and PipingSystem introduced in Revit 2010 provide the following top-level system functionality:

- Access to equipment, connectors and system type.- Access to system properties such as flow and static pressure.- Analysis of system contents through the DuctNetwork or PipeNetwork property.

The elements contained within a system are not automatically returned in the flow direction. The TraverseSystem SDK sample demonstrates how to query the connector managers to implement traversal in direction of flow for a given system.

Ducts and pipes are represented by the classes Duct, FlexDuct, Pipe and FlexPipe, derived from the MEP curve type. They provide the following functionality:

- Read access to duct properties, types, and geometry.- Change duct or pipe type.- Move or layout duct or pipe.

The layout functionality can be driven by two points, one point and a connector, or two connectors.

Fittings are represented by family instances, which are created with dedicated methods on the Autodesk.Revit.Creation.Document class for inserting Elbow, Tee, Cross, Takeoff, Transition, and Union elements. The fitting properties, shape and dimensions can be accessed through the FamilyInstance.MEPModel property.

Connectors limited to the electrical realm were introduced in Revit 2009. In Revit 2010, we have connectors for the mechanical and piping domains as well. The connector is an item that is a part of another element such as a duct, pipe, fitting, or equipment. The basic features of connectors and the different classed used to represent them in the family and project contexts were mentioned above. The additional HVAC-specific connector features include properties to read the flow, coefficient and demand. They also provide access to physical properties like Origin, Angle, Height, Width and Radius.

We have read and write access to so-called assigned connector properties. These are connector properties that can be overridden by the user and are dependent on the overall network condition, including characteristics like Flow, Flow Config, Coefficient, Loss etc. These depend on the system being well connected, the configuration of the family within the system, and how the family is constructed.

Connector size and location can be modified. Connectors can be disconnected to insert a new fitting into a network or connect two ducts into a transition.

- The family connector elements define Flow, Flow Configuration, Coefficients, and Loss Method.- Read duct, pipe, and fitting connector properties such as Flow, Coefficient, and Demand.- Access physical connector properties e.g. Origin, Angle, Height, Width, Radius.- Change connector size and location.- Connect and disconnect.

The creation document provides a number of methods for the creation of HVAC and plumbing elements within the project environment, including new mechanical or piping systems, new duct and pipe elements, and new fittings. In the context of a family, the different types of duct, pipe and electrical connectors are represented by specific elements that can be created by the FamilyItemFactory class, accessed through the Document.FamilyCreate property.

#### Sample Applications

The MEP-specific sample applications include both those provided in the standard Revit SDK as well as a non-SDK one created for this presentation on the RME API.

The introduction of an MEP specific API with Revit 2009 was accompanied by two SDK samples:

- AddSpaceAndZone demonstrates handling the new space and zone elements.- PowerCircuit demonstrates manipulation of electrical power circuits.

The Revit 2010 SDK includes the following new MEP-specific SDK sample applications:

- AutoRoute: Route a set of ducts and fittings between a source, the air supply equipment, and the sink, two air outlet terminals.- AvoidObstruction: Detect and resolve obstructions between ducts, pipes, and beams.- TraverseSystem: Traverse a well-connected mechanical or piping system in the direction of flow.- CreateAirHandler: Create an air handler and add connectors.

The first three demonstrate RME API features in the project context, the fourth uses the family API.

Finally, we provide a non-SDK sample application named mep. It implements some HVAC oriented commands making use of the generic Revit API to analyse and manipulate duct elements for automatic air terminal sizing, and others which implement an electrical sample for analysis and display of an electrical system in a tree view reproducing the Revit system browser structure or showing the full connection hierarchy tree:

- HVAC air terminal analysis and sizing.- Electrical system connectivity analysis and hierarchical display.

#### Materials and Further Reading

A more extensive document discussing the Revit MEP API and the related samples in deeper detail as well as other useful material is provided with the recording of Revit MEP Programming Webcast held on August 27th 2009, available through the
ADN or Autodesk Developer Network
[API training schedule](http://www.adskconsulting.com/adn/cs/api_course_sched.php)
(also accessible via
[www.autodesk.com/apitraining](http://www.autodesk.com/apitraining)),
filter for 'Revit MEP API', and click on the
[download](http://download.autodesk.com/media/adn/RevitMEPAPI_27Aug09.zip)
link.
This material is accessible to all developers, regardless of ADN membership.