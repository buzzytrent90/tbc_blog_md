---
ai_optimized: true
author: Jeremy Tammik
complexity_score: 3.7
content_type: code_example
optimization_date: '2025-12-11T11:44:13.403261'
original_url: https://thebuildingcoder.typepad.com/blog/0125_new_area.html
post_number: '0125'
reading_time_minutes: 2
series: general
slug: new_area
source_file: 0125_new_area.htm
tags:
- csharp
- elements
- levels
- revit-api
- rooms
- views
title: Create an Area
word_count: 429
---

### Create an Area

Sebastian recently submitted a
[question](http://thebuildingcoder.typepad.com/blog/2009/03/more-questions.html?cid=6a00e553e16897883301156f20ba0d970c#comment-6a00e553e16897883301156f20ba0d970c)
on creating a new area element:

**Question:**
I would like to create an area element overlaid on a room in an area plan view.

**Answer:**
The Revit API provides one method on the Document class in the Autodesk.Revit.Creation namespace for creating new areas,

```
Area NewArea( ViewPlan areaView, UV point );
```

It is similar to one of the overloads of the NewRoom method mentioned in the discussion on
[how to create a room](http://thebuildingcoder.typepad.com/blog/2009/03/create-room-on-level-in-phase.html):

```
Room NewRoom( Level level, UV point );
```

An area can only be created in an area plan view.
The determination of the area boundaries is automatic, given a point in its interior.

Therefore, if a room already exists, you just need to determine a point in its interior and feed that and the area plan view to the NewArea method.
One easy way to obtain a point in the interior of a room is to use its location point.
That returns an XYZ instance.
I checked that the returned Z coordinate was zero and simply discarded that to create the UV point required by the NewArea method.
Of course, there may be more complex situations where some kind of transformation is required, but I am not sure.

Here is the implementation of the Execute method of a new external command class CmdNewArea which demonstrates this.
First, it checks that the current view is an area plan view.
It then checks whether a single room has been selected, or prompts you to do so interactively.
Once these preliminaries have been completed, the following code creates a new area element based on the selected room boundaries:

```csharp
CmdResult rc = CmdResult.Failed;

ViewPlan view = commandData.View as ViewPlan;

if( null == view
  || view.ViewType != ViewType.AreaPlan )
{
  message = "Please run this command in an area plan view.";
  return rc;
}

Application app = commandData.Application;
Document doc = app.ActiveDocument;

Element room = Util.GetSingleSelectedElement( doc );

if( null == room || !(room is Room) )
{
  room = Util.SelectSingleElement( doc, "a room" );
}

if( null == room || !( room is Room ) )
{
  message = "Please select a single room element.";
}
else
{
  Location loc = room.Location;
  LocationPoint lp = loc as LocationPoint;
  XYZ p = lp.Point;
  UV q = new UV( p.X, p.Y );
  Area area = doc.Create.NewArea( view, q );
  rc = CmdResult.Succeeded;
}
return rc;
```

Here is
[version 1.0.0.30](http://thebuildingcoder.typepad.com/blog/files/bc10030.zip)
of the complete Visual Studio solution with the new command.