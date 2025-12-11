---
ai_optimized: true
author: Jeremy Tammik
complexity_score: 5.5
content_type: code_example
optimization_date: '2025-12-11T11:44:13.464024'
original_url: https://thebuildingcoder.typepad.com/blog/0161_volumecomputationenable.html
post_number: '0161'
reading_time_minutes: 2
series: general
slug: volumecomputationenable
source_file: 0161_volumecomputationenable.htm
tags:
- csharp
- elements
- python
- revit-api
- rooms
- selection
- windows
title: Volume Computation Enable
word_count: 453
---

### Volume Computation Enable

We discussed abstract
[volume computation](http://thebuildingcoder.typepad.com/blog/2009/06/convex-hull-and-volume-computation.html)
of a convex hull or cloud of points last week.
Revit can also compute certain volumes, of course, for instance the volume of rooms.
However, this functionality is not automatically enabled.
To switch it on you need to set a certain toggle called VolumeComputationEnable on the document settings volume calculation options.
Unfortunately, there is a slight issue with the API documentation on this property ...

**Question:**
When I try turn on the value of the VolumeComputationEnable property so I can export the volumes of rooms, the value doesn't change.

This is the code that I use for modify this property:

```
Dim revitapp As Autodesk.Revit.Application _
  = revitCommandData.Application

Dim activedoc As Autodesk.Revit.Document _
  = revitapp.ActiveDocument

activedoc.Settings.VolumeCalculationSetting _
  .VolumeCalculationOptions.VolumeComputationEnable _
  = True
```

**Answer:**
The code sample on the VolumeCalculationOptions.VolumeComputationEnable property in the Revit API help file does not work. It says that one can set the property using
```csharp
doc.Settings.VolumeCalculationSetting
.VolumeCalculationOptions
.VolumeComputationEnable = true;
```

This code snippet will not work, because VolumeCalculationOptions is a value class and just returns information about the current status. Therefore you need to write it back to the VolumeCalculationSetting property to turn on the computation.
One has to use something like this instead:
```csharp
  VolumeCalculationOptions options
    = doc.Settings.VolumeCalculationSetting
      .VolumeCalculationOptions;

  options.VolumeComputationEnable = true;

  doc.Settings.VolumeCalculationSetting
    .VolumeCalculationOptions = options;
```

I copied the sample code from the API document and removed one line to create this little reporting method which does not modify anything:
```csharp
public void GetRoomDimensions(
  Document doc,
  Room room )
{
  string roominfo = "\nRoom dimensions:";
  roominfo += "\nVolume: " + room.Volume;
  roominfo += "\nArea: " + room.Area;
  roominfo += "\nPerimeter: " + room.Perimeter;
  roominfo += "\nUnbounded height: "
    + room.UnboundedHeight;

  Debug.Print( roominfo );
}
```

Then I implemented the following external command to test it and verify that it works fine now:
```python
public IExternalCommand.Result Execute(
  ExternalCommandData commandData,
  ref string message,
  ElementSet elements )
{
  Application app = commandData.Application;
  Document doc = app.ActiveDocument;
  ElementSet a = doc.Selection.Elements;
  Room room = null;
  foreach ( Element e in a )
  {
    if ( e is Room )
    {
      room = e as Room;
      break;
    }
  }
  if( null == room )
  {
    message = "Please select a room.";
  }
  else
  {
    Debug.Print( "VolumeComputationEnable = {0}",
      (doc.Settings.VolumeCalculationSetting
        .VolumeCalculationOptions
        .VolumeComputationEnable
      ? "true" : "false") );

    GetRoomDimensions( doc, room );
    // turn on volume calculations:
    VolumeCalculationOptions options
      = doc.Settings.VolumeCalculationSetting
        .VolumeCalculationOptions;

    options.VolumeComputationEnable = true;

    doc.Settings.VolumeCalculationSetting
      .VolumeCalculationOptions = options;

    GetRoomDimensions( doc, room );
  }
  return IExternalCommand.Result.Failed;
}
```

Here is the log displayed in the debug output window after selecting a sample room and running this command:

```
VolumeComputationEnable = false

Room dimensions:
Volume: 0
Area: 73.6251472502946
Perimeter: 36.745406824147
Unbounded height: 13.1233595800525

Room dimensions:
Volume: 966.209281499929
Area: 73.6251472502946
Perimeter: 36.745406824147
Unbounded height: 13.1233595800525
```