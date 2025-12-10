---
post_number: "1362"
title: "Answer Day Create Roof"
slug: "answer_day_create_roof"
author: "Jeremy Tammik"
tags: ['csharp', 'elements', 'family', 'filtering', 'geometry', 'levels', 'parameters', 'python', 'revit-api', 'rooms', 'selection', 'sheets', 'views', 'walls']
source_file: "1362_answer_day_create_roof.md"
original_url: "https://thebuildingcoder.typepad.com/blog/1362_answer_day_create_roof.html"
---

### Revit Answer Day and Creating a Roof
Today I'll just highlight two items that were also already pointed out
by [Jaime Rosales](http://adndevblog.typepad.com/aec/jaime-rosales.html) on
the [AEC DevBlog](http://adndevblog.typepad.com/aec), plus an addendum for Kinjal and Carolina:
- [Revit Answer Day](#2)
- [Creating a Roof](#3)
- [Setting FootPrintRoof Slope](#4)
#### Revit Answer Day
Do you have any Revit or Revit LT questions that you’ve always wanted to get answered?
If so, please join the next instalment
of [Ask Autodesk Anything](http://www.autode.sk/answerdays) events,
the [Revit Answer Day](http://forums.autodesk.com/t5/revit-answer-day/revit-answer-day-october-7th-2015/td-p/5787599) on
October 7, 2015.
This event is dedicated to answering your questions about Revit and Revit LT.
We’ll have some DevTech engineers attending the event to answer your API questions.
The event runs from 6 am to 6 pm Pacific Time and will take place in Autodesk Community.
You can spend a minute, an hour, or the whole day in the Autodesk Community to interact directly with the folks who can answer every question about Revit you can think of.
#### Creating a Roof
This question was asked last week both on
the [Revit API discussion forum](http://forums.autodesk.com/t5/revit-api/help-creating-roof/m-p/5828806)
and [Stack Overflow](http://stackoverflow.com/questions/32718999/creating-a-roof-function/32732012#32732012):
\*\*Question:\*\*
I'm having trouble programmatically creating a roof.
I know how to create a stairs, for example: I start a `StairsEditScope`, use `CreateSketchedLanding` with the right parameters to create my stairs and commit the `StairsEditScope`.
I can't find a clue on how to create a roof from scratch, though.
Any leads?
\*\*Answer:\*\*
Please always search the Revit API help file RevitAPI.chm and
the [Revit online help](http://help.autodesk.com/view/RVT/2016/ENU) before
asking questions like this.
Here is the sample code provided by the latter, in the section
on [Roofs](http://help.autodesk.com/view/RVT/2016/ENU/?guid=GUID-33E6A6BD-96AA-4FAF-B660-1DD0C06CCD29):

```
  // Before invoking this sample, select some walls
  // to add a roof over. Make sure there is a level
  // named "Roof" in the document.

  Level level
    = new FilteredElementCollector( doc )
      .OfClass( typeof( Level ) )
      .Where<Element>( e =>
        !string.IsNullOrEmpty( e.Name )
        && e.Name.Equals( "Roof" ) )
      .FirstOrDefault<Element>() as Level;

  RoofType roofType
    = new FilteredElementCollector( doc )
      .OfClass( typeof( RoofType ) )
      .FirstOrDefault<Element>() as RoofType;

  // Get the handle of the application
  Application application = doc.Application;

  // Define the footprint for the roof based on user selection
  CurveArray footprint = application.Create
    .NewCurveArray();

  UIDocument uidoc = new UIDocument( doc );

  ICollection<ElementId> selectedIds
    = uidoc.Selection.GetElementIds();

  if( selectedIds.Count != 0 )
  {
    foreach( ElementId id in selectedIds )
    {
      Element element = doc.GetElement( id );
      Wall wall = element as Wall;
      if( wall != null )
      {
        LocationCurve wallCurve = wall.Location as LocationCurve;
        footprint.Append( wallCurve.Curve );
        continue;
      }

      ModelCurve modelCurve = element as ModelCurve;
      if( modelCurve != null )
      {
        footprint.Append( modelCurve.GeometryCurve );
      }
    }
  }
  else
  {
    throw new Exception(
      "Please select a curve loop, wall loop or "
      + "combination of walls and curves to "
      + "create a footprint roof." );
  }

  ModelCurveArray footPrintToModelCurveMapping
    = new ModelCurveArray();

  FootPrintRoof footprintRoof
    = doc.Create.NewFootPrintRoof(
      footprint, level, roofType,
      out footPrintToModelCurveMapping );

  ModelCurveArrayIterator iterator
    = footPrintToModelCurveMapping.ForwardIterator();

  iterator.Reset();
  while( iterator.MoveNext() )
  {
    ModelCurve modelCurve = iterator.Current as ModelCurve;
    footprintRoof.set_DefinesSlope( modelCurve, true );
    footprintRoof.set_SlopeAngle( modelCurve, 0.5 );
  }
```

To test it, please make sure you have some walls to hold up the roof and one of your levels is named `Roof`.
I just created a simple four-wall rectangle, selected them and ran the command.
This code creates a footprint roof.
Revit also provides other kinds.
It is important to understand the Revit product and the various roof types from an end user point of view before thinking about driving them programmatically.
The footprint roof created above is defined by a horizontal outline and is created using the `Document.NewFootPrintRoof` method. Such a roof can be flat, or you can specify a slope for each edge of the outline profile.
The Building Coder [Xtra labs](https://github.com/jeremytammik/AdnRevitApiLabsXtra) provides
another working sample in the external
command [Lab2_0_CreateLittleHouse in Labs2.cs](https://github.com/jeremytammik/AdnRevitApiLabsXtra/blob/master/XtraCs/Labs2.cs).
[The Building Coder](http://thebuildingcoder.typepad.com/) also
provides these other roof-related posts:
- [RoomsRoofs SDK Sample](http://thebuildingcoder.typepad.com/blog/2008/09/roomsroofs-sdk.html)
- [Roof Eave Cut](http://thebuildingcoder.typepad.com/blog/2009/08/roof-eave-cut-in-revit-and-aca.html)
- [Creating an Extrusion Roof](http://thebuildingcoder.typepad.com/blog/2014/09/events-again-and-creating-an-extrusion-roof.html)
![Little house roof](img/little_house_roof.png)
#### Addendum – Setting FootPrintRoof Slope
For Kinjal and Carolina, to answer the Revit discussion forum thread
on [Creating a surface through the Revit API](http://forums.autodesk.com/t5/revit-api/create-a-surface-through-revit-api/m-p/5872832):
Here are all the discussions I found on this, before hitting the [last and hopefully ultimate answer above](#3) that I already forgot I had written:
- [Slope is slope, not radians](http://thebuildingcoder.typepad.com/blog/2010/08/slope-is-slope-not-radians.html)
- [Modifying floor slope programmatically – or not](http://thebuildingcoder.typepad.com/blog/2014/03/creating-a-sloped-floor.html#3)
- [New opening](http://forums.autodesk.com/t5/Revit-API/New-Opening/m-p/5104388)
- [The discussion with Dodd on creating a sloped floor](http://thebuildingcoder.typepad.com/blog/2014/03/creating-a-sloped-floor.html#comment-2231365409)
- Internal ADN cases 09714390 \*New Opening\*, 08886290 \*create footprint roof\* and 07928659 \*Revit 2013如何设置屋顶的坡度\*
I hope this helps once and for all and I do not forget about this answer again.