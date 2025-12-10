---
post_number: "0899"
title: "Retrieving Schedules on a Sheet"
slug: "schedules_on_sheet"
author: "Jeremy Tammik"
tags: ['csharp', 'elements', 'filtering', 'revit-api', 'schedules', 'sheets', 'views']
source_file: "0899_schedules_on_sheet.htm"
original_url: "https://thebuildingcoder.typepad.com/blog/0899_schedules_on_sheet.html"
---

### Retrieving Schedules on a Sheet

[Guy Robinson](http://redbolts.com) recently
explained an easy way to
[access all elements in a schedule](http://thebuildingcoder.typepad.com/blog/2012/12/accessing-all-element-in-a-schedule.html).

[Victor Chekalin](http://www.facebook.com/profile.php?id=100003616852588) picked up and expanded on that.

Here is his solution to retrieve all schedules in a view and elements from a schedule.
In his own words:

One wonderful day I had to retrieve schedules which are presented on a
sheet:

![Schedules on a sheet](img/vc_schedules_on_sheet_1.png)

At first I looked at the Revit API Help but unfortunately there is no method such as GetSchdulesOnView in the ViewSheet class.
There is no explicit way to get it.
After a couple of hours trying to achieve that I need I encountered Guy's simple solution to
[access all elements in a schedule](http://thebuildingcoder.typepad.com/blog/2012/12/accessing-all-element-in-a-schedule.html).
The solution is use FilteredElementCollector and pass in the element id of the view in which you want to collect elements.

I created the following simple extension method GetSchedules to get schedules on view:
```csharp
  public static class ViewSheetExtensions
  {
    public static IEnumerable<ViewSchedule>
      GetSchedules( this ViewSheet viewSheet )
    {
      var doc = viewSheet.Document;

      FilteredElementCollector collector =
        new FilteredElementCollector(
          doc, viewSheet.Id );

      var scheduleSheetInstances =
        collector
          .OfClass( typeof( ScheduleSheetInstance ) )
          .ToElements()
          .OfType<ScheduleSheetInstance>();

      foreach( var scheduleSheetInstance in
        scheduleSheetInstances )
      {
        var scheduleId =
          scheduleSheetInstance
            .ScheduleId;

        if( scheduleId == ElementId.InvalidElementId )
          continue;

        var viewSchedule =
          doc.GetElement( scheduleId )
          as ViewSchedule;

        if( viewSchedule != null )
          yield return viewSchedule;
      }
    }
  }
```

It can be used like this:
```csharp
  var schedules = viewSheet
    .GetSchedules()
    .ToList();
  foreach( var viewSchedule in schedules )
  {
    // Do something
  }
```

Here is the demo result:

![Demo result](img/vc_schedules_on_sheet_2.png)

But I decided not to stop and go deeper.
The next step is to get all elements which are involved in a schedule.
The approach is the same as with schedules. Using FiltereredElementCollector but instead of the ViewSheet element id we pass in the ViewSchedule id.

The extension method is similar:
```csharp
  public static class ViewScheduleExtensions
  {
    public static IEnumerable<ElementId>
      GetElementIdsInSchedule(
        this ViewSchedule viewSchedule )
    {
      var doc = viewSchedule.Document;

      FilteredElementCollector collector =
        new FilteredElementCollector(
          doc, viewSchedule.Id );

      var elementIds = collector
        .WhereElementIsNotElementType()
        .ToElementIds();

      return elementIds;
    }
  }
```

But there is an additional little nuance.
If I retrieve elements from a material takeoff schedule, the method will return all materials in the project together with other elements. The solution is to just skip all elements which are materials:

```csharp
  foreach( var id in elementIds )
  {
    var element = doc.GetElement( id );

    if( element is Material )
      continue;

    // Do something
  }
```

Here is the result of my demo project:

![Demo result](img/vc_schedules_on_sheet_3.png)

I attached the
[demo Visual Studio solution and Revit project](zip/VcScheduleAPIDemo.zip).

I hope this information will useful for developers.

Have a nice day,

Victor.