---
post_number: "0682"
title: "Accessing Room Data"
slug: "accessing_room_data"
author: "Jeremy Tammik"
tags: ['csharp', 'elements', 'filtering', 'geometry', 'revit-api', 'rooms', 'views', 'walls', 'windows']
source_file: "0682_accessing_room_data.htm"
original_url: "https://thebuildingcoder.typepad.com/blog/0682_accessing_room_data.html"
---

### Accessing Room Data

Happy [Thanksgiving](http://en.wikipedia.org/wiki/Thanksgiving) to all you Americans!

For me, Thursday November 24 is hopefully a happy travelling day, arriving in Las Vegas a few days in advance to acclimatize.

Here is a nice little beginner's question that I decided to answer by implementing a new command in The Building Coder samples:

**Question:** How can I access the room information in a Revit BIM?
I would like to see a list of all rooms with their position, properties, and boundary polygon.

**Answer:** A list of rooms can be easily obtained using a filtered element collector.

Unfortunately, there is one small complication when trying to directly use the Room class for filtering. If you try that, the Revit API throws an exception:

"Input type is of an element type that exists in the API, but not in Revit's native object model. Try using Autodesk.Revit.DB.SpatialElement instead, and then postprocessing the results to find the elements of interest."

This situation and how to handle it is described in several blog posts, e.g. on
[filtering for a non-native class](http://thebuildingcoder.typepad.com/blog/2010/08/filtering-for-a-nonnative-class.html) and the
[MeasurePanelArea update](http://thebuildingcoder.typepad.com/blog/2010/10/measurepanelarea-update.html).

I wrote a new Building Coder sample command CmdListAllRooms for you which shows how to retrieve all rooms and list their basic information.

Here is the report it produces in the Visual Studio debug output window when run on the rac\_basic\_sample\_project.rvt provided in the Revit Architecture installation (copy and paste to a text editor to see the truncated lines in full):

```
Room nr. '1' named 'BEDROOM 1 1' at (0,0,0) with bounding box <null> and area 0 sqf has 0 loops and 0 segments in first loop.
Room nr. '2' named 'BEDROOM 2 2' at (0,0,0) with bounding box <null> and area 0 sqf has 0 loops and 0 segments in first loop.
Room nr. '4' named 'BATH 4' at (0,0,0) with bounding box <null> and area 0 sqf has 0 loops and 0 segments in first loop.
Room nr. '5' named 'BEDROOM 1 5' at (-37.7,13.68,0) with bounding box ((-44.09,6.56,0),(-32.11,20.04,8)) and area 160.523659936209 sqf has 1 loop and 8 segments in first loop.
Room nr. '6' named 'BEDROOM 2 6' at (-25.19,13.68,0) with bounding box ((-31.72,6.56,0),(-19.61,20.04,8)) and area 162.29204125075 sqf has 1 loop and 9 segments in first loop.
Room nr. '7' named 'HALL 7' at (-30.33,21.65,0) with bounding box ((-35.47,20.43,0),(-19.42,23.7,8)) and area 52.541410916154 sqf has 1 loop and 6 segments in first loop.
Room nr. '8' named 'BATH 8' at (-38.52,23.22,0) with bounding box ((-44.09,20.43,0),(-35.86,26.87,8)) and area 52.9879883370874 sqf has 1 loop and 5 segments in first loop.
Room nr. '9' named 'STORAGE 9' at (-9.09,9.09,0) with bounding box ((-19.22,6.56,0),(0.75,15.04,8)) and area 104.9958666584 sqf has 1 loop and 8 segments in first loop.
Room nr. '10' named 'LIVING 10' at (0,0,0) with bounding box <null> and area 0 sqf has 0 loops and 0 segments in first loop.
Room nr. '11' named 'MECH 11' at (-25.47,25.29,0) with bounding box ((-35.47,24.1,0),(-19.61,26.87,8)) and area 44.0105942711896 sqf has 1 loop and 4 segments in first loop.
Room nr. '12' named 'LIVING ROOM 12' at (-12.69,20.48,0) with bounding box <null> and area 0 sqf has 0 loops and 0 segments in first loop.
```

It uses the helper method ListRoomData to list the room information for a given element:
```csharp
/// <summary>
/// List some properties of a given room to the
/// Visual Studio debug output window.
/// </summary>
void ListRoomData( Room room )
{
  SpatialElementBoundaryOptions opt
    = new SpatialElementBoundaryOptions();

  string nr = room.Number;
  string name = room.Name;
  double area = room.Area;

  Location loc = room.Location;
  LocationPoint lp = loc as LocationPoint;
  XYZ p = ( null == lp ) ? XYZ.Zero : lp.Point;

  BoundingBoxXYZ bb = room.get\_BoundingBox( null );

  IList<IList<BoundarySegment>> boundary
    = room.GetBoundarySegments( opt );

  int nLoops = boundary.Count;

  int nFirstLoopSegments = 0 < nLoops
    ? boundary[0].Count
    : 0;

  Debug.Print( string.Format(
    "Room nr. '{0}' named '{1}' at {2} with "
    + "bounding box {3} and area {4} sqf has "
    + "{5} loop{6} and {7} segment{8} in first "
    + "loop.",
    nr, name, Util.PointString( p ),
    BoundingBoxString2( bb ), area, nLoops,
    Util.PluralSuffix( nLoops ), nFirstLoopSegments,
    Util.PluralSuffix( nFirstLoopSegments ) ) );
}
```

This method is called from the mainline Execute method which retrieves the rooms from the model:
```csharp
public Result Execute(
  ExternalCommandData commandData,
  ref string message,
  ElementSet elements )
{
  UIApplication uiapp = commandData.Application;
  UIDocument uidoc = uiapp.ActiveUIDocument;
  Application app = uiapp.Application;
  Document doc = uidoc.Document;

  // Filtering for Room elements throws an exception:
  // Input type is of an element type that exists in
  // the API, but not in Revit's native object model.
  // Try using Autodesk.Revit.DB.SpatialElement
  // instead, and then postprocessing the results to
  // find the elements of interest.

  //FilteredElementCollector a
  //  = new FilteredElementCollector( doc )
  //    .OfClass( typeof( Room ) );

  FilteredElementCollector a
    = new FilteredElementCollector( doc )
      .OfClass( typeof( SpatialElement ) );

  foreach( SpatialElement e in a )
  {
    Room room = e as Room;

    if( null != room )
    {
      ListRoomData( room );
    }
  }
  return Result.Succeeded;
}
```

Here is
[version 2012.0.94.0](zip/bc_12_94.zip) of
The Building Coder samples including the new CmdListAllRooms command.

A couple of other samples should be useful to you in this context as well:

- Rooms SDK sample- RoomViewer SDK sample- The Building Coder sample command CmdRoomWallAdjacency

The Revit SDK sample Rooms demonstrates how to retrieve room information such as Number, Area, Department, etc., add room tags and change room numbers.

The RoomViewer SDK sample demonstrates the access and usage of the room boundary and displays the room boundary geometry in a dedicated viewer.

The Building Coder sample command CmdRoomWallAdjacency shows a more advanced usage of the room boundary information to
[determine room and wall adjacency](http://thebuildingcoder.typepad.com/blog/2009/01/room-and-wall-adjacency.html).
The access to the boundary information has changed since that sample was published, though, so you need to look at a more recent version of The Building Coder samples in order to run it in Revit 2012, for instance the version 2012.0.94.0 provided above.