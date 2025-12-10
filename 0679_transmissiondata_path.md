---
post_number: "0679"
title: "Access Central File TransmissionData on Revit Server"
slug: "transmissiondata_path"
author: "Jeremy Tammik"
tags: ['elements', 'parameters', 'python', 'revit-api', 'transactions']
source_file: "0679_transmissiondata_path.htm"
original_url: "https://thebuildingcoder.typepad.com/blog/0679_transmissiondata_path.html"
---

### Access Central File TransmissionData on Revit Server

We already discussed using the TransmissionData class to
[list linked files](http://thebuildingcoder.typepad.com/blog/2011/05/list-linked-files-and-transmissiondata.html) and
[make changes using the WriteTransmissionData method](http://thebuildingcoder.typepad.com/blog/2011/10/using-the-writetransmissiondata-method.html).

Here is another question on this topic answered by my colleague Joe Ye:

**Question:** I want to access the TransmissionData object in a central file stored in the Revit Server. How can I achieve that via the Revit API, please?

I tried the suggestions listed in the developer guide and none of them worked for me.

I also tried using IsValidUserVisibleFullServerPath on various permutations of the server name, path and model name as reported in my journal file, but it returned false on all my attempts.

**Answer:** To read the TransmissionData object, you need to call the static method TransmissionData.ReadTransmissionData.
It requires a ModelPath object.
We will show you how to construct the ModelPath object that refers to the central file.

There are two ways to construct a ModelPath object.

#### Way #1

- Compose the user-visible path string of the central file, e.g. using the string returned by ModelPathUtils.GetRevitServerPrefix plus the relative path.
  Note: The folder separator used in the relative path is a forward slash '/'.
  The Revit 2012 API help documentation RevitAPI.chm mistakenly uses a backslash to separate folders.
  The correct separator is a forward slash.- Create a ModelPath object via the ModelPathUtils.ConvertUserVisiblePathToModelPath method.
    Pass in the string composed in the previous step.- Read the transmission data via the TransmissionData.ReadTransmissionData method.
      Pass in the ModelPath obtained in the previous step.

Let's look at accessing the following central file on a Revit Server:

![Revit Server central file](img/revit_server_central_file.png)

Here is the code showing the implementation of way #1:
```python
[TransactionAttribute( TransactionMode.Manual )]
public class RevitCommand : IExternalCommand
{
  public Result Execute(
    ExternalCommandData commandData,
    ref string messages,
    ElementSet elements )
  {
    UIApplication app = commandData.Application;
    Document doc = app.ActiveUIDocument.Document;

    Transaction trans = new Transaction( doc );
    trans.Start( "ExComm" );

    string visiblePath = ModelPathUtils
      .GetRevitServerPrefix() + "/testmodel.rvt";

    ModelPath serverPath = ModelPathUtils
      .ConvertUserVisiblePathToModelPath(
        visiblePath );

    TransmissionData transData = TransmissionData
      .ReadTransmissionData( serverPath );

    if( transData != null )
    {
      // Access the data in the transData here.
    }
    else
    {
      TaskDialog.Show( "Transmission Data",
        "The document does not have any transmission data" );
    }

    trans.Commit();

    return Result.Succeeded;
  }
}
```

#### Way #2

Use this if your program knows the local server name.
This is mostly not recommended, since the server name may be changed manually from the user interface by the Revit user.

- Create a ServerPath object using ServerPath constructor taking the server name IP and the relative path without the initial forward slash.

  Please note that the first parameter is the server name, not the string returned by the ModelPathUtils.GetRevitServerPrefix.

  The second parameter does not include the initial forward slash. See the following sample code.
  The folder separator is a forward slash '/' here as well.- Read the TransmissionData object via the TransmissionData.ReadTransmissionData method, passing in the ServerPath obtained in the previous step.

Here is the code showing the implementation of way#2.
```python
[TransactionAttribute( TransactionMode.Manual )]
public class RevitCommand : IExternalCommand
{
  public Result Execute(
    ExternalCommandData commandData,
    ref string messages,
    ElementSet elements )
  {
    UIApplication app = commandData.Application;
    Document doc = app.ActiveUIDocument.Document;

    Transaction trans = new Transaction( doc );
    trans.Start( "ExComm" );

    ServerPath serverPath = new ServerPath(
      "SHACNG035WQRP", "testmodel.rvt" );

    TransmissionData transData = TransmissionData
      .ReadTransmissionData( serverPath );

    if( transData != null )
    {
      // Access the data in the transData here.
    }
    else
    {
      TaskDialog.Show( "Transmission Data",
        "The document does not have any transmission data" );
    }
    trans.Commit();

    return Result.Succeeded;
  }
}
```