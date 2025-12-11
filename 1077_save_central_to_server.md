---
ai_optimized: true
author: Jeremy Tammik
complexity_score: 3.0
content_type: qa
optimization_date: '2025-12-11T11:44:15.258269'
original_url: https://thebuildingcoder.typepad.com/blog/1077_save_central_to_server.html
post_number: '1077'
reading_time_minutes: 5
series: general
slug: save_central_to_server
source_file: 1077_save_central_to_server.htm
tags:
- csharp
- family
- filtering
- levels
- revit-api
- transactions
- windows
title: Saving a New Central File to Revit Server
word_count: 916
---

### Saving a New Central File to Revit Server

One topic brought up at Scott Conover's
[worksharing roundtable](http://thebuildingcoder.typepad.com/blog/2013/12/au-day-2-worksharing-and-revit-2014-api-roundtables.html#5) was
the question of how to save a new Revit central file to a Revit Server.

Participants even questioned whether this is possible at all with the current API.

Well, it is, and we look at the exact steps to achieve that below as well as:

- [DevDay in Farnborough, cabs and flights](#2)
- [Saving a new central file to Revit Server](#3)
- [Consistent naming of transactions and transaction groups](#4)
- [Changing the category of a family in project](#5)

#### DevDay in Farnborough, Cabs and Flights

We started the day in a rather bumpy British cab. Here is the resulting blurred image of Philippe, Caroline and Jim all cosy on the back seat:

![Philippe, Caroline and Jim in a cab](file:///j/photo/jeremy/2013/2013-12-11_farnborough/philippe_caroline_jim_london_cab_x_4.jpeg)

The conference was lively and exciting.
Here are Paavo and Adam happily on the way back to the airport afterwards in another British cab:

![Paavo and Adam in a cab](file:///j/photo/jeremy/2013/2013-12-11_farnborough/paavo_adam_in_cab.jpg)

Philippe and Peter may be less happy.
They attempted to board their Easyjet flight from Gatwick to Munich and were rejected because they had not checked in in advance.

Now we are a significantly reduced team sitting in the hotel bar wondering how to reorganise and handle the presentations tomorrow:

![Planning the Munich DevDay presentations](file:///j/photo/jeremy/2013/2013-12-11_farnborough/paavo_jeremy_paul_adam_jim_david_in_angelo.jpg)

#### Saving a New Central File to Revit Server

Ensure the following conditions are met to successfully save a new central file to a Revit Server ServerPath using the Document.SaveAs method:

- WorksharingSaveAsOptions.SaveAsCentral = true.
- The file is currently workshared – note that there is currently no API to make this happen.
- You have a server set for the Revit session, and you know or can obtain the correct relative path on the server for the new server document.

With that in place, the following code achieves this task:

```csharp
  public void OpenDetachedAndSaveAsNewCentral(
    UIApplication uiapp )
  {
    // Open non-interactive document with options

    Application app = uiapp.Application;

    ModelPath toOpen = GetWSAPIModelPath(
      "open\_detached.rvt" );

    Document openedDoc = OpenDetached( app, toOpen );

    ShowInfoOnOpenedWorksharedDocument( openedDoc );

    String serverPathRoot = uiapp.Application
      .GetRevitServerNetworkHosts().First();

    ModelPath modelPath = new ServerPath(
      serverPathRoot,
      "all\_new\_saved\_central\_on\_server\_preserve.rvt" );

    SaveAsOptions options = new SaveAsOptions();

    WorksharingSaveAsOptions wsOptions
      = new WorksharingSaveAsOptions();

    wsOptions.SaveAsCentral = true;
    options.SetWorksharingOptions( wsOptions );
    openedDoc.SaveAs( modelPath, options );

    ShowInfoOnOpenedWorksharedDocument( openedDoc );

    openedDoc.Close( false );
  }

  /// <summary>
  /// Helper to return a local path
  /// for a target model file.
  /// </summary>
  static ModelPath GetWSAPIModelPath(
    string fileName )
  {
    FileInfo filePath = new FileInfo( Path.Combine(
      @"C:\test\WS API Models",
      fileName ) );

    ModelPath mp = ModelPathUtils
      .ConvertUserVisiblePathToModelPath(
        filePath.FullName );

    return mp;
  }

  static Document OpenDetached(
    Application app,
    ModelPath modelPath )
  {
    OpenOptions opt = new OpenOptions();

    opt.DetachFromCentralOption
      = DetachFromCentralOption
        .DetachAndPreserveWorksets;

    Document openedDoc = app.OpenDocumentFile(
      modelPath, opt );

    return openedDoc;
  }

  /// <summary>
  /// Show popup with info about worksets
  /// and worksharing status
  /// </summary>
  static void ShowInfoOnOpenedWorksharedDocument(
    Document doc )
  {
    String documentName = doc.Title;
    bool isWorkshared = doc.IsWorkshared;

    FilteredWorksetCollector fwc
      = new FilteredWorksetCollector( doc );

    fwc.OfKind( WorksetKind.UserWorkset );

    int wsCount = fwc.Count<Workset>();

    TaskDialog td = new TaskDialog(
      "Opened document info" );

    td.MainInstruction
      = "Application has opened the document "
        + documentName;

    string mainContent = "Workshared: "
      + isWorkshared
      + ( isWorkshared
        ? "\nassociated to: "
          + ModelPathUtils
            .ConvertModelPathToUserVisiblePath(
              doc.GetWorksharingCentralModelPath() )
        : "" )
      + "\nWorkset count: " + wsCount + "\n"
      + String.Join( "\n",
        fwc.Select<Workset, String>(
          ws => ws.Name + " - "
            + ( ws.IsOpen ? "open" : "closed" ) ) );

    td.MainContent = mainContent;

    td.Show();
  }
```

#### Consistent Naming of Transactions and Transaction Groups

Somebody pointed out an interesting little issue concerning the naming of transactions and transaction groups which might possibly cause some confusion for an add-in user and can easily be worked around:

Let's assume that your add-in has defined a transaction group named A. Within it, it committs one single transaction named B. In that case, the undo stack displayed to the user shows B instead of A.

The suggested workaround to handle this cleanly is simple: keep a count of commited transactions within the group. As long as the count is still zero, name the first transaction A as well, just like the top level group.

One aspect of this situation is described in the Revit API help file RevitAPI.chm: "There are two ways of committing a group – Commit and Assimilate. By committing, all transactions committed inside a group stay as they are, while by assimilating, all inner transactions will be merged into a single transaction."

#### Changing the Category of a Family in Project

**Question:** How can I change a curtain panel family category from Curtain Panel to Window?

I know I can do it through the user interface, but how do I achieve the same thing programmatically?

I can imagine something along the following steps: iterate the FamilySymbols, get the Family, edit it as a Document – which I can do – change the category and add it back to the RVT file.

Can this be done without editing it as a Document?

What is the proper workflow for this task, please?

**Answer:** You can only change the category for the family associated with a family document, i.e. the OwnerFamily of the family Document. This means that you have to be editing the family when this happens, i.e. open and access the family document via the
[EditFamily](http://thebuildingcoder.typepad.com/blog/2012/05/edit-family-requires-no-transaction.html) method.
There are probably some other restrictions as well, which match what the UI allows you to do.