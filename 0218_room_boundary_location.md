---
ai_optimized: true
author: Jeremy Tammik
complexity_score: 1.9
content_type: qa
optimization_date: '2025-12-11T11:44:13.569589'
original_url: https://thebuildingcoder.typepad.com/blog/0218_room_boundary_location.html
post_number: 0218
reading_time_minutes: 2
series: general
slug: room_boundary_location
source_file: 0218_room_boundary_location.htm
tags:
- csharp
- elements
- revit-api
- rooms
- selection
- vbnet
- walls
title: Room Boundary Location
word_count: 339
---

### Room Boundary Location

We already had a look at
[enabling the room volume computation](http://thebuildingcoder.typepad.com/blog/2009/06/volume-computation-enable.html)
in C#.
Here is a closely related question that also prompted me to implement something similar in VB for a change.

**Question:**
Could you please provide some VB.NET code showing how to set the Autodesk.Revit.Rooms.BoundaryLocationType in the document, for instance to 'WallCenter'?

**Answer:**
The boundary location type is applied when calculating the room volume.
These settings are defined by the document settings volume calculation options.
The post mentioned above discusses how to access and modify these settings.

I implemented a VB.NET sample application RoomBoundaryLocation for you to demonstrate this in VB as well.
Here is the mainline code of its Execute method:
```vbnet
Public Function Execute( \_
  ByVal commandData As ExternalCommandData, \_
  ByRef message As String, \_
  ByVal elements As ElementSet) \_
  As CmdResult \_
  Implements IExternalCommand.Execute
  Dim app As Application = commandData.Application
  Dim doc As Document = app.ActiveDocument
  Dim opt As VolumeCalculationOptions \_
    = doc.Settings.VolumeCalculationSetting \_
      .VolumeCalculationOptions
  opt.VolumeComputationEnable = True
  opt.RoomAreaBoundaryLocation \_
    = Rooms.BoundaryLocationType.WallCenter
  doc.Settings.VolumeCalculationSetting \_
    .VolumeCalculationOptions = opt
  Dim volumes As List(Of String) = New List(Of String)
  Dim els As ElementSet = doc.Selection.Elements
  Dim e As Autodesk.Revit.Element
  For Each e In els
    If TypeOf e Is Room Then
      Dim room As Room = CType(e, Room)
      volumes.Add(room.Volume.ToString("0.##"))
      'Else
      '  volumes.Add("Not a room")
    End If
  Next
  If 0 = volumes.Count Then
    message = "Please select some rooms."
  Else
    Dim s As String = \_
      +"Selected room volumes in cubic feet: " \_
      + String.Join(", ", volumes.ToArray()) \_
      + "."
    MsgBox(s)
  End If
  Return CmdResult.Failed
End Function
```

Here are some sample rooms and spaces selected in the Revit user interface:
![Selected rooms and spaces](img/selected_rooms.png)

This is the resulting dialogue displayed by running the command:
![Selected room volumes](img/selected_room_volumes.png)

Here is the complete Visual Studio solution
[RoomBoundaryLocation](zip/RoomBoundaryLocation.zip)
implementing the new command.