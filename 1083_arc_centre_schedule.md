---
post_number: "1083"
title: "Happy New Year!"
slug: "arc_centre_schedule"
author: "Jeremy Tammik"
tags: ['csharp', 'elements', 'family', 'filtering', 'references', 'revit-api', 'rooms', 'schedules', 'sheets', 'views']
source_file: "1083_arc_centre_schedule.htm"
original_url: "https://thebuildingcoder.typepad.com/blog/1083_arc_centre_schedule.html"
---

### Happy New Year!

Welcome back after the break and

Happy New Year!

I have already been busily answering cases for the past few days.

Here are two of the topics that came up:

- [Access to model arc centre point reference](#2)
- [API-generated schedules on sheets lack title](#3)

Before getting to those, let me present a photo and an interesting syntactic challenge from my non-technical activities in the past ten days.

I spent some time after Christmas in the hills in Toggenburg, and we had some very fine weather and views:

![Winter scene in Toggenburg](file:////j/photo/jeremy/2013/2013-12-31_wildhaus/256.jpg)

The syntactic challenge is perfectly normal English, and simply consists in understanding the following valid sentence:
Buffalo buffalo Buffalo buffalo buffalo buffalo Buffalo buffalo.

For a few hints, please refer to this interesting
[explanation on Wikipedia](http://en.wikipedia.org/wiki/Buffalo_buffalo_Buffalo_buffalo_buffalo_buffalo_Buffalo_buffalo).

Back to the Revit API...

#### Access to Model Arc Centre Point Reference

**Question:** How can I obtain a valid reference to a model arc centre point?

Given an extrusion element in a family file, you can access its sketch and individual geometric sketch curve elements like this in managed C++:

```csharp
Extrusion ^ext = createExtrusion(...)
CurveArrArray ^sketchCvArrArr = ext->Sketch->Profile;
for each( CurveArray ^oneCvLoop in sketchCvArrArr )
{
for each( Curve ^oneSketchCv in oneCvLoop )
{
// ...
}
}
```

The curve end point references are provided by the Curve::EndPointReference property and can be used to constrain profile curves to reference planes, for example.

However, we could not find a way to obtain a centre point reference for arc sketch curves.

The ModelArc class provides a CenterPointReference property.

However, attempting to cast a geometric curve to a model curve obviously returns a null result.

Our goal is to lock the arc centre to the intersection point of two reference planes.

Can you suggest any way to achieve this?

**Answer:** Looking at the Model Curve and Curve class hierarchy shows that the direct conversion is not possible.

The CenterPointReference property is declared at CurveElement, so ModelCurve will also have it.
You can create a ModelArc with a geometric arc (Curve), but it will appear in the project (be added to the database).

The Curve itself is in-memory only.
From a Curve, you can call Document.Create.NewModelCurve, which will add a new model curve and modify the document.
However, that will still not provide the reference that you are after.

On the other hand, the geometric sketch curve itself also provides a Reference property, and that can be used to access the existing model curve in the sketch.
From that, you can obtain the centre point reference, in case of an arc element.

Here is a snapshot of the debugger showing code that retrieves the sketch model curve in order to get the model arc centre point reference that can be used to create a radial dimension:

![Getting element by reference](img/getting_element_by_ref.png)

This method is perfectly reliable, since Document::GetElement(Reference) always returns the actual model curve element itself.

#### API-generated Schedules on Sheets Lack Title

**Question:** I am working on a project that involves building schedules and placing them on sheets programmatically.
The ones I create with the API don't include a title.

The issue can be illustrated with the Revit SDK ScheduleCreation sample project.

Here is a snapshot of a manually created schedule:

![Schedule created manually](img/schedule_created_manually.jpg)

The programmatically generated one looks like this:

![Schedule created through API](img/schedule_created_through_api.jpg)

**Answer:** Let's start by reiterating the history of this feature:
the Schedule API was initially introduced in
[Revit 2013](http://thebuildingcoder.typepad.com/blog/2012/03/revit-2013-and-its-api.html#2),
enabling an add-in to create schedules and add or remove fields and filters, control sorting and grouping, add, read, place a schedule on a sheet or modify its placement, and access all the schedule data, e.g. to export to a text file.

In
[Revit 2014](http://thebuildingcoder.typepad.com/blog/2013/03/revit-2014-api-and-room-plan-view-boundary-polygon-loops.html) (fourth
bullet item), it was enhanced to provide formatting control and read-write access to individual data items.
More details on the feature are provided there by the Revit 2014 DevDays presentation, recording and sample code.

It is also discussed in the overview of the 'ViewSchedule changes' in
[What's New in the Revit 2014](http://thebuildingcoder.typepad.com/blog/2013/04/whats-new-in-the-revit-2014-api.html).

The Schedule API functionality is illustrated by the Revit 2013 ScheduleCreation that you tested and the Revit 2014 ScheduleAutomaticFormatter and ScheduleToHTML SDK samples.

Now, to address your actual question:

The issue you encountered is known and was resolved in
[Revit 2014 update release 1](http://thebuildingcoder.typepad.com/blog/2013/08/revit-2014-update-release-1.html#2):

- Creates the default schedule header automatically when schedules are created by ViewSchedule.Create methods.
- Allows ViewSchedule.GroupHeaders to succeed even if the schedule is not active.

Have you installed the update release 1, or, better still, 2?

I would recommend installing each update release as soon as it becomes available, if you can.

**Response:** Applying the Update Release 2 patch fixed the title problem.

Thank you, and I will add keeping up on these to my New Year's resolutions!

![Raring to go on towards new challenges in the New Year](file:////j/photo/jeremy/2013/2013-12-31_wildhaus/247_cropped.jpg)