---
ai_optimized: true
author: Jeremy Tammik
complexity_score: 3.2
content_type: qa
optimization_date: '2025-12-11T11:44:14.410777'
original_url: https://thebuildingcoder.typepad.com/blog/0694_avf_hilite_rooms.html
post_number: 0694
reading_time_minutes: 7
series: general
slug: avf_hilite_rooms
source_file: 0694_avf_hilite_rooms.htm
tags:
- csharp
- elements
- filtering
- geometry
- references
- revit-api
- rooms
- selection
- transactions
- views
- walls
title: Using AVF to Display Intersections and Highlight Rooms
word_count: 1353
---

### Using AVF to Display Intersections and Highlight Rooms

We held a DevLab in Gothenburg yesterday with lots of interesting things to discuss with our Scandinavian developer guests.
Today we are in Munich, Germany.
So is
[Kean](http://through-the-interface.typepad.com/through_the_interface/2011/12/back-in-munich.html) :-)

One of the topics that came up yesterday is a thing I have been wanting to write about for over a year now and never gotten around to, the use of the AVF or Analysis Visualisation Framework to display temporary geometry to the user.

I used it myself to display a
[live webcam image on a building element](http://thebuildingcoder.typepad.com/blog/2010/06/display-webcam-image-on-building-element-face.html),
and Saikat used it in the AVF part of his
[structural AVF and DMU sample](http://thebuildingcoder.typepad.com/blog/2010/08/structural-dynamic-model-update-sample.html),
although I only documented the DMU or Dynamic Model Update aspect of it in detail.

This kind of use of the AVF is also demonstrated by the Revit SDK sample GeometryCreation\_BooleanOperation.
It creates geometry solids using the GeometryCreationUtils and BooleanOperationUtils classes and displays them in a view using AVF.

#### FindColumns Using Geometry Creation and Booleans

Transient AVF visualisation was used very impressively in the presentation of new geometry functionality at the developer conferences last year.
You may and should be aware of the
[FindColumns](http://thebuildingcoder.typepad.com/blog/2010/01/findreferencesbydirection.html)
example of using the FindReferencesByDirection method to determine collisions between walls and columns in a situation like this:

![Wall and column intersections](img/wall_column_intersections.png)

As I mentioned in the
[DevDays 2010 Online with Revit 2012 API news](http://thebuildingcoder.typepad.com/blog/2011/04/devdays-2010-online-with-revit-2012-api-news.html#3),
it was converted to use the exact wall and column geometry instead of the original approximate ray tracing approach, and the results are displayed exactly by hiding the wall and column categories in the view and displaying their intersection solids using AVF instead.

It defines two commands:

- HighlightIntersectionsCommand to determine and display the intersections, and- RestoreViewCommand to restore the view afterwards, i.e. reset the wall and column category visibility in the view.

Here is the original 3D view of the situation:

![3D view of wall and column intersections](img/wall_column_intersections_3d.png)

Running HighlightIntersectionsCommand performs the following steps:

- Hide the wall and column categories in the current view.- Set up an AVF view.- Collect all walls and column elements.- Query them for their solid geometry.- Calculate their intersections.- List and display the results.

Here is the list of the resulting intersections:

![Wall and column intersection results](img/wall_column_intersections_result.png)

This is what the intersection geometry looks like in AVF:

![Wall and column intersection solids in AVF](img/wall_column_intersections_avf.png)

As said, you can call RestoreViewCommand afterwards to restore the view by resetting the wall and column category visibility in the view.

For your reference, here is
[Geometry2012.zip](zip/Geometry2012.zip) containing the C# sample code and Visual Studio solution together the suitable sample model to run it in.

So all of that is really important stuff from last year that I have been dying to publish for such a long time and never gotten around to.
Finally!

#### Highlight Rooms Using AVF in VB

I passed the geometry sample on to Betina Mette Zimmermann of
[NTI CAD Center A/S](http://www.nti.dk) in
Denmark, and she converted it to VB and modified it to highlight the room geometry returned by the Room.ClosedShell method instead of wall and column intersections within half an hour:
```vbnet
#Region "Namespaces"
Imports System
Imports System.Collections.Generic
Imports Autodesk.Revit.ApplicationServices
Imports Autodesk.Revit.Attributes
Imports Autodesk.Revit.DB
Imports Autodesk.Revit.DB.Analysis
Imports Autodesk.Revit.DB.Architecture
Imports Autodesk.Revit.UI
Imports Autodesk.Revit.UI.Selection
#End Region

<Transaction(TransactionMode.Manual)>
Public Class Command
  Implements IExternalCommand

  Private schemaId As Integer = -1

  Public Function Execute(
    ByVal commandData As ExternalCommandData,
    ByRef message As String,
    ByVal elements As ElementSet) \_
  As Result Implements IExternalCommand.Execute

    Dim uidoc As UIDocument = commandData.Application.ActiveUIDocument
    Dim doc As Document = uidoc.Document

    Dim transaction As Transaction = New Transaction(doc)

    Try
      Dim col As FilteredElementCollector \_
        = New FilteredElementCollector(uidoc.Document)

      col.WhereElementIsNotElementType.ToElements()
      col.OfCategory(BuiltInCategory.OST\_Rooms)

      Dim value As Double = 1.0
      For Each room As Room In col
        Dim geo As GeometryElement = room.ClosedShell()
        Dim solid As Solid = GetGeometry(geo)
        If Not solid Is Nothing Then
          PaintSolid(doc, solid, value)
        End If
        value = value + 1
      Next

      transaction.Start("Hide Walls")

      Dim categories As Categories \_
        = uidoc.Document.Settings.Categories

      SetCategoryInvisible( \_
        categories, BuiltInCategory.OST\_Walls,
        uidoc.ActiveView)

      transaction.Commit()

      Return Result.Succeeded

    Catch ex As Exception
      message = ex.Message
      Return Result.Failed
    End Try

  End Function

  Public Shared Sub SetCategoryInvisible( \_
    ByVal categories As Categories, \_
    ByVal bic As BuiltInCategory, \_
    ByVal view As View)

    SetCategoryVisibility(categories, bic, view, False)

  End Sub

  Private Shared Sub SetCategoryVisibility( \_
    ByVal categories As Categories, \_
    ByVal bic As BuiltInCategory, \_
    ByVal view As View, \_
    ByVal visible As Boolean)

    Dim category As Category = categories.Item(bic)
    category.Visible(view) = visible

  End Sub

  Public Function GetGeometry( \_
    ByVal geomElem As GeometryElement) As Solid

    For Each geomObj As GeometryObject In geomElem.Objects

      ' Walls and some columns will have a solid
      ' directly in its geometry

      If TypeOf geomObj Is Solid Then
        Dim solid As Solid = DirectCast(geomObj, Solid)
        If solid.Volume > 0 Then
          Return solid
        End If
      End If

      ' Some columns will have a instance
      ' pointing to symbol geometry

      If TypeOf geomObj Is GeometryInstance Then

        Dim geomInst As GeometryInstance \_
          = DirectCast(geomObj, GeometryInstance)

        ' Instance geometry is obtained so that the
        ' intersection works as expected without
        ' requiring transformation

        Dim instElem As GeometryElement \_
          = geomInst.GetInstanceGeometry()

        For Each instObj As GeometryObject In instElem.Objects
          If TypeOf instObj Is Solid Then
            Dim solid As Solid = DirectCast(instObj, Solid)
            If solid.Volume > 0 Then
              Return solid
            End If
          End If
        Next
      End If
    Next
    Return Nothing
  End Function

  Private Sub PaintSolid( \_
    ByVal doc As Document, \_
    ByVal s As Solid, \_
    ByVal value As Double)

    Dim app As Application = doc.Application

    Dim view As View = doc.ActiveView

    If view.AnalysisDisplayStyleId \_
      = ElementId.InvalidElementId Then

      CreateAVFDisplayStyle(doc, view)
    End If

    Dim sfm As SpatialFieldManager \_
      = SpatialFieldManager.GetSpatialFieldManager(view)

    If sfm Is Nothing Then
      sfm = SpatialFieldManager \_
        .CreateSpatialFieldManager(view, 1)
    End If

    If schemaId <> -1 Then
      Dim results As IList(Of Integer) \_
        = sfm.GetRegisteredResults()

      If Not results.Contains(schemaId) Then
        schemaId = -1
      End If
    End If

    If schemaId = -1 Then
      Dim resultSchema1 As New AnalysisResultSchema( \_
        "PaintedSolid", "Description")

      schemaId = sfm.RegisterResult(resultSchema1)
    End If

    Dim faces As FaceArray = s.Faces
    Dim trf As Transform = Transform.Identity

    For Each face As Face In faces
      Dim idx As Integer \_
        = sfm.AddSpatialFieldPrimitive(face, trf)

      Dim uvPts As IList(Of UV) = New List(Of UV)()
      Dim doubleList As New List(Of Double)()

      Dim valList As IList(Of ValueAtPoint) \_
        = New List(Of ValueAtPoint)()

      Dim bb As BoundingBoxUV = face.GetBoundingBox()
      uvPts.Add(bb.Min)
      doubleList.Add(value)
      valList.Add(New ValueAtPoint(doubleList))
      Dim pnts As New FieldDomainPointsByUV(uvPts)
      Dim vals As New FieldValues(valList)

      sfm.UpdateSpatialFieldPrimitive( \_
        idx, pnts, vals, schemaId)
    Next
  End Sub

  Private Sub CreateAVFDisplayStyle( \_
    ByVal doc As Document, \_
    ByVal view As View)

    Dim t As New Transaction(doc)
    t.Start("Create AVF Style")

    Dim coloredSurfaceSettings As \_
      New AnalysisDisplayColoredSurfaceSettings()

    coloredSurfaceSettings.ShowGridLines = True

    Dim colorSettings As \_
      New AnalysisDisplayColorSettings()

    Dim legendSettings As \_
      New AnalysisDisplayLegendSettings()

    legendSettings.ShowLegend = False

    Dim analysisDisplayStyle As AnalysisDisplayStyle \_
      = analysisDisplayStyle.CreateAnalysisDisplayStyle( \_
        doc, "Paint Solid", coloredSurfaceSettings, \_
        colorSettings, legendSettings)

    view.AnalysisDisplayStyleId = analysisDisplayStyle.Id

    t.Commit()
  End Sub

End Class
```

To test this, I created the following Revit model of a cow hotel, implemented according to a design suggested by Jim Quanci in the Gothenburg airport:

![Jim's cow hotel](img/highlight_rooms_cow_hotel.png)

Here are the resulting highlighted rooms using AVF:

![Cow hotel rooms highlighted](img/highlight_rooms_cow_hotel_shaded.png)

Here is
[HighlightRooms.zip](zip/HighlightRooms.zip) containing
the VB code and complete Visual Studio solution for this command.

Many thanks to Betina for implementing and sharing this!