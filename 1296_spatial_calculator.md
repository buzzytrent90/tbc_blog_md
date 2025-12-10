---
post_number: "1296"
title: "Calculating Gross and Net Wall Areas"
slug: "spatial_calculator"
author: "Jeremy Tammik"
tags: ['csharp', 'doors', 'elements', 'family', 'filtering', 'geometry', 'parameters', 'python', 'revit-api', 'rooms', 'transactions', 'vbnet', 'views', 'walls', 'windows']
source_file: "1296_spatial_calculator.htm"
original_url: "https://thebuildingcoder.typepad.com/blog/1296_spatial_calculator.html"
---

### Calculating Gross and Net Wall Areas

Determination of gross and net areas and volumes is fundamental to BIM.

Here is an interesting solution to determine the gross and net area of a wall, i.e. with and without its openings, making use of the SpatialElementGeometryCalculator and the
[temporary transaction trick](http://thebuildingcoder.typepad.com/blog/2012/10/the-temporary-transaction-trick-for-gross-slab-data.html).

The question was raised and solved by
Phillip Miller of Kiwi Codes Solutions Ltd, starting with the
[Revit API discussion forum](http://forums.autodesk.com/t5/revit-api/bd-p/160) thread on
[door/window areas](http://forums.autodesk.com/t5/revit-api/door-window-areas/td-p/5535565):

**Question:** I need to determine the cut areas of windows and doors in a wall. I thought this would be a simple matter of grabbing the BIP "HOST\_Area\_Computed" as with my testing that was returning the correct area. It turns out that this is not the case from further testing.

Please refer to the attached RVT file with two doors inserted into a wall. They look similar but one is reporting through RevitLookup 4m2 and the other 2m2.

Why is this? What actually is "HOST\_Area\_Computed" reporting? What is the most reliable way of getting the cut area of windows and doors?

**Answer:** I cannot really say anything about the values reported by the HOST\_Area\_Computed parameter. That belongs to the user interface, as far as I am concerned, and you would have to ask a product usage expert or application engineer to explain the meaning of that.

For a pure API perspective, I assume you are aware of the
[Revit SDK MaterialQuantities sample](http://thebuildingcoder.typepad.com/blog/2010/02/material-quantity-extraction.html)?

Does that not give you pretty much exactly what you need?

**Response:** Thank you for your reply and also the link to the SDK example. I will look into that in the morning.

To clarify what I'm doing, is, I want to find the surface areas of walls that are associated with Rooms. To do this I'm making use of the SpatialElementGeometryCalculator class that is perfect as it does give areas of walls, floor and ceilings even if they are angled. The only problem is it doesn't subtract things like doors and windows.

Your link to your blog has jogged my memory though with a workaround that I could implement. I know what doors and windows are associated with the room and also the wall associated with the rooms face, so I could get the area of the wall (which does calculate the gross - openings), then remove the associated doors and windows grab the new area and minus it from the gross and then turn back the transaction.
I'm pretty sure that will then give me the true area of the room's walls.

**Answer:** Your use of the SpatialElementGeometryCalculator sounds very interesting.

The Building Coder currently does not provide any samples of using that except the rather complex
[space adjacency for heat load calculation](http://thebuildingcoder.typepad.com/blog/2013/07/football-and-space-adjacency-for-heat-load-calculation.html#3).

**Response:** Thank you so much for your suggestions. The good news is that the temporary deletion of cutting objects in the wall works very well. I'm not to sure why the BIP Area parameter is reporting odd values but I suppose that does not matter any more.

I took a look at your suggested SDK examples and they didn't really apply to my situation as I was requiring the surface areas of walls associated to rooms.
As a wall can span multiple rooms I had two choices.
First up I used the room bounding object to extract the wall information.
The downside to this approach is that you had to take the boundary segment length and then multiply that by the wall height, hoping that the wall had a consistent wall height (not always the case).
I also was requiring the floor and ceiling areas and information and this was achieved by shooting rays from the room location point directly up and down to gather that information. We found the results in the approach to be a bit unreliable and not very accurate at times depending on how the walls etc. were modelled.

That led us to investigate the "SpatialElementGeometryCalculator" class, which after some trial and error does exactly what we were after. I'm happy to provide you with some small sample code of this working for your users.

**Answer:** It seems to me the one of the door areas seems to be counting something double.

Yes, of course the spatial element geometry calculator is much more suited to the need you describe.

**Response:** Here is a very simple stripped down version of the above calculating the wall surface area, minus cutting family instances.

The advantages of this method over any other method that I have seen so far is that if your ceilings are room bounding and they are vaulted the wall areas are calculated correctly.

The solution is a VB.NET solution that could easily be converted to C# if needed (copy to an editor to see the truncated lines in full, or look at the module
[extCmd.vb on GitHub](https://github.com/jeremytammik/SpatialElementGeometryCalculator/blob/master/SpatialElementGeometryCalculatorVb/extCmd.vb):

```vbnet
Imports Autodesk.Revit.ApplicationServices
Imports System.IO
Imports Autodesk.Revit.DB.Architecture
Imports System.Collections.Generic
Imports System.Diagnostics
Imports Autodesk.Revit.Attributes
Imports Autodesk.Revit.DB
Imports BoundarySegment = Autodesk.Revit.DB.BoundarySegment
Imports Autodesk.Revit.DB.ExternalService
Imports Autodesk.Revit.UI

<Autodesk.Revit.Attributes.Transaction(Autodesk.Revit.Attributes.TransactionMode.Manual)> \_
<Autodesk.Revit.Attributes.Regeneration(Autodesk.Revit.Attributes.RegenerationOption.Manual)> \_
<Autodesk.Revit.Attributes.Journaling(Autodesk.Revit.Attributes.JournalingMode.NoCommandData)> \_
Public Class extCmd
  Implements IExternalCommand

  Public Function Execute(commandData As ExternalCommandData, ByRef message As String, elements As ElementSet) As Result Implements IExternalCommand.Execute
    Try

      Dim app As Autodesk.Revit.UI.UIApplication = commandData.Application
      Dim doc As Document = app.ActiveUIDocument.Document

      Dim roomCol As FilteredElementCollector = New FilteredElementCollector(app.ActiveUIDocument.Document).OfClass(GetType(SpatialElement))
      Dim s As String = "Finished populating Rooms with Boundary Data" + vbNewLine + vbNewLine
      For Each e As SpatialElement In roomCol
        Dim room As Room = TryCast(e, Room)
        If room IsNot Nothing Then
          Try
            Dim spatialElementBoundaryOptions As New SpatialElementBoundaryOptions()
            spatialElementBoundaryOptions.SpatialElementBoundaryLocation = SpatialElementBoundaryLocation.Finish
            Dim calculator1 As New Autodesk.Revit.DB.SpatialElementGeometryCalculator(doc, spatialElementBoundaryOptions)
            Dim results As SpatialElementGeometryResults = calculator1.CalculateSpatialElementGeometry(room)

            ' get the solid representing the room's geometry
            Dim roomSolid As Solid = results.GetGeometry()
            Dim iSegment As Integer = 1
            For Each face As Face In roomSolid.Faces
              Dim subfaceList As IList(Of SpatialElementBoundarySubface) = results.GetBoundaryFaceInfo(face)
              For Each subface As SpatialElementBoundarySubface In subfaceList
                If subface.SubfaceType = SubfaceType.Side Then ' only interested in walls in this example
                  Dim ele As Element = doc.GetElement(subface.SpatialBoundaryElement.HostElementId)
                  Dim subfaceArea As Double = subface.GetSubface().Area
                  Dim netArea As Double = sqFootToSquareM(calwallAreaMinusOpenings(subfaceArea, ele, doc, room))
                  s += "Room " + room.Parameter(BuiltInParameter.ROOM\_NUMBER).AsString + " : Wall " + ele.Parameter(BuiltInParameter.DOOR\_NUMBER).AsString + " : Area " + netArea.ToString + "m2" + vbNewLine
                End If

              Next
            Next
            s += vbNewLine
          Catch ex As Exception
          End Try

        End If

      Next
      MsgBox(s, MsgBoxStyle.Information, "Room Boundaries")
      Return Result.Succeeded
    Catch ex As Exception
      MsgBox(ex.Message.ToString + vbNewLine +
       ex.StackTrace.ToString)
      Return Result.Failed
    End Try
  End Function

  Private Function calwallAreaMinusOpenings(ByVal subfaceArea As Double, ByVal ele As Element, ByVal doc As Document, ByVal room As Room) As Double
    Dim fiCol As FilteredElementCollector = New FilteredElementCollector(doc).OfClass(GetType(FamilyInstance))
    Dim lstTotempDel As New List(Of ElementId)
    'Now find the familyInstances that are associated to the current room
    For Each fi As FamilyInstance In fiCol
      If fi.Parameter(BuiltInParameter.HOST\_ID\_PARAM).AsValueString = ele.Id.ToString Then
        If fi.Room IsNot Nothing Then
          If fi.Room.Id = room.Id Then
            lstTotempDel.Add(fi.Id)
            Continue For
          End If
        End If
        If fi.FromRoom IsNot Nothing Then
          If fi.FromRoom.Id = room.Id Then
            lstTotempDel.Add(fi.Id)
            Continue For
          End If
        End If
        If fi.ToRoom IsNot Nothing Then
          If fi.ToRoom.Id = room.Id Then
            lstTotempDel.Add(fi.Id)
            Continue For
          End If
        End If
      End If
    Next

    If lstTotempDel.Count > 0 Then
      Dim t As New Transaction(doc, "tmp Delete")
      Dim wallnetArea As Double = ele.Parameter(BuiltInParameter.HOST\_AREA\_COMPUTED).AsDouble
      t.Start()
      doc.Delete(lstTotempDel)
      doc.Regenerate()
      Dim wallGrossArea As Double = ele.Parameter(BuiltInParameter.HOST\_AREA\_COMPUTED).AsDouble
      t.RollBack()
      Dim fiArea As Double = wallGrossArea - wallnetArea
      Return subfaceArea - fiArea
    Else
      Return subfaceArea
    End If
  End Function

  Private Function sqFootToSquareM(ByVal sqFoot As Double) As Double
    Return Math.Round(sqFoot \* 0.092903, 2)
  End Function

End Class
```

**Answer:**
Thank you very much for your sample code.

I created a
[SpatialElementGeometryCalculator GitHub repository](https://github.com/jeremytammik/SpatialElementGeometryCalculator) for
it and placed both the original VB.NET implementation and my migration efforts to C# in it.

You can look at the
[list of commits](https://github.com/jeremytammik/SpatialElementGeometryCalculator/commits/master) to
see the steps that I took:

- Commits on Mar 17, 2015

- added comments – an hour ago
- added comments – an hour ago
- reformatted long lines and compare element id integer instead of string – an hour ago
- added comments – 2 hours ago
- cleaned up foreach statement and compiled successfully – 2 hours ago

- Commits on Mar 16, 2015

- C# code generated by disassembling VB.NET assembly using Reflector – 22 hours ago
- original VB.NET implementation – 23 hours ago
- added link to discussion forum thread – 23 hours ago
- added authors and tbc link – 23 hours ago
- added authors and tbc link – 23 hours ago
- Initial commit – 23 hours ago

I started off by compiling, installing and testing the VB.NET solution, then disassembling the VB.NET add-in assembly DLL using
[Reflector](http://thebuildingcoder.typepad.com/blog/2008/10/converting-between-vb-and-c-and-net-decompilation.html) to
generate some raw C# code as a starting point.

The result was not ready to compile, so some additional steps were required, both just to compile it and for further clean-up.

One little enhancement that I added was to use an integer comparison instead of string comparison to identify the family instances hosted by the given wall.
Using `AsElementId().IntegerValue.Equals( Id.IntegerValue )` is more efficient than comparing AsValueString and ele.Id.ToString, since the string conversion is eliminated.
Integer comparison is faster, because you are just comparing one single binary register instead of a whole sequence of bytes.

Another significant potential enhancement in calwallAreaMinusOpenings would be to use a parameter filter for the HOST\_ID\_PARAM parameter value instead of the .NET element id comparison to narrow down the family instances to those hosted by the wall before returning the filtered element collector result from Revit back to .NET.

That would vastly speed up the process for large models, because in the current implementation, all family instances are retrieved and marshalled across from native Revit to the .NET add-in memory space before the parameter value is checked in .NET.

As I pointed out repeatedly in the past, just marshalling all of the family instance data from the Revit memory space over to the add-in .NET realm costs a lot of time. Using a parameter filter speeds things up significantly, at least by 50 percent. In a large model, where the great majority of the family instances will not be hosted by the wall of interest, this could cause a huge performance improvement.

The end result looks like this in C#:

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

    Debug.Assert(
      room.Document.ProjectInformation.UniqueId.Equals(
        doc.ProjectInformation.UniqueId ),
      "expected wall and room from same document" );

    // Determine all openings in the given wall.

    FilteredElementCollector fiCol
      = new FilteredElementCollector( doc )
        .OfClass( typeof( FamilyInstance ) );

    List<ElementId> lstTotempDel
      = new List<ElementId>();

    foreach( FamilyInstance fi in fiCol )
    {
      // The family instances hosted by this wall
      // could be filtered out more efficiently using
      // a filtered element collector parameter filter.
      // This would be important in a large model.

      if( fi.get\_Parameter(
        BuiltInParameter.HOST\_ID\_PARAM )
          .AsElementId().IntegerValue.Equals(
            wall.Id.IntegerValue ) )
      {
        if( ( fi.Room != null )
          && ( fi.Room.Id == room.Id ) )
        {
          lstTotempDel.Add( fi.Id );
        }
        else if( ( fi.FromRoom != null )
          && ( fi.FromRoom.Id == room.Id ) )
        {
          lstTotempDel.Add( fi.Id );
        }
        else if( ( fi.ToRoom != null )
          && ( fi.ToRoom.Id == room.Id ) )
        {
          lstTotempDel.Add( fi.Id );
        }
      }
    }

    // Determine total area of all openings.

    double openingArea = 0;

    if( 0 < lstTotempDel.Count )
    {
      Transaction t = new Transaction( doc );

      double wallAreaNet = wall.get\_Parameter(
        BuiltInParameter.HOST\_AREA\_COMPUTED )
          .AsDouble();

      t.Start( "tmp Delete" );
      doc.Delete( lstTotempDel );
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
      TaskDialog.Show( "Room Boundaries", s);

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

Let's look at the results obtained from the little house model generated by the ADN Xtra labs:

![Little house 3D view](img/segc_little_house_3d.png)

The front wall hosts a door and two windows, so its area differs from the back wall's:

![Little house plan view](img/segc_little_house_plan.png)

The add-in reports the following net wall areas:

![Little house wall area results](img/segc_result_msg.png)

As you can see, the door and the two small windows consume a bit less than 2.5 square meters of wall area.

The complete source code, Visual Solution files and add-in manifests for both the VB.NET and C# implementations are hosted by the
[SpatialElementGeometryCalculator GitHub repository](https://github.com/jeremytammik/SpatialElementGeometryCalculator),
and the version presented here is
[release 2015.0.0.0](https://github.com/jeremytammik/SpatialElementGeometryCalculator/releases/tag/2015.0.0.0).

Many thanks to Phillip for exploring and developing this powerful solution!

#### Wall Area and Orientation

Another recent query that might also make use of this discussion was raised by Arif Hanif in a
[comment](http://thebuildingcoder.typepad.com/blog/2009/01/room-and-wall-adjacency.html?cid=6a00e553e16897883301b8d0ec5711970c#comment-6a00e553e16897883301b8d0ec5711970c) on
[room and wall adjacency](http://thebuildingcoder.typepad.com/blog/2009/01/room-and-wall-adjacency.html):

**Question:** Love the website, I have recently started to learn C# and have started exploring the API to help with MEP Design. It has been frustrating dealing with gbXML. I am trying to get Wall Area and Orientation for Room to Excel. Can you point me to some of your past postings that could help me.

**Answer:** There are a lot of discussions on getting the wall area, e.g. for mass quantities, the
[Revit SDK MaterialQuantities sample](http://thebuildingcoder.typepad.com/blog/2010/02/material-quantity-extraction.html) mentioned
above and the discussion of
[2D polygon areas and outer loop](http://thebuildingcoder.typepad.com/blog/2008/12/2d-polygon-areas-and-outer-loop.html).

For the orientation, here are some starting points:

- [Azimuth](http://thebuildingcoder.typepad.com/blog/2008/10/azimuth.html)
- [Facing North](http://thebuildingcoder.typepad.com/blog/2010/01/project-location.html)

Those initial ponderings led to these further discussions:

- [Unrotate North](http://thebuildingcoder.typepad.com/blog/2009/10/unrotate-north.html)
- [Rotate True North](http://thebuildingcoder.typepad.com/blog/2012/03/rotate-true-north.html)
- [Real-world concrete corner coordinates](http://thebuildingcoder.typepad.com/blog/2012/06/real-world-concrete-corner-coordinates.html)

We put in some additional work on converting Revit project coordinates to real-world ones, including the orientation transformation, when working on the setout point application:

- [Structural concrete setout point add-in](http://thebuildingcoder.typepad.com/blog/2012/08/structural-concrete-setout-point-add-in.html)
- [Concrete setout points for Revit Structure 2015](http://thebuildingcoder.typepad.com/blog/2014/11/concrete-setout-points-for-revit-structure-2015.html)

I hope this helps.

Enjoy.