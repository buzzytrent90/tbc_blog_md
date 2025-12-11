---
ai_optimized: true
author: Jeremy Tammik
complexity_score: 1.5
content_type: qa
optimization_date: '2025-12-11T11:44:14.697365'
original_url: https://thebuildingcoder.typepad.com/blog/0833_ifc_area_enhancement.html
post_number: 0833
reading_time_minutes: 4
series: general
slug: ifc_area_enhancement
source_file: 0833_ifc_area_enhancement.htm
tags:
- csharp
- elements
- parameters
- revit-api
- rooms
- vbnet
- views
- walls
title: IFC Export Area Calculation Enhancement
word_count: 744
---

﻿

### IFC Export Area Calculation Enhancement

Here is a typical question that may arise when working with IFC export and shows the importance of the extension possibilities provided by the
[IFC export open source project](http://thebuildingcoder.typepad.com/blog/2011/09/revit-ifc-exporter-released-as-open-source.html):

**Question:** How are the areas and volumes of walls, rooms, floors, etc. calculated for the IFC export?
Are any standards being used?

Some examples of values and parameters in question when we export IFC files from Revit:

- For rooms IfcSpace under BaseQuantities: GrossFloorArea, NetFloorArea; these values are the same in my trials.- Base quantities of walls are missing for IfcWall but present in IfcWallStandardCase; why is this?- For IfcWall under dimensions (IfcPropertySet): Area, Volume, Length.- For IfcWallStandardCase under BaseQuantities (IfcElementQuantity): Height, Length, Width, GrossSideArea, GrossFootPrintArea, GrossVolume.

Some other CAD products define other parameters as well; for example, a wall may provide both GrossSideArea and NetSideArea, GrossVolume and NetVolume.

Can we get this from Revit also?
Would this require us to modify the export, e.g. through the open source project, calculate net parameters, and so on?

**Answer:** Good questions!

- Regarding the rooms IfcSpace BaseQuantities GrossFloorArea and NetFloorArea: I think currently they point to the same internal area, which of course is not ideal.
  Revit only has one type of room area calculation at a time, so it is a bit difficult to get multiple areas from the same room.- Some of the wall areas are calculated based on the extrusion that is generated on output.
    Generally, for IfcWall, there is no extrusion, so we can't calculate the particular values.
    That said, we can re-examine to see if there are more properties that can be retrieved directly from the original Revit element.- Regarding 'more parameters from other products': I'll summarize how it currently works, and what could be done:
      1. We generally export the parameters that Revit already calculates.
         The IFC export intentionally does not implement a lot of its own 'QTO' (Quantity Take-Off) calculations.- Since the IFC code was originally written for QTO, it is possible that more Revit quantities have been added to the project since then.
           Adding to the open source project would easily write out those quantities to the IFC file.- For others, we would have to do some calculations.
             These could be done as part of the open source project, requested from the Revit development team, or implemented as a separate Revit add-in, e.g. by adding custom parameters or extensible storage data to the model, which the IFC export could pick up and handle if it is found.

If you are thinking of amending the open source code and you make changes that you think would be beneficial to the entire community, we'd appreciate if you considered 'donating' your changes to the project.

#### Async and Await Easily Enable Asynchronous Interaction

Continuing our cloud and web service ruminations, here is an interesting little hint on asynchronous interactions:

If you are thinking of implementing asynchronous interaction in your web service, which is often a very commendable and necessary thing to do in order to ensure responsiveness and avoid user frustration at a program that just hangs and does nothing, you can save yourself a lot of effort by looking at the async and await keywords provided in the .NET Framework 4.5 Beta.

The asynchronous programming model used in last few years requires a lot of careful attention while coding, which is tremendously simplified by the async features in the latest versions of C# and Visual Basic.

For a very readable overview of this new feature, here is an MSDN article by Brandon Bray,
[Async in 4.5: Worth the Await](http://blogs.msdn.com/b/dotnet/archive/2012/04/03/async-in-4-5-worth-the-await.aspx),
and an
[async and await FAQ](http://blogs.msdn.com/b/pfxteam/archive/2012/04/12/10293335.aspx) by
Stephen Toub.

Unfortunately, as far as I can tell, .NET 4.5 cannot be used with Visual Studio 2010, so I cannot play with this until I move on a bit, which I will try to postpone until I get a new machine, which should happen any time now.
I recently installed
[MVC 4](http://www.microsoft.com/en-us/download/details.aspx?id=30683) and
thus proved first-hand that it works with VS 2010, which in turn proves that that does not require .NET 4.5.