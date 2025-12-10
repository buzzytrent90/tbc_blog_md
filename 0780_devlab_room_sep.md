---
post_number: "0780"
title: "DevLab and Room Separation"
slug: "devlab_room_sep"
author: "Jeremy Tammik"
tags: ['csharp', 'levels', 'revit-api', 'rooms', 'views']
source_file: "0780_devlab_room_sep.htm"
original_url: "https://thebuildingcoder.typepad.com/blog/0780_devlab_room_sep.html"
---

### DevLab and Room Separation

Today is DevLab with almost fifty participants, so we are in several separate rooms.
I never left the Revit room, and am busy enough with that.
Talk about room separation.

Here are one or two little items that cropped up today, contributed by Eric Boehlke of
[truevis.com](http://www.truevis.com).
The first is actually a question on the creation of room separation lines that I already replied to in response to a
[comment](http://thebuildingcoder.typepad.com/blog/2012/03/on-creating-a-mass-from-3d-points.html?cid=6a00e553e168978833016303fac96e970d#comment-6a00e553e168978833016303fac96e970d) from
Ammar on
[creating a mass](http://thebuildingcoder.typepad.com/blog/2012/03/on-creating-a-mass-from-3d-points.html):

#### Creating a Room Separation Line

**Question:** It seems that we have no access to create room separation lines?

**Answer:** Please check out the NewRoomBoundaryLines, NewSpaceBoundaryLines, NewSpaces and other related methods.

Eric ran into the same issue, and the answer I provided to Ammar does not show up in his Google search attempt, so here it is now, promoted to a main blog post status.

Here is Eric's code snippet to generate a 10 by 8 room by first creating a rectangular sequence of four room separation lines using the NewRoomBoundaryLines method on a given level:
```csharp
/// <summary>
/// Create a room on a given level.
/// </summary>
void CreateRoom(
  Document doc,
  Level level )
{
  Application app = doc.Application;

  Autodesk.Revit.Creation.Application
    appCreation = app.Create;

  Autodesk.Revit.Creation.Document
    docCreation = doc.Create;

  XYZ pt1 = new XYZ( 0, -5, 0 );
  XYZ pt2 = new XYZ( 0, 5, 0 );
  XYZ pt3 = new XYZ( 8, 5, 0 );
  XYZ pt4 = new XYZ( 8, -5, 0 );

  Line line1 = appCreation.NewLine( pt1, pt2, true );
  Line line2 = appCreation.NewLine( pt2, pt3, true );
  Line line3 = appCreation.NewLine( pt3, pt4, true );
  Line line4 = appCreation.NewLine( pt4, pt1, true );

  CurveArray curveArr = new CurveArray();

  curveArr.Append( line1 );
  curveArr.Append( line2 );
  curveArr.Append( line3 );
  curveArr.Append( line4 );

  docCreation.NewRoomBoundaryLines(
    doc.ActiveView.SketchPlane,
    curveArr, doc.ActiveView );

  // Create a new room

  UV tagPoint = new UV( 4, 0 );

  Room room = docCreation.NewRoom(
    level, tagPoint );

  if( null == room )
  {
    throw new Exception(
      "Create a new room failed." );
  }
  room.Number = "42";
  room.Name = "Lobby";

  RoomTag tag = docCreation.NewRoomTag(
    room, tagPoint, doc.ActiveView );
}
```

#### CopyPathEx and Other Nifty Utilities

Eric also pointed to a list of
[great freeware for Revit work](http://revitlearningclub.blogspot.com/2011/12/great-freeware-for-revit-work.html),
from which I immediately installed
**[PathCopyEx](http://www.mlin.net/other.shtml)**
[PathCopyEx.msi](http://www.mlin.net/files/PathCopyEx.msi).
Many thanks to Eric for his code snippet and helpful pointers!

Now I have to start thinking about getting down to the airport in time for my flight back home to Europe...