---
post_number: "1326"
title: "Revit 2016 SP1 and Sheets Missing from Print Dialogue"
slug: "2016_sp1_print_sheets"
author: "Jeremy Tammik"
tags: ['csharp', 'elements', 'filtering', 'parameters', 'revit-api', 'sheets', 'transactions', 'views']
source_file: "1326_2016_sp1_print_sheets.htm"
original_url: "https://thebuildingcoder.typepad.com/blog/1326_2016_sp1_print_sheets.html"
---

### Revit 2016 SP1 and Sheets Missing from Print Dialogue

Today, I can present the explanation and prophylactic measure required to prevent a problem of
[sheets missing from print dialogue](http://forums.augi.com/showthread.php?150007-Sheets-missing-from-print-dialogue).

It was only occasionally reported, and I finally heard about the fact and its resolution last week.

In brief, when creating sheets programmatically,
[only create one sheet per transaction](#3)!

Before looking at that in more detail, I'll just mention that the
[Revit 2016 service pack 1](#2) was
automatically installed for me when I started up Revit today.

First of all, though, here is my pre-full-moon photo of a huge ancient imposing
[lime tree with its perfect offspring](https://en.wikipedia.org/wiki/Tilia) in the moonlight, taken while spending a beautiful starry night sleeping out under a pear tree last weekend:

![Ancient and young lime trees in moonshine](file:////j/photo/jeremy/2015/2015-05-30_wiechs/195_linde_im_mondschein.jpg)

#### Revit 2016 Service Pack 1

As said, when I started up Revit today, the Autodesk application manager automatically detected the need to update and installed
[Revit 2016 SP1 – Service Pack 1 for Autodesk Revit 2016](http://knowledge.autodesk.com/support/revit-products/downloads/caas/downloads/content/autodesk-revit-2016-service-pack-1.html?v=2016) for
me, released 2015-05-27.
A full list of available updates can always be found on the
[Revit products download](http://knowledge.autodesk.com/support/revit-products/downloads) page.

The
[SP1 release notes](http://revit.downloads.autodesk.com/download/2016RVT_SP1/Docs/RelNotes/AutodeskRevit2016-SP1ReleaseNotes.html) list
the issues resolved.

This is the list of API related issues addressed:

- An issue with the DimensionSegment.TextPostion API when handling a dimension with more than one segment.
- Ensure that third-party developers always have the correct value when using the public API to get the upper value or lower value of the conditions.
- An issue copying or mirroring an electrical circuit: ensure that wire types are correctly copied as part of the electrical system.

#### Only Create One Sheet Per Transaction!

Now, to address the main topic.

Last week, a strange issue related to sheets missing in the print dialogue came up:

**Question:**
The majority of our sheets are not showing in the View/Sheet Set selector.
I have tested the files in Revit 2014 and 2015 with the same result.

This is going to be a major issue for with our projects around 1000 sheets;
printing them one at a time is not going to go down well.

This issue has already been raised in the past:
[sheets missing from print dialogue](http://forums.augi.com/showthread.php?150007-Sheets-missing-from-print-dialogue).

Here is the view sheet set dialogue in a Revit file containing two sheets, but only one of them shows up:

![View sheet set dialogue](img/view_sheet_set_dialogue.png)

According to the API, there is a CanBePrinted setting applied to sheets.

I would presume that this setting would be false, but it is true on both sheets, as per RevitLookup result.

I have run the following code across the file:

```csharp
  public void viewsheet( UIDocument uidoc )
  {
    Document doc = uidoc.Document;

    FilteredElementCollector filteredElementCollector
      = new FilteredElementCollector( doc );

    filteredElementCollector.OfClass(
      typeof( ViewSheet ) );

    ViewSheetSetting viewSheetSetting
      = doc.PrintManager.ViewSheetSetting;

    Transaction tr = new Transaction( doc, "test" );
    tr.Start();
    try
    {
      foreach( ViewSheet vs in filteredElementCollector )
      {
        MessageBox.Show( vs.SheetNumber + " + "
          + vs.CanBePrinted.ToString() );

        viewSheetSetting.AvailableViews.Insert( vs );
      }
      tr.Commit();
    }
    catch( Exception ex )
    {
      MessageBox.Show( ex.ToString() );
      tr.RollBack();
    }
    foreach( Autodesk.Revit.DB.View view in
      viewSheetSetting.AvailableViews )
    {
      MessageBox.Show( view.Name + " + "
        + view.CanBePrinted.ToString() );
    }
  }
```

It appears the sheet is not in the AvailableViews; I tried to add it to this set, but it is a read-only set.

The files and sheets it is affecting appear to be random.

It affects a large number of sheets now, which will take months to print individually.

In our smallest project with a few dozens of sheets, it would take more than half an hour to print manually sheet by sheet.

Our larger projects would take an entire week!

I was only using the API to test and see if there was a setting that controls their ability to be seen in the View/Sheet Set dialog, no more. This is not an API support request, unless this can be solved using the API, which I am unsure it can as you have confirmed the ViewSheetSetting.AvailableViews is read-only.

I assume that some non-API setting is responsible for this behaviour.

I am getting real pressure to resolve this.

**Answer:**
The ViewSheetSetting.AvailableViews property is definitely read-only according to the Revit API documentation.

That means that you cannot change it by setting it to be a new collection.

That does not necessarily mean that you cannot modify the contents of the existing collections, though, which is what your sample code snippet is attempting to achieve.

I created an add-in to test your code in the model you provided.

By the way, your way to manage the transaction is not optimal. Encapsulating it within a using block is safer:

- [Using `using` automagically disposes and rolls back](http://thebuildingcoder.typepad.com/blog/2012/04/using-using-automagically-disposes-and-rolls-back.html)
- [Handling transaction status and errors](http://thebuildingcoder.typepad.com/blog/2014/11/handling-transaction-status-and-errors.html)
- [Using transaction groups](http://thebuildingcoder.typepad.com/blog/2015/02/using-transaction-groups.html)

Unfortunately, I cannot run your code, though.

When I try to access the doc.PrintManager.ViewSheetSetting property, it throws an exception saying 'this property is only available when user choose Select of Print Range':

![Exception accessing ViewSheetSetting property](img/view_sheet_setting_exception.png)

Later: we seem to have heard about similar issues in the past, e.g.:

- REVIT-53076 [Sheets are missing from print]
- REVIT-26814 [Sheets do not display in the Print/Export View/Sheet Set dialog]
- REVIT-68252 [sheets missing from print dialogue -- 10776470]

The files have minor corruption, in which sheet tracking lost several sheets.
They can be fixed manually.

Unfortunately, simply upgrading the model does not fix this issue.

In past, this kind of situation has come up when sheets were created by API add-ins such as
[RushForth](http://www.rushforthprojects.com/Revit_To_Excel_and_Shared_Parameters_Tools_Download.html).

Can you verify that a tool such as this was being used in this workflow?

If so, can the maker of the add-in be contacted?

After more analysis: similar problems occurred when users created sheets via API add-ins or macros and then the transaction that created the sheets was undone and redone.
After this sequence of events, Revit will lose track of the IDs of the API-created sheets, and the UI for printing and exporting will fail to report them.

**Response:**
Some, but not all the sheets were created using an API tool that I developed.
There is only one way to create sheets within the API, is there not?
I’m not sure if the API can be blamed, as there are also sheets that have not been created via the API that are affected.
I tried to recreate this issue using the API with no luck.

**Answer:**
There is no API way to fix file with these problems once the document is in this state.
You may be able to use the API to delete and recreate the fault sheets...

The bug is that we have an internal tracking system that is supposed to maintain accurate lists of sheets that are present in the document. The purpose is to have a quick lookup of sheet IDs without have having to iterate the whole document. The problem occurs if API is used to create batches of multiple sheets in 1 transaction, and then the user undoes and subsequently redoes the API transaction. Our undo/redo framework had an implicit assumption that only 1 sheet could be created in a transaction, and so the tracking system loses track of the remaining API-generated sheets.

API writers should change their code to create each new sheet in its own individual transaction.
This should work around the problem with API sheet creation and undo/redo.

Here is more info on how to write an updated API function that avoids the problem:

*I tried updating the API Macro to create a batch of Sheet Placeholders and then convert the placeholders to true sheets.
If the placeholders and conversion all happen in 1 transaction (which is basically how the CreateSheets API is written), then the undo/redo bug still occurs.
However, if the placeholders use one transaction and the conversion uses a second transaction, then there is no problem.*

Here is rough VB sample API function that avoids the issue:

```vbnet
  Public Sub TwoTrans\_placeholderToReal(doc As Document)

    Dim fec As FilteredElementCollector =
      New FilteredElementCollector(doc)

    fec.OfCategory(BuiltInCategory.OST\_TitleBlocks)

    Dim NumSheets As Integer = 10
    Dim SheetsCreated(NumSheets) As ViewSheet

    'Dim SheetSubTransaction As Transaction

    Using SheetSubTransaction As New Transaction(doc)
      SheetSubTransaction.Start("Create Placeholders")
      For ii = 1 To NumSheets
        SheetsCreated(ii) = ViewSheet.CreatePlaceholder(doc)
        SheetsCreated(ii).Name = "Sheet" & ii 'NewSheetsToCreate.Name
        SheetsCreated(ii).SheetNumber = ii & "Number" 'NewSheetsToCreate.Number
      Next
      SheetSubTransaction.Commit()
    End Using

    Using ConvertPlaceholderTransaction As New Transaction(doc)
      ConvertPlaceholderTransaction.Start("Convert to Real Sheets")
      For ii = 1 To NumSheets
        SheetsCreated(ii).ConvertToRealSheet(fec.FirstElementId())
      Next
      ConvertPlaceholderTransaction.Commit()
    End Using

  End Sub
```

In summary, API writers should change their code to either:

- Create each new sheet in its own individual transaction, or
- Create a batch of placeholder sheets in one transaction and then convert the placeholders to real sheets in a second transaction.

Both approaches avoid the problem with API sheet creation and undo/redo.

Dear Revit add-in developer, please take heed!