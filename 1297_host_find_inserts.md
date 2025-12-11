---
ai_optimized: true
author: Jeremy Tammik
complexity_score: 4.0
content_type: code_example
optimization_date: '2025-12-11T11:44:15.709216'
original_url: https://thebuildingcoder.typepad.com/blog/1297_host_find_inserts.html
post_number: '1297'
reading_time_minutes: 6
series: general
slug: host_find_inserts
source_file: 1297_host_find_inserts.htm
tags:
- csharp
- doors
- elements
- family
- filtering
- geometry
- parameters
- python
- revit-api
- rooms
- transactions
- vbnet
- walls
- windows
title: FindInserts Retrieves All Openings in All Wall Types
word_count: 1110
---

### FindInserts Retrieves All Openings in All Wall Types

On Tuesday, I presented the new
[SpatialElementGeometryCalculator sample](https://github.com/jeremytammik/SpatialElementGeometryCalculator) for
[calculating gross and net wall areas](http://thebuildingcoder.typepad.com/blog/2015/03/calculating-gross-and-net-wall-areas.html).

It discusses a whole bunch of interesting aspects, e.g.:

- Use of the SpatialElementGeometryCalculator class.
- Porting a VB.NET Revit add-in to C#.
- Use of the [temporary transaction trick](http://thebuildingcoder.typepad.com/blog/2012/10/the-temporary-transaction-trick-for-gross-slab-data.html).
- Use of filtered element collectors to determine all openings in a wall.
- Filtered element collector optimisation possibilities, e.g. integer instead of string comparison, use of a parameter filter instead of .NET post-processing.

Vilo submitted a very relevant
[comment](http://thebuildingcoder.typepad.com/blog/2015/03/calculating-gross-and-net-wall-areas.html?cid=6a00e553e16897883301b8d0eee480970c#comment-6a00e553e16897883301b8d0eee480970c) on that sample, asking:

**Question:**
Why not use `(wall as HostObject).FindInserts(...)` to determine the instances nested into the given wall?

Maybe the method above gives better performance?

**Response:**
That is a very valid question indeed.

The simple answer is: I was unaware of it, in spite of it being used in the
[space adjacency for heat load calculation](http://thebuildingcoder.typepad.com/blog/2013/07/football-and-space-adjacency-for-heat-load-calculation.html#3) sample.

In olden times, methods similar to that shown above were the only way to retrieve this information.

Hence, for instance, this [relationship inverter implementation](http://thebuildingcoder.typepad.com/blog/2008/10/relationship-in.html).

Thank you very much for this valuable hint!

By the way, there is no need to say 'wall as HostObject', because the wall is a HostObject, being derived from it. You can simply use wall. FindInserts directly.

**Later:**
I see what you mean now, of course. In the code above, the 'wall' variable is of type Element. I added the following debug code to verify that at least the number of openings retrieved is equal:

```csharp
  // This approach is much more efficient and
  // entirely avoids the use of all filtered
  // element collectors.

  IList<ElementId> inserts = ( wall as HostObject )
    .FindInserts( true, true, true, true );

  Debug.Assert(
    lstTotempDel.Count.Equals( inserts.Count ),
    "expected FindInserts to return the same openings" );
```

The
[SpatialElementGeometryCalculator GitHub repository](https://github.com/jeremytammik/SpatialElementGeometryCalculator) code
is updated now.

Thank you!

I passed on this information to Phillip Miller of Kiwi Codes Solutions Ltd, who provided the initial VB.NET implementation based on the
[Revit API discussion forum](http://forums.autodesk.com/t5/revit-api/bd-p/160) thread on
[door/window areas](http://forums.autodesk.com/t5/revit-api/door-window-areas/td-p/5535565), and he replies:

**Response:**
You will not believe the timing on yours and Vilo's post.

I had already made the filtered element collector more optimised as you suggested in your blog post, but like you I didn't know about the `FindInserts` method. It rocks.

Just this morning I was contacted by my client saying the custom API works great on walls with hosted families but not with walls that had structure separated with lining walls, e.g., three walls making up one wall with the linings walls joined the structural wall.

As you can see in my code I was comparing FamilyInstance.HostID with the wall.id. In a compound wall situation like above the cut opening in the linings wall has a different ID that the FamilyInstance HostID, so it didn't find any openings.

The `FindInserts` method solved all this in a much nicer way.

**Answer:**
Thank you very much for your confirmation and providing samples in which the `FindInserts` method really makes a significant difference, besides the performance improvement.

Implementing a solution for the compound wall situation would probably be possible using filtered element collectors as well, but much more complex.

The code using FindInserts is a lot simpler and more succinct than the filtered element collector code, even for the non-compound situation.

Here is the updated C# implementation using it:

```python
[Transaction( TransactionMode.Manual )]
public class Command : IExternalCommand
{
  /// <summary>
  /// Convert square feet to square meters
  /// with two decimal places precision.
  /// </summary>
  double sqFootToSquareM( double sqFoot )
  {
    return Math.Round( sqFoot \* 0.092903, 2 );
  }

  /// <summary>
  /// Calculate wall area minus openings. Temporarily
  /// delete all openings in a transaction that is
  /// rolled back.
  /// </summary>
  /// <param name="subfaceArea">Initial gross subface area</param>
  /// <param name="wall"></param>
  /// <param name="doc"></param>
  /// <param name="room"></param>
  /// <returns></returns>
  double calwallAreaMinusOpenings(
    double subfaceArea,
    Element wall,
    Room room )
  {
    Document doc = wall.Document;

    // Is this a reliable way to compare documents?

    Debug.Assert(
      room.Document.ProjectInformation.UniqueId.Equals(
        doc.ProjectInformation.UniqueId ),
      "expected wall and room from same document" );

    // Determine all openings in the given wall.

    IList<ElementId> inserts = ( wall as HostObject )
      .FindInserts( true, true, true, true );

    // Determine total area of all openings.

    double openingArea = 0;

    if( 0 < inserts.Count )
    {
      Transaction t = new Transaction( doc );

      double wallAreaNet = wall.get\_Parameter(
        BuiltInParameter.HOST\_AREA\_COMPUTED )
          .AsDouble();

      t.Start( "tmp Delete" );
      doc.Delete( inserts );
      doc.Regenerate();
      double wallAreaGross = wall.get\_Parameter(
        BuiltInParameter.HOST\_AREA\_COMPUTED )
          .AsDouble();
      t.RollBack();

      openingArea = wallAreaGross - wallAreaNet;
    }

    return subfaceArea - openingArea;
  }

  public Result Execute(
    ExternalCommandData commandData,
    ref string message,
    ElementSet elements )
  {
    UIApplication app = commandData.Application;
    Document doc = app.ActiveUIDocument.Document;

    SpatialElementBoundaryOptions sebOptions
      = new SpatialElementBoundaryOptions();

    sebOptions.SpatialElementBoundaryLocation
      = SpatialElementBoundaryLocation.Finish;

    Result rc;

    try
    {
      FilteredElementCollector roomCol
        = new FilteredElementCollector( doc )
          .OfClass( typeof( SpatialElement ) );

      string s = "Finished populating Rooms with "
        + "Boundary Data\r\n\r\n";

      foreach( SpatialElement e in roomCol )
      {
        Room room = e as Room;

        if( room != null )
        {
          try
          {
            Autodesk.Revit.DB
              .SpatialElementGeometryCalculator
                calc = new Autodesk.Revit.DB
                  .SpatialElementGeometryCalculator(
                    doc, sebOptions );

            SpatialElementGeometryResults results
              = calc.CalculateSpatialElementGeometry(
                room );

            Solid roomSolid = results.GetGeometry();

            foreach( Face face in roomSolid.Faces )
            {
              IList<SpatialElementBoundarySubface>
                subfaceList = results.GetBoundaryFaceInfo(
                  face );

              foreach( SpatialElementBoundarySubface
                subface in subfaceList )
              {
                if( subface.SubfaceType
                  == SubfaceType.Side )
                {
                  Element wall = doc.GetElement(
                    subface.SpatialBoundaryElement
                      .HostElementId );

                  double subfaceArea = subface
                    .GetSubface().Area;

                  double netArea = sqFootToSquareM(
                    calwallAreaMinusOpenings(
                      subfaceArea, wall, room ) );

                  s = s + "Room "
                    + room.get\_Parameter(
                      BuiltInParameter.ROOM\_NUMBER )
                        .AsString()
                    + " : Wall " + wall.get\_Parameter(
                      BuiltInParameter.ALL\_MODEL\_MARK )
                        .AsString()
                    + " : Area " + netArea.ToString()
                    + " m2\r\n";
                }
              }
            }
            s = s + "\r\n";
          }
          catch( Exception )
          {
          }
        }
      }
      TaskDialog.Show( "Room Boundaries", s );

      rc = Result.Succeeded;
    }
    catch( Exception ex )
    {
      TaskDialog.Show( "Room Boundaries",
        ex.Message.ToString() + "\r\n"
        + ex.StackTrace.ToString() );

      rc = Result.Failed;
    }
    return rc;
  }
}
```

I updated the C# implementation, but not the VB.NET one, in the
[SpatialElementGeometryCalculator GitHub repository](https://github.com/jeremytammik/SpatialElementGeometryCalculator) containing
the complete source code, Visual Solution files and add-in manifests for both and tagged the version presented here as
[release 2015.0.0.2](https://github.com/jeremytammik/SpatialElementGeometryCalculator/releases/tag/2015.0.0.2).

Many thanks to Phillip and Vilo for the significant improvement of this important solution!