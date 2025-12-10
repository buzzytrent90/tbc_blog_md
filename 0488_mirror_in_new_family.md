---
post_number: "0488"
title: "Mirroring in a New Family and Changing the Active View"
slug: "mirror_in_new_family"
author: "Jeremy Tammik"
tags: ['csharp', 'elements', 'family', 'filtering', 'levels', 'references', 'revit-api', 'schedules', 'transactions', 'views', 'walls', 'windows']
source_file: "0488_mirror_in_new_family.htm"
original_url: "https://thebuildingcoder.typepad.com/blog/0488_mirror_in_new_family.html"
---

### Mirroring in a New Family and Changing the Active View

I spent the past weekend in the desert outside Las Vegas climbing in the daytime and watching the beautiful waning and still powerful half moon and lots of stars at night.
I also saw a couple of shooting stars, and surprised some
[wild burros](http://en.wikipedia.org/wiki/Donkey) in
the dark as well.
I was lucky enough to meet a very nice couple, Joachim and Kathleen.
We did some climbs together on Oasis, then the multi-pitch
[Cat in a Hat](http://www.summitpost.org/cat-in-the-hat/157466) on
[Mescalito](http://www.summitpost.org/mescalito/151383) in freezing cold weather, and finally a couple more on the Ultraman wall, before they had to head back home towards the Bay Area.
I did not realise until later that I had
[been on Ultraman previously](http://thebuildingcoder.typepad.com/blog/2009/11/visible-elements.html#redrock) already... it did seem slightly familiar :-)

It was really great to meet so many of my ADN DevTech colleagues for dinner here in the hotel last night!

To celebrate the first day of conferences at Autodesk University, before the scheduled classes begin, here is a tricky and efficient multi-step solution to a simple yet convoluted issue by Joe Offord of the facade system expert
[Enclos](http://www.enclos.com).

The original aim is simply to mirror a family instance in a new family document.
We already looked at
[mirroring an element](http://thebuildingcoder.typepad.com/blog/2009/07/mirror-an-element.html) and
[accessing the newly created elements](http://thebuildingcoder.typepad.com/blog/2010/04/retrieving-newly-created-elements-in-revit-2011.html),
but the situation has a twist in the newly created family document.

The first issue that arises is that the mirror command requires a current active view, which is not automatically present in the family document.
Joe discovers a workaround for that issue using the ShowElements method.
It generates an unwanted warning message, so a second step is required to deal with eliminating that as well.

As you can see on reading the final solution carefully, you can use the ShowElements method to change the active view and even switch it between the family and project documents.
The official Revit 2011 API does not provide any method to switch the active view, but using ShowElements can be used to create a workaround for that.

Here is the initial query:

**Question:** I am trying to make a new family (Profile-Mullion.rfa), insert some detail components, and then mirror them about the vertical axis.
When I try and mirror the detail component FamilyInstance it throws an exception saying "Invalid active view!"

The active view of the family document will always be null, because I just created it in the API using Application.NewFamilyDocument(templateFileName).
I tried changing it manually, but the Document.ActiveView property is read-only.
Is there a workaround for this?
I tried using Mirror(Element, Reference) and Mirror(ElementSet, Line), and they both throw the same exception.

**First step and next query:** I developed a workaround for the mirroring issue, but I need some help to make it more efficient.

One way to get a document to change its Active View is to use the UIDocument.ShowElements() method.
For example, in my newly created family I can draw a DetailLine on the Ref. Plan View and then use the ShowElements to change the view.
This will visually open the FamilyDocument on the Revit screen.
However, upon calling ShowElements I get a message saying "There is no open view...searching closed views to find a good view could take a long time. Continue?"

![ShowElements message](img/ShowElements_message.png)

Given that the new family has a small number of views, it doesn't really take much time.
Here is some example code:
```csharp
  ' Mirror element about vertical center line
  Dim pt As New XYZ(0, 1, 0)
  Dim vertAxis As Line = app.Create.NewLine( \_
    origin, pt, True)

  Dim elemSet As New ElementSet
  elemSet.Insert(famInst)

  Dim detLine As DetailLine = familyDoc \_
    .FamilyCreate.NewDetailCurve( \_
      planView, vertAxis)

  Dim uidoc As New UIDocument(familyDoc)

  uidoc.ShowElements(detLine)

  ' ACTIVE VIEW = NULL
  familyDoc.Mirror(elemSet, vertAxis)

  ' Delete the old family instance
  familyDoc.Delete(famInst)

  ' Delete the detail line
  familyDoc.Delete(detLine)
```

Is there a way to automatically hit OK on this dialog box?
If I manually click OK, the mirror command will execute just fine.

I also noticed that the FamilyDocument will remain open even if I execute familyDoc.Close(False).
I assume this is because the screen is now opened on the family document.
Do you see a quick way to resolve this and get back to the project document?

**Answer:** Regarding how to disable the message, yes, there are two possible approaches. The more powerful and complete
solution is to use the
[Failure API](http://thebuildingcoder.typepad.com/blog/2010/04/failure-api.html), which was later
[discussed in more depth](http://thebuildingcoder.typepad.com/blog/2010/11/failure-api-take-two.html) by Joe Ye.
The older and simpler method uses the
[DialogBoxShowing event](http://thebuildingcoder.typepad.com/blog/2009/06/autoconfirm-save-using-dialogboxshowing-event.html).
We discussed two examples comparing and making use of both of these, to handle
[editing elements inside groups](http://thebuildingcoder.typepad.com/blog/2010/08/editing-elements-inside-groups.html) and
[suppression of an unwanted dialogue](http://thebuildingcoder.typepad.com/blog/2010/08/suppress-unwanted-dialogue.html).

Regarding closing the family document, I do not see any officially supported solution.
You could always try the dangerous
[Windows message workaround](http://thebuildingcoder.typepad.com/blog/2010/10/closing-the-active-document-and-why-not-to.html).

**Second step:** I found a good approach to suppress the dialog box from appearing.

First off, the message box that appears does not trigger the IFailuresPreprocessor, so I had to resort to DialogBoxShowingEvent and a global Boolean that could control it.
```vbnet
  ' Application Start-Up
  AddHandler Application.DialogBoxShowing, \_
    AddressOf UIDialogBoxEvent
' . . .
Private Sub UIDialogBoxEvent( \_
  ByVal sender As Object, \_
  ByVal args As DialogBoxShowingEventArgs)
  If CGlobal.SuppressDialogBoxes Then
    args.OverrideResult(DialogResult.OK)
  End If
End Sub
```

Here's what the final code ended up looking like.
This also switches the active view back to the original document:
```vbnet
Public Shared Function NewFamilySymbol( \_
  ByVal projectDoc As Document) \_
  As FamilySymbol
  Dim app As Autodesk.Revit.ApplicationServices \_
    .Application = projectDoc.Application
  Dim templateFileName As String \_
    = "C:\Program Files\Autodesk" \_
      + "\Revit Architecture 2011" \_
      + "\Content\Imperial Templates" \_
      + "\Profile-Mullion.rft"
  Dim familyDoc As Document \_
    = app.NewFamilyDocument(templateFileName)
  If familyDoc Is Nothing Then
    Dim tdError As New TaskDialog("Error")
    tdError.MainInstruction = "Could not find" \_
      + " family template: Profile-Mullion.rft"
    tdError.MainContent = "Designated path '" \_
      + templateFileName + "' is invalid."
    tdError.CommonButtons \_
      = TaskDialogCommonButtons.Close
    tdError.Show()
    Return Nothing
  End If
  ' Store some elements in the original project
  ' that we can use later (that are in the
  ' current ActiveView)
  Dim projDocCollector As New  \_
    FilteredElementCollector( \_
    projectDoc, projectDoc.ActiveView.Id)
  Dim projFamInsts As List(Of ElementId) \_
    = projDocCollector.OfClass( \_
      GetType(FamilyInstance)) \_
      .ToElementIds
  Dim famDocCollector As New  \_
    FilteredElementCollector(familyDoc)
  Dim views As List(Of Element) \_
    = famDocCollector \_
      .OfClass(GetType(View)) \_
      .ToElements
  Dim planView As View = Nothing
  ' Find the default view
  For Each v As View In views
    If v.Name = "Ref. Level" Then
      planView = v
      Exit For
    End If
  Next
  Dim fileName As String = Nothing
  Dim familyDocTrans As New Transaction( \_
    familyDoc, "CreateFamilySymbol")
  familyDocTrans.Start()
  Dim famUIdoc As New UIDocument(familyDoc)
  ' ... Create FamilyInstances in the familyDoc
  ' Mirror elements about vertical centerline
  Dim origin As New XYZ(0, 0, 0)
  Dim pt As New XYZ(0, 1, 0)
  Dim vertAxis As Line = app.Create.NewLine( \_
    origin, pt, True)
  Dim detLine As DetailLine = familyDoc \_
    .FamilyCreate.NewDetailCurve( \_
      planView, vertAxis)
  ' Set up event handler to automatically
  ' click "OK" when the ShowElements dialog
  ' box appears
  CGlobal.SuppressDialogBoxes = True
  Try
    ' Transfer the active view to the
    ' Family(Document)
    famUIdoc.ShowElements(detLine)
    familyDoc.Mirror(detailComps, vertAxis)
    ' Delete the old family instance
    familyDoc.Delete(detailComps)
  Catch ex As Exception
  End Try
  ' Delete the detail line
  familyDoc.Delete(detLine)
  ' Transfer the active view back to the project
  Dim projUIdoc As New UIDocument(projectDoc)
  projUIdoc.ShowElements(projFamInsts.Item(0))
  ' Reset the dialog box event handler
  CGlobal.SuppressDialogBoxes = False
  familyDocTrans.Commit()
  ' Define a save path for the new family
  If familyDoc.SaveAs(fileSavePath) Then
    ' Bring the family into the current project
    projectDoc.LoadFamily( \_
      fileSavePath, projectFam)
    ' Close the family edit,
    ' do not save changes
    familyDoc.Close(False)
    ' Return Family or FamilySymbol
End Function
```

Kind a long solution to a simple mirror issue but hey it works!

Thank you very much, Joe, for this interesting example of perseverance and cascaded workarounds, for your persistent research and complete and interesting solution!