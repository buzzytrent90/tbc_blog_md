---
ai_optimized: true
author: Jeremy Tammik
complexity_score: 4.1
content_type: tutorial
optimization_date: '2025-12-11T11:44:15.985252'
original_url: https://thebuildingcoder.typepad.com/blog/1423_wall_opening_areas.html
post_number: '1423'
reading_time_minutes: 11
series: general
slug: wall_opening_areas
source_file: 1423_wall_opening_areas.md
tags:
- doors
- elements
- family
- filtering
- geometry
- parameters
- revit-api
- rooms
- sheets
- views
- walls
- windows
title: Wall Opening Areas
word_count: 2115
---

### Determining Wall Opening Areas per Room
We continue the rather exhaustive exploration of calculating net and gross wall areas per room, and two other announcements, pointers to interesting sources of information besides The Building Coder:
- [Why Autodesk has a Labs](#2)
- [Dal BIM in poi – Italian BIM](#3)
- [Determining wall opening areas per room](#4)
- [Håvard's SpatialElementGeometryCalculator enhancement](#5)
- [External command mainline](#6)
- [Test run](#7)
#### Why Autodesk has a Labs
I hope you know and appreciate that Autodesk has a Labs, and why.
If not, please refer to Scott Sheppard's
explanation [why Autodesk has a Labs](http://labs.blogs.com/its_alive_in_the_lab/2016/04/why-does-autodesk-have-a-labs.html).
#### Dal BIM in Poi – Italian BIM
It is my pleasure to announce a new Italian AEC and BIM blog,
[dal BIM in poi](http://blogs.autodesk.com/dalbiminpoi), by Ilaria Lagazio and Stefano Toparini.
'Dal BIM in poi' means 'Beyond BIM'.
\*Stefano si occupa da molti anni di BIM per le Infrastrutture nonché di reti tecnologiche e di gestione del territorio, con esperienze professionali maturate sia in Italia che all'estero. Lavora nella filiale Italiana di Autodesk Inc. da oltre 10 anni dopo esperienze in varie altre aziende di informatica e servizi nel mondo della progettazione e delle pubbliche amministrazioni.\*
Stefano has been working with BIM for Infrastructure and GIS for many years, with professional experience both in Italy and abroad. He has been with the Italian subsidiary of Autodesk Inc. for over 10 years after experiences in several other computer companies and services in the design and public administration domains.
\*Ilaria e Laureata in Ingegneria Civile e specializzata in Strutture, dopo una breve esperienza nel campo della progettazione si dedica all’industrializzazione dei sistemi edilizi come Building System Development Manager, gestendo il flusso delle informazioni dei componenti edilizi dal modello al cantiere. L’interesse per l’industrializzazione del cantiere e la gestione del dato progettuale la porta ad una esperienza negli Emirati Arabi e dal 2008 in Autodesk, dove oggi ricopre il ruolo di Senior Technical Sales Specialist.\*
Ilaria graduated in civil engineering and specialised in structures. After a short experience in design she worked as Building System Development Manager, managing the information flow from BIM to construction site. Her interest in efficient construction site and project management took her to the UAE, and from 2008 onwards to Autodesk in the role as Senior Technical Sales Specialist.
I wish the new blog and all its readers all the best and many exciting stories!
#### Determining Wall Opening Areas per Room
As I pointed out last week in the preceding article on this topic, Håvard Dagsvik of the newly renamed Scandinavian AEC and BIM company [Symetri](http://www.symetri.com), previously CAD-Q, invested significant effort in enhancing our joint efforts to determine wall opening areas per room.
Here are links to the previous discussions:
- [Calculating gross and net wall areas](http://thebuildingcoder.typepad.com/blog/2015/03/calculating-gross-and-net-wall-areas.html)
- [IFCExportUtils to determine the door and window area](http://thebuildingcoder.typepad.com/blog/2015/03/ifcexportutils-methods-determine-door-and-window-area.html)
- [Determining wall cut area for a specific room](http://thebuildingcoder.typepad.com/blog/2016/04/determining-wall-cut-area-for-a-specific-room.html)
The results are captured in these two Revit add-in projects and GitHub repositories:
- [SpatialElementGeometryCalculator](https://github.com/jeremytammik/SpatialElementGeometryCalculator)
- [ExporterIfcUtilsWinArea](https://github.com/jeremytammik/ExporterIfcUtilsWinArea)
#### Håvard's SpatialElementGeometryCalculator Enhancement
I integrated Håvard's final running test code into
the [SpatialElementGeometryCalculator project](https://github.com/jeremytammik/SpatialElementGeometryCalculator) and
am very happy to present the following results that you can now reproduce yourself as well.
In Håvard's own words:
Here it is.
Simplified and cleaned up.
I couldn’t use the existing dictionary so I implemented a small class `SpatialBoundaryCache` to cache the core spatial data.
I demonstrate how you can structure and present the resulting data, areas square metres, in three different ways:
- By room
- By wall type
- By wall material
Material is my choice because I need to know if the surface is plaster or concrete – note that walls can have different layers on each side.
Just as before, I use both geometric analysis based on the wall solids as well as the IFC utility classes for the calculations.
I removed the `GetAreaFromFamilyParameters` method because it is somewhat unreliable and I already use the IFC utils instead.
I also used your `IsInRoom` method instead of mine.
In the long run, I think all of it will be just solid intersections also IsInRoom.
Both stacked walls and embedded curtain walls are handled, though probably not yet an embedded curtain wall within a stacked wall.
That could be solved following the same solid intersection logic.
I am quite sure there is still room for improvement in the opening handler.
Take a look and see what you make of it :-)
I am attaching [the sample file](zip/SpatialElementGeometryCalculatorTest.rvt) that I worked with as well.
Many thanks to Håvard for his research, hard work, and sharing this!
I integrated Håvard's functionality in the
public [SpatialElementGeometryCalculator GitHub project](https://github.com/jeremytammik/SpatialElementGeometryCalculator),
in [release 2016.0.0.3](https://github.com/jeremytammik/SpatialElementGeometryCalculator/releases/tag/2016.0.0.3).
#### External Command Mainline
The main execute method demonstrates how to:
- Retrieve the rooms from the model
- Run the `SpatialElementGeometryCalculator` calculator
- Retrieve the results
- Extract, sort and report the data
Here is the complete implementation:

```
public Result Execute(
  ExternalCommandData commandData,
  ref string message,
  ElementSet elements )
{
  UIApplication uiapp = commandData.Application;
  Document doc = uiapp.ActiveUIDocument.Document;
  Result rc;

  try
  {
    SpatialElementBoundaryOptions sebOptions
      = new SpatialElementBoundaryOptions {
        SpatialElementBoundaryLocation
          = SpatialElementBoundaryLocation.Finish };

    IEnumerable<Element> rooms
      = new FilteredElementCollector( doc )
        .OfClass( typeof( SpatialElement ) )
        .Where<Element>( e => (e is Room) );

    List<string> compareWallAndRoom = new List<string>();
    OpeningHandler openingHandler = new OpeningHandler();

    List<SpatialBoundaryCache> lstSpatialBoundaryCache
      = new List<SpatialBoundaryCache>();

    foreach( Room room in rooms )
    {
      if( room == null ) continue;
      if( room.Location == null ) continue;
      if( room.Area.Equals( 0 ) ) continue;

      Autodesk.Revit.DB.SpatialElementGeometryCalculator calc =
        new Autodesk.Revit.DB.SpatialElementGeometryCalculator(
          doc, sebOptions );

      SpatialElementGeometryResults results
        = calc.CalculateSpatialElementGeometry(
          room );

      Solid roomSolid = results.GetGeometry();

      foreach( Face face in results.GetGeometry().Faces )
      {
        IList<SpatialElementBoundarySubface> boundaryFaceInfo
          = results.GetBoundaryFaceInfo( face );

        foreach( var spatialSubFace in boundaryFaceInfo )
        {
          if( spatialSubFace.SubfaceType
            != SubfaceType.Side )
          {
            continue;
          }

          SpatialBoundaryCache spatialData
            = new SpatialBoundaryCache();

          Wall wall = doc.GetElement( spatialSubFace
            .SpatialBoundaryElement.HostElementId )
              as Wall;

          if( wall == null )
          {
            continue;
          }

          WallType wallType = doc.GetElement(
            wall.GetTypeId() ) as WallType;

          if( wallType.Kind == WallKind.Curtain )
          {
            // Leave out, as curtain walls are not painted.

            LogCreator.LogEntry( "WallType is CurtainWall" );

            continue;
          }

          HostObject hostObject = wall as HostObject;

          IList<ElementId> insertsThisHost
            = hostObject.FindInserts(
              true, false, true, true );

          double openingArea = 0;

          foreach( ElementId idInsert in insertsThisHost )
          {
            string countOnce = room.Id.ToString()
              + wall.Id.ToString() + idInsert.ToString();

            if( !compareWallAndRoom.Contains( countOnce ) )
            {
              Element elemOpening = doc.GetElement(
                idInsert ) as Element;

              openingArea = openingArea
                + openingHandler.GetOpeningArea(
                  wall, elemOpening, room, roomSolid );

              compareWallAndRoom.Add( countOnce );
            }
          }

          // Cache SpatialElementBoundarySubface info.

          spatialData.roomName = room.Name;
          spatialData.idElement = wall.Id;
          spatialData.idMaterial = spatialSubFace
            .GetBoundingElementFace().MaterialElementId;
          spatialData.dblNetArea = Util.sqFootToSquareM(
            spatialSubFace.GetSubface().Area );
          spatialData.dblOpeningArea = Util.sqFootToSquareM(
            openingArea );

          lstSpatialBoundaryCache.Add( spatialData );

        } // end foreach subface from which room bounding elements are derived

      } // end foreach Face

    } // end foreach Room

    List<string> t = new List<string>();

    List<SpatialBoundaryCache> groupedData
      = SortByRoom( lstSpatialBoundaryCache );

    foreach( SpatialBoundaryCache sbc in groupedData )
    {
      t.Add( sbc.roomName
        + "; all wall types and materials: "
        + sbc.AreaReport );
    }

    Util.InfoMsg2( "Total Net Area in m2 by Room",
      string.Join(System.Environment.NewLine, t ) );

    t.Clear();

    groupedData = SortByRoomAndWallType(
      lstSpatialBoundaryCache );

    foreach( SpatialBoundaryCache sbc in groupedData )
    {
      Element elemWall = doc.GetElement(
        sbc.idElement ) as Element;

      t.Add( sbc.roomName + "; " + elemWall.Name
        + "(" + sbc.idElement.ToString() + "): "
        + sbc.AreaReport );
    }

    Util.InfoMsg2( "Net Area in m2 by Wall Type",
      string.Join( System.Environment.NewLine, t ) );

    t.Clear();

    groupedData = SortByRoomAndMaterial(
      lstSpatialBoundaryCache );

    foreach( SpatialBoundaryCache sbc in groupedData )
    {
      string materialName
        = (sbc.idMaterial == ElementId.InvalidElementId)
          ? string.Empty
          : doc.GetElement( sbc.idMaterial ).Name;

      t.Add( sbc.roomName + "; " + materialName + ": "
        + sbc.AreaReport );
    }

    Util.InfoMsg2(
      "Net Area in m2 by Outer Layer Material",
      string.Join( System.Environment.NewLine, t ) );

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

///
/// Convert square feet to square meters
/// with two decimal places precision.
///
static double SqFootToSquareM( double sqFoot )
{
  return Math.Round( sqFoot * 0.092903, 2 );
}

List<SpatialBoundaryCache> SortByRoom(
  List<SpatialBoundaryCache> lstRawData )
{
  var sortedCache
    = from rawData in lstRawData
      group rawData by new { room = rawData.roomName }
        into sortedData
        select new SpatialBoundaryCache()
        {
          roomName = sortedData.Key.room,
          idElement = ElementId.InvalidElementId,
          dblNetArea = sortedData.Sum( x => x.dblNetArea ),
          dblOpeningArea = sortedData.Sum(
            y => y.dblOpeningArea ),
        };

  return sortedCache.ToList();
}

List<SpatialBoundaryCache> SortByRoomAndWallType(
  List<SpatialBoundaryCache> lstRawData )
{
  var sortedCache
    = from rawData in lstRawData
      group rawData by new
      {
        room = rawData.roomName,
        wallid = rawData.idElement
      }
        into sortedData
        select new SpatialBoundaryCache()
        {
          roomName = sortedData.Key.room,
          idElement = sortedData.Key.wallid,
          dblNetArea = sortedData.Sum( x => x.dblNetArea ),
          dblOpeningArea = sortedData.Sum(
            y => y.dblOpeningArea ),
        };

  return sortedCache.ToList();
}

List<SpatialBoundaryCache> SortByRoomAndMaterial(
  List<SpatialBoundaryCache> lstRawData )
{
  var sortedCache
    = from rawData in lstRawData
      group rawData by new
      {
        room = rawData.roomName,
        mid = rawData.idMaterial
      }
        into sortedData
        select new SpatialBoundaryCache()
        {
          roomName = sortedData.Key.room,
          idMaterial = sortedData.Key.mid,
          dblNetArea = sortedData.Sum( x => x.dblNetArea ),
          dblOpeningArea = sortedData.Sum(
            y => y.dblOpeningArea ),
        };

  return sortedCache.ToList();
}
```

#### Test Run
Here are the results from a test run on the sample model provided:
The simple test model looks like this in plan view:
![Test model plan view](img/SpatialElementGeometryCalculator1Test.png)
In 3D view, you can see the stacked wall and the opening spanning several different spaces:
![Test model 3D view](img/SpatialElementGeometryCalculator2Test3d.png)
The results sorted by room, wall type and material are presented in three sequential task dialogues.
The first one, by room, is the simplest:
![Net, opening and gross area by room](img/SpatialElementGeometryCalculator6AreaByRoom2.png)
Each room is bounded by several different wall types:
![Net, opening and gross area by wall type](img/SpatialElementGeometryCalculator7AreaByWallType2.png)
Finally, here are the areas grouped by material:
![Net, opening and gross area by material](img/SpatialElementGeometryCalculator8AreaByMaterial2.png)
For the sake of the search engines and legibility, here are the same results in text form as well, as reported on the Visual Studio debug console:

```
Total Net Area in m2 by Room

Room 3; all wall types and materials: net 150; opening 9.2; gross 159.2
Room 5; all wall types and materials: net 42; opening 3.97; gross 45.97
Room 6; all wall types and materials: net 120; opening 7; gross 127
Room 7; all wall types and materials: net 120; opening 2; gross 122

Net Area in m2 by Wall Type

Room 3; Exterior - Block on Mtl. Stud(308814): net 30; opening 0; gross 30
Room 3; CW 102-85-215p(309825): net 45; opening 4; gross 49
Room 3; Wall4(310052): net 30; opening 0; gross 30
Room 3; Wall1(308815): net 45; opening 5.2; gross 50.2
Room 5; Wall1(308815): net 9; opening 1.97; gross 10.97
Room 5; Exterior - Block on Mtl. Stud(311213): net 3.6; opening 0.9; gross 4.5
Room 5; Exterior - Brick on Mtl. Stud(311214): net 8.4; opening 1.1; gross 9.5
Room 5; Wall1(310409): net 9; opening 0; gross 9
Room 5; Generic - 225mm Masonry(308816): net 12; opening 0; gross 12
Room 6; Exterior - Block on Mtl. Stud(308814): net 30; opening 0; gross 30
Room 6; Wall1(308815): net 30; opening 3; gross 33
Room 6; Generic - 225mm Masonry(308816): net 30; opening 2; gross 32
Room 6; Wall3(308817): net 30; opening 2; gross 32
Room 7; Exterior - Block on Mtl. Stud(308814): net 30; opening 0; gross 30
Room 7; Wall3(308817): net 30; opening 2; gross 32
Room 7; Generic - 225mm Masonry(308816): net 30; opening 0; gross 30
Room 7; Generic - 225mm Masonry(309758): net 30; opening 0; gross 30

Net Area in m2 by Outer Layer Material

Room 3; Gypsum Wall Board: net 75; opening 4; gross 79
Room 3; Default Wall: net 75; opening 5.2; gross 80.2
Room 5; Default Wall: net 18; opening 1.97; gross 19.97
Room 5; Gypsum Wall Board: net 12; opening 2; gross 14
Room 5; Concrete Masonry Units: net 12; opening 0; gross 12
Room 6; Gypsum Wall Board: net 30; opening 0; gross 30
Room 6; Default Wall: net 60; opening 5; gross 65
Room 6; Concrete Masonry Units: net 30; opening 2; gross 32
Room 7; Gypsum Wall Board: net 30; opening 0; gross 30
Room 7; Default Wall: net 30; opening 2; gross 32
Room 7; Concrete Masonry Units: net 60; opening 0; gross 60
```

Wonderful, isn't it?
Thank you again, Håvard!
#### Addendum
[Zhbing0322](https://github.com/zhbing0322) added
a [comment](https://github.com/jeremytammik/SpatialElementGeometryCalculator/commit/18ebcd1283bf64a595505871987c3089d53037e6#commitcomment-19391593) on
a [commit](https://github.com/jeremytammik/SpatialElementGeometryCalculator/commit/18ebcd1283bf64a595505871987c3089d53037e6):
> The result of `spatialSubFace.GetSubface().Area` is the Gross area of the Wall in a room.