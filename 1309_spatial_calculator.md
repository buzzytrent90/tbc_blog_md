---
ai_optimized: true
author: Jeremy Tammik
complexity_score: 3.2
content_type: qa
optimization_date: '2025-12-11T11:44:15.740519'
original_url: https://thebuildingcoder.typepad.com/blog/1309_spatial_calculator.html
post_number: '1309'
reading_time_minutes: 7
series: general
slug: spatial_calculator
source_file: 1309_spatial_calculator.htm
tags:
- csharp
- doors
- elements
- family
- filtering
- geometry
- parameters
- revit-api
- rooms
- schedules
- transactions
- vbnet
- views
- walls
title: Gross and Net Wall Area Calculation Enhancement
word_count: 1372
---

### Gross and Net Wall Area Calculation Enhancement

We make further enhancements to the on-going project to determine gross and net areas and volumes, obviously a fundamental issue to BIM.

And I have more news to share as well:

- [Playing with my WebGL viewer](#1)
- [WebGL Developers meetup in NYC April 21](#2)
- [AEC Hackathon in Dallas May 1-3](#3)
- [Angelhack in Dubai May 7-9](#4)
- [Angelhack in Athens June 5-7](#5)
- [Wall area calculation handling multiple openings in multiple walls in multiple rooms](#6)

#### Playing with my WebGL Viewer

My last post was on
[exporting 3D element geometry to a WebGL viewer](http://thebuildingcoder.typepad.com/blog/2015/04/exporting-3d-element-geometry-to-a-webgl-viewer.html).

If this is of interest to you, you absolutely must check out two subsequent
[The 3D Web Coder](http://the3dwebcoder.typepad.com) posts on the
[desktop driven WebGL viewer](http://the3dwebcoder.typepad.com/blog/2015/04/webgl-viewer-cloud-accelerator-verold-and-rtc.html#5) and
[converting it to a node.js server](http://the3dwebcoder.typepad.com/blog/2015/04/webgl-node-server-with-github-and-heroku-repo-sync.html).

#### WebGL Developers Meetup in NYC April 21

Get ready for our [WebGL Developers Meetup in NYC](http://www.meetup.com/NYC-WebGL-Developers/events/221871315) on Tuesday April 21.

Come and join Jim Quanci [@jimquanci](https://twitter.com/jimquanci) from Autodesk to learn how to easily add 3D viewing functionality to your web and mobile apps using [View and Data API](https://developer.autodesk.com).

#### AEC Hackathon in Dallas May 1-3

As an AEC developer, you will certainly not want to miss the [AEC Hackathon 2.2 in Dallas](http://aechackathon.com/aec-hackathon-dallas).

We will be on site to show the [View and Data API](https://developer.autodesk.com/api/view-and-data-api) and of course help all hackers on our AEC products.

#### Angelhack in Dubai May 7-9

Of special interest to me personally, of course:

I will be going to the [Angelhack in Dubai](http://angelhack.com/hackathon/dubai-2015) on May 7-9.

Since the weekend is on Friday and Saturday there, we have scheduled an
[Angel workshop](http://www.meetup.com/I-love-3D-Dubai/events/221497781) meetup session on Thursday evening, where, once again, I will be presenting on the
[View and Data API](https://developer.autodesk.com/api/view-and-data-api) and anything else you want to learn.

Then you will be fully primed and up to speed for the subsequent hackathon!

I am looking forward to meeting you there!

#### Angelhack in Athens June 5-7

Same thing again... of special interest to me personally:

I will be going to the [Angelhack in Athens](http://angelhack.com/hackathon/athens-2015) on June 5-7.

Again, we have scheduled an
[Angel workshop](http://www.meetup.com/I-love-3D-Athens/events/221674308) meetup
session on Friday evening to get us all well prepared and up to speed for the subsequent hackathon.

I am looking forward to meeting you there!

#### Wall Area Calculation Handling Multiple Openings in Multiple Walls in Multiple Rooms

I recently presented a promising solution to address this in the discussion on
[calculating gross and net wall areas](http://thebuildingcoder.typepad.com/blog/2015/03/calculating-gross-and-net-wall-areas.html),
publishing the initial VB.NET solution by Phillip Miller of Kiwi Codes Solutions Ltd and
an enhanced C# implementation in the
[SpatialElementGeometryCalculator GitHub repository](https://github.com/jeremytammik/SpatialElementGeometryCalculator) incorporating
the
[suggestion](http://thebuildingcoder.typepad.com/blog/2015/03/calculating-gross-and-net-wall-areas.html#comment-6a00e553e16897883301b8d0eee480970c) by Vilo to
[use FindInserts to retrieve all openings in all wall types](http://thebuildingcoder.typepad.com/blog/2015/03/findinserts-retrieves-all-openings-in-all-wall-types.html).

[Håkon Clausen](http://hclausen.net) responded to that in this
[comment](http://thebuildingcoder.typepad.com/blog/2015/03/findinserts-retrieves-all-openings-in-all-wall-types.html?cid=6a00e553e16897883301b7c7789a82970b#comment-6a00e553e16897883301b7c7789a82970b):

I found two issues with this solution after playing around with it for some hours.

1. In contrast to the "Calculating Gross and Net Wall Areas" solution, after getting the wall inserts with FindInserts it does not check that the inserts actually belongs to the room and thus removes inserts to other rooms if the wall is part of multiple rooms. This will of course lead to wrong result. Checking the instance room/toroom/fromroom parameter like the previous solution rectifies this.
2. If a wall has multiple faces or subfaces, the opening area is subtracted multiple times. Take e.g. the hall from the 2013 basic sample (or any room with a wall with multiple doors to other rooms. This will make calwallAreaMinusOpenings report a negative wall area. Creating a gross sum for each wall, then looping through the walls found and calculate the opening area once seems to be an approach.

I added that to the SpatialElementGeometryCalculator GitHub repository as
[issue #1, improvements for multiple rooms and multiple subfaces](https://github.com/jeremytammik/SpatialElementGeometryCalculator/issues/1), and
suggested to Håkon to fork the repo, add his changes to that, and create a pull request for me to merge them back in.

He did so and created
[pull request #2, rework based on comment on blog](https://github.com/jeremytammik/SpatialElementGeometryCalculator/pull/2).

For your convenience, here is the updated implementation:

```csharp
public Result Execute(
  ExternalCommandData commandData,
  ref string message,
  ElementSet elements )
{
  var app = commandData.Application;
  var doc = app.ActiveUIDocument.Document;
  Result rc;
  string s = string.Empty;
  try
  {
    var roomCol = new FilteredElementCollector( doc )
      .OfClass( typeof( SpatialElement ) );

    foreach( var e in roomCol )
    {
      var room = e as Room;
      if( room == null ) continue;
      if( room.Location == null ) continue;

      var sebOptions = new SpatialElementBoundaryOptions
      { SpatialElementBoundaryLocation
        = SpatialElementBoundaryLocation.Finish
      };
      var calc = new Autodesk.Revit.DB
        .SpatialElementGeometryCalculator(
          doc, sebOptions );

      var results = calc
        .CalculateSpatialElementGeometry( room );

      // To keep track of each wall and
      // its total area in the room

      var walls = new Dictionary<string, double>();

      foreach( Face face in results.GetGeometry().Faces )
      {
        foreach( var subface in results
          .GetBoundaryFaceInfo( face ) )
        {
          if( subface.SubfaceType
            != SubfaceType.Side ) { continue; }

          var wall = doc.GetElement( subface
            .SpatialBoundaryElement.HostElementId )
              as HostObject;

          if( wall == null ) { continue; }
          var grossArea = subface.GetSubface().Area;
          if( !walls.ContainsKey( wall.UniqueId ) )
          {
            walls.Add( wall.UniqueId, grossArea );
          }
          else
          {
            walls[wall.UniqueId] += grossArea;
          }
        }
      }

      foreach( var id in walls.Keys )
      {
        var wall = (HostObject) doc.GetElement( id );
        var openings = CalculateWallOpeningArea(
          wall, room );

        s += string.Format(
          "Room: {2} Wall: {0} Area: {1} m2\r\n",
          wall.get\_Parameter( BuiltInParameter.ALL\_MODEL\_MARK ).AsString(),
          SqFootToSquareM( walls[id] - openings ),
          room.get\_Parameter( BuiltInParameter.ROOM\_NUMBER ).AsString() );
      }
    }
    TaskDialog.Show( "Room Boundaries", s );
    rc = Result.Succeeded;
  }
  catch( Exception ex )
  {
    TaskDialog.Show( "Room Boundaries",
      ex.Message + "\r\n" + ex.StackTrace );
    rc = Result.Failed;
  }

  return rc;
}

/// <summary>
/// Convert square feet to square meters
/// with two decimal places precision.
/// </summary>
private static double SqFootToSquareM(
  double sqFoot )
{
  return Math.Round( sqFoot \* 0.092903, 2 );
}

/// <summary>
/// Calculate wall area minus openings. Temporarily
/// delete all openings in a transaction that is
/// rolled back.
/// </summary>
private static double CalculateWallOpeningArea(
  HostObject wall,
  Room room )
{
  var doc = wall.Document;
  var wallAreaNet = wall.get\_Parameter(
    BuiltInParameter.HOST\_AREA\_COMPUTED ).AsDouble();

  var t = new Transaction( doc );
  t.Start( "Temp" );
  foreach( var id in wall.FindInserts(
    true, true, true, true ) )
  {
    var insert = doc.GetElement( id );
    if( insert is FamilyInstance
      && IsInRoom( room, (FamilyInstance) insert ) )
    {
      doc.Delete( id );
    }
  }

  doc.Regenerate();
  var wallAreaGross = wall.get\_Parameter(
    BuiltInParameter.HOST\_AREA\_COMPUTED ).AsDouble();
  t.RollBack();

  return wallAreaGross - wallAreaNet;
}

/// <summary>
/// Predicate to determine whether the given
/// family instance belongs to the given room.
/// </summary>
static bool IsInRoom(
  Room room,
  FamilyInstance f )
{
  ElementId rid = room.Id;
  return ( ( f.Room != null && f.Room.Id == rid )
    || ( f.ToRoom != null && f.ToRoom.Id == rid )
    || ( f.FromRoom != null && f.FromRoom.Id == rid ) );
}
```

Many thanks to Håkon for suggesting, implementing and sharing this with us all!

The complete source code, Visual Solution files and add-in manifests for both the original, now outdated, VB.NET and the enhanced C# implementations are hosted by the
[SpatialElementGeometryCalculator GitHub repository](https://github.com/jeremytammik/SpatialElementGeometryCalculator).

The improved version by Håkon presented here is
[release 2015.0.0.3](https://github.com/jeremytammik/SpatialElementGeometryCalculator/releases/tag/2015.0.0.3).

I look forward to hearing how you can make use of this.

Further enhancements are welcome!

Enjoy.