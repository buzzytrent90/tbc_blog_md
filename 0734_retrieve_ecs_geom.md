---
ai_optimized: true
author: Jeremy Tammik
complexity_score: 3.5
content_type: qa
optimization_date: '2025-12-11T11:44:14.482102'
original_url: https://thebuildingcoder.typepad.com/blog/0734_retrieve_ecs_geom.html
post_number: '0734'
reading_time_minutes: 4
series: general
slug: retrieve_ecs_geom
source_file: 0734_retrieve_ecs_geom.htm
tags:
- csharp
- elements
- family
- geometry
- revit-api
title: Retrieve Geometry in Element Coordinate System
word_count: 813
---

### Retrieve Geometry in Element Coordinate System

I am travelling rather further afield than usual today, and actually have been doing so all day yesterday as well.
My normal paths remain inside of Europe, with occasional trips once or twice a year to the USA, for AEC DevCamp and Autodesk University.
Now, exceptionally, I am on my way to Melbourne, Australia, to present a Revit API training and DevCamp next week.

Meanwhile, here is an interesting little issue brought up by Olli Kattelus, software developer of
[MagiCAD](http://www.magicad.com) for
[Progman Oy](http://www.magicad.com/en/content/welcome-house-progman) in
the context of IFC export of MEP ducts and pipes.
It is a teeny weeny bit tricky and can yet be solved quite easily using the geometrical tools provided by the Revit API.

**Question:** My problem in brief:

I can't get positions of the geometry primitives of the duct/pipe etc. in the instance's own coordinate system. Or is it possible?

More in-depth explanation:

For standard family instances such as air terminal, fitting, etc., I CAN retrieve the geometry primitives (faces etc.) and their coordinates in the symbol's own coordinate system.
I also can transform them to the global coordinate system by using the transform provided by the GeometryInstance class.

For system families, however, things are different.
The first obstacle is that ducts/pipes etc. don't have any GeometryInstance as their geometry.
When I fetch their geometry primitives, they are returned in the global coordinate system.

I would really need to get them also in the "symbols" own coordinate system.

The reason I need this is that I'm working on an IFC export.
In IFC, the geometry is exported only once per object type, or at least that is much preferred.
The coordinates of geometry must be therefore be given in the element coordinate system because when the IFC file is read, the location and orientation of the object tells the IFC interpreter where to place the resulting geometry.

If the geometry is exported using the global coordinate system, things go wrong.
Some geometry transformation methods are provided by the ExporterIFCUtils classes, but unfortunately I can't use them because I'm not using the Revit API to create the export.

**Answer:** Here is a suggestion for an approach:

1. Determine the duct or pipe start and end points in the global coordinate system GCS.- Define what you mean by "instance's own coordinate system" ICS, for instance start point equals origin, duct or pipe direction equals X axis, something or other equals the Z axis etc.- Determine the transformation T from the GCS to the ICS.- Obtain the duct or pipe geometry primitives and apply T to them to place them in ICS.

Probably the easiest way to determine the transform from GCS to ICS is to first create the inverse one from ICS to GCS.

This can then easily be inverted using the Revit API Transform.Inverse property.

To create the transformation from ICS to GCS is also normally easy, since you can decide for yourself how you consider it should be set up.
For instance, for a duct, you could say that the ICS is located at its origin and has cardinal X, Y and Z axes.

To define the appropriate Revit API Transform instance, you just have to plug in the coordinates of the GCS start and end points appropriately into the Transform basis properties, cf. the description of the Transform.Basis property in the Revit API help file.
You may have to fiddle around a bit to get it right, but it should work.

**Response:** Thanks A LOT for your instructions!
I had kind of tried this at some previous point but I wasn't sure whether it could work this way.
After your encouragement I had enough 'faith' to try until it worked :-)

As you said, I had to 'fiddle' a little bit with those but eventually managed to get it work in most of the cases.

I ended up with this kind of scenario:

For all the objects (generally system families) whose geometry primitives are required in ICS, I define origin, direction and normal as follows similar to the approach used in IFC, using the given element and its 'end connector' coordinate system:
```csharp
coordSys = m\_revElem
->endConnectors[0]
->CoordinateSystem;
org = coordSys->Origin;
dir = coordSys->BasisZ->Negate();
norm = coordSys->BasisY->Negate();
```

Sorry that this is not pure Revit API code, but as said 'm\_revElem->endConnectors[0]' simply means some connector on the element.

To get the geometry to work correctly in our IFC output, I created the following transform:
```csharp
Transform ^tf = Transform::Identity;
tf->BasisX = coordSys->BasisZ->Negate();
tf->BasisY = coordSys->BasisX;
tf->BasisZ = coordSys->BasisY->Negate();
tf->Origin = coordSys->Origin;
tf = tf->Inverse;
```

So this really DID help a lot.
Thanks again for this!!