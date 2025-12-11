---
ai_optimized: true
author: Jeremy Tammik
complexity_score: 2.0
content_type: qa
optimization_date: '2025-12-11T11:44:14.251172'
original_url: https://thebuildingcoder.typepad.com/blog/0607_cable_tray_shape.html
post_number: '0607'
reading_time_minutes: 2
series: general
slug: cable_tray_shape
source_file: 0607_cable_tray_shape.htm
tags:
- csharp
- elements
- family
- filtering
- parameters
- revit-api
- transactions
title: Modifying Cable Tray Shape
word_count: 310
---

### Modifying Cable Tray Shape

**Question:** I am able to successfully duplicate cable tray family types via the Revit API.

I would like to use that feature to duplicate an existing cable tray family type and then change the shape of a given cable tray to the new type.

So far, I have not been able to find any API methods to achieve this.
Is it possible?
If so, how, please?

**Answer:** There is nothing special at all about changing the cable tray type from channel to ladder.
In fact, the approach to use is completely generic and can be used to change the type of almost any Revit element from its current element type to any other compatible type.
I already provided a pretty generic solution to
[change the element type](http://thebuildingcoder.typepad.com/blog/2010/07/change-element-type.html).
Please note that you can often achieve a lot of powerful functionality through the Revit API just making use of very generic and basic element and parameter access.

Still, here is a dedicated code snippet which selects the cable trays in a given model and changes their type from Channel to Ladder.

You can run this code in any model with a channel shaped cable tray and a cable tray type named "Ladder Cable Tray" loaded.
As said, it changes all cable trays to ladder shaped:
```csharp
  Document doc = commandData.Application
    .ActiveUIDocument.Document;

  // Get the trays

  FilteredElementCollector trays
    = new FilteredElementCollector( doc )
      .OfCategory( BuiltInCategory.OST\_CableTray )
      .WhereElementIsNotElementType();

  // Get the ladder tray type

  FilteredElementCollector trayTypes
    = new FilteredElementCollector( doc )
      .OfClass( typeof( CableTrayType ) );

  Element ladderType = trayTypes.First<Element>(
    e => e.Name.Equals( "Ladder Cable Tray" ) );

  // Set all trays type to ladder

  foreach( Element tray in trays )
  {
    Transaction trans = new Transaction( doc, "Edit Type" );
    trans.Start();
    tray.ChangeTypeId( ladderType.Id );
    trans.Commit();
  }
  return Result.Succeeded;
```

Many thanks to Saikat Bhattacharya for this solution.