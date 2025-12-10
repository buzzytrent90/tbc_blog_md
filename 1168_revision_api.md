---
post_number: "1168"
title: "The Revision API and a Form on the Fly"
slug: "revision_api"
author: "Jeremy Tammik"
tags: ['csharp', 'elements', 'family', 'parameters', 'revit-api', 'sheets', 'transactions', 'views', 'windows']
source_file: "1168_revision_api.htm"
original_url: "https://thebuildingcoder.typepad.com/blog/1168_revision_api.html"
---

### The Revision API and a Form on the Fly

Poetical, ain't it?

One of the
[major Revit 2015 API additions](http://thebuildingcoder.typepad.com/blog/2014/04/whats-new-in-the-revit-2015-api.html#3) is
access to
[revisions](http://thebuildingcoder.typepad.com/blog/2014/04/whats-new-in-the-revit-2015-api.html#3.04).

All prior versions provided very limited access to revision data in a project.
Here are some things people achieved in spite of the limitations:

- [Max' revision wrapper class](http://thebuildingcoder.typepad.com/blog/2013/09/max-revision-wrapper-class.html)
- [Wrangling revisions with Ruby](http://thebuildingcoder.typepad.com/blog/2014/02/wrangling-revisions-with-ruby.html)

Let's now take a look at an elegant example of accessing and displaying the complete revision data in a project,
[GetRevisionData](https://github.com/jeremytammik/GetRevisionData),
prompted by the following query by Dan Tartaglia of
[design technology@NBBJ](http://www.nbbj.com):

**Question:** Selecting the View tab in Revit and then Revisions in the Sheet Composition pane displays the 'Sheet Issues/Revisions' dialogue:

![Sheet Issues Revisions dialogue](img/sheet_issues_revisions.png)

I am trying to access the information displayed programmatically, in particular the information for these revisions found in the 'Show' column with the three possible choices 'None', 'Tag' and 'Cloud and Tag'.

Is that possible?

I am currently using the GetAllProjectRevisionIds method, but that does not return all the required information.

**Answer:** What you ask for is now possible in Revit 2015 using the new
[revision classes](http://thebuildingcoder.typepad.com/blog/2014/04/whats-new-in-the-revit-2015-api.html#3.04).

I am not aware of any way to access it programmatically in Revit 2014, though.
Developers have been asking for it for a long time, and that access was one of the major Revit 2015 API enhancements.

Examining your
[Revit 2014 GetRevisionData attempt](https://github.com/jeremytammik/GetRevisionData/releases/tag/2014.0.0.0) in
a little more detail, I have some comments on that before getting to the Revit 2015 solution.

It includes a nice utility method GetParameterInformation to convert a parameter value to a string representation.
That is used by the ParamsFromGetAllRevElements method to retrieve all project revisions via their ids and list all their parameter values.

However, as said, it does not return the information we are after.

I fixed the
[architecture mismatch warnings](http://thebuildingcoder.typepad.com/blog/2013/06/processor-architecture-mismatch-warning.html) in
the initial 2014 project using my recursive project parser and fixer
[DisableMismatchWarning.exe](http://thebuildingcoder.typepad.com/blog/2013/07/recursively-disable-architecture-mismatch-warning.html).

I like the parameter to value to string converter, so let's list it here for posterity to enjoy:

```csharp
  /// <summary>
  /// Extract the parameter information.
  /// By Dan Tartaglia.
  /// </summary>
  public string GetParameterInformation(
    Parameter para,
    Document doc )
  {
    string defName = "";

    // Use different method to get parameter
    // data according to the storage type

    switch( para.StorageType )
    {
      // Determine the parameter type

      case StorageType.Double:

        // Convert the number into Metric

        defName = para.AsValueString();
        break;

      case StorageType.ElementId:

        // Find out the name of the element

        Autodesk.Revit.DB.ElementId id
          = para.AsElementId();

        defName = ( id.IntegerValue >= 0 )
          ? doc.GetElement( id ).Name
          : id.IntegerValue.ToString();

        break;

      case StorageType.Integer:
        if( ParameterType.YesNo
          == para.Definition.ParameterType )
        {
          if( para.AsInteger() == 0 )
          {
            defName = "False";
          }
          else
          {
            defName = "True";
          }
        }
        else
        {
          defName = para.AsInteger().ToString();
        }
        break;

      case StorageType.String:
        defName = para.AsString();
        break;

      default:
        defName = "Unexposed parameter";
        break;
    }
    return defName;
  }
```

One little suggestion I have is to encapsulate the text writer instance in a using block, e.g. like this:

```csharp
using( TextWriter tw = new StreamWriter(
"C:/tmp/RevisionTest.txt" ) )
{
tw.WriteLine( ". . ." );
tw.Close();
}
```

For comparison with the Revit 2015 results, let's list the limited data accessible via the Revit 2014 API here, printed out to a text file `C:/tmp/RevisionTest.txt`:

```csharp
C:\tmp>cat RevisionTest.txt
Hidden = False
Element Name: Revisions
oParamRevEnum = 0
oParamRevDate = Date 1
oParamRevDescrip = Revision 1
oParamRevIssued = False
oParamRevIssuedBy =
oParamRevIssuedTo =
oParamRevNumber = 1
oParamSeqNumber = 1
==============================
Hidden = False
Element Name: Revisions
oParamRevEnum = 0
oParamRevDate = Date 2
oParamRevDescrip = Revision 2
oParamRevIssued = False
oParamRevIssuedBy =
oParamRevIssuedTo =
oParamRevNumber = 2
oParamSeqNumber = 2
==============================
Hidden = False
Element Name: Revisions
oParamRevEnum = 0
oParamRevDate = Date 5
oParamRevDescrip = Revision 5
oParamRevIssued = False
oParamRevIssuedBy =
oParamRevIssuedTo =
oParamRevNumber = 5
oParamSeqNumber = 5
==============================
```

I published the Revit 2014 version in the
[GetRevisionData GitHub repository](https://github.com/jeremytammik/GetRevisionData) as
[version 2014.0.0.0](https://github.com/jeremytammik/GetRevisionData/releases/tag/2014.0.0.0).

**Response:** Yes, I verified that I can get what I need with the Revit 2015 API like this:

```csharp
  IList<ElementId> oElemIDs = oViewSheet.GetAllRevisionIds();

  if( oElemIDs.Count == 0 )
    return;

  foreach( ElementId elemID in oElemIDs )
  {
    Element oEl = doc.GetElement( elemID );

    Revision oRev = oEl as Revision;

    // Add text line to text file
    tw.WriteLine( "Rev Category Name: " + oRev.Category.Name );

    // Add text line to text file
    tw.WriteLine( "Rev Description: " + oRev.Description );

    // Add text line to text file
    tw.WriteLine( "Rev Issued: " + oRev.Issued.ToString() );

    // Add text line to text file
    tw.WriteLine( "Rev Issued By: " + oRev.IssuedBy.ToString() );

    // Add text line to text file
    tw.WriteLine( "Rev Issued To: " + oRev.IssuedTo.ToString() );

    // Add text line to text file
    tw.WriteLine( "Rev Number Type: " + oRev.NumberType.ToString() );

    // Add text line to text file
    tw.WriteLine( "Rev Date: " + oRev.RevisionDate );

    // Add text line to text file
    tw.WriteLine( "Rev Visibility: " + oRev.Visibility.ToString() );

    // Add text line to text file
    tw.WriteLine( "Rev Sequence Number: " + oRev.SequenceNumber.ToString() );

    // Add text line to text file
    tw.WriteLine( "Rev Number: " + oRev.RevisionNumber );
  }
```

**Answer:** Congratulations on solving it!

I am glad that the Revit 2015 API provides all you need.

I implemented a sample command grabbing all the revision data displayed in the Revit 'Sheet Issues/Revisions' form and displaying that in a Windows form generated on the fly.

It avoids the text writer and file output completely by implementing a revision data holder class and a container dictionary, using a Windows forms data grid view container and its DataSource property to access and display the data:

![Revision data](img/revision_data.png)

It demonstrates several other nice features, in addition to the revision functionality:

- Accessing and displaying all the revision information without ever actually touching or formatting any individual data members.
- Generating a Windows form programmatically on the fly.
- Populating a DataGridView via its DataSource property.

The Revit 2015 version is published in the
[GetRevisionData GitHub repository](https://github.com/jeremytammik/GetRevisionData) as
[version 2015.0.0.0](https://github.com/jeremytammik/GetRevisionData/releases/tag/2015.0.0.0).

The hardest challenge was actually not implementing the DataGridView population, but the automatic resizing.
That took quite a while, testing and debugging numerous combinations of the automatic resizing properties until finally finding one that worked.

The complete external command implementation defines just three simple little items:

- RevisionData – a container for the revision data displayed in the Revit 'Sheet Issues/Revisions' dialogue.
- DisplayRevisionData – generate a Windows modeless form on the fly and display the revision data in it in a DataGridView.
- Execute – external command mainline.

Each one on its own is pretty trivial.

Together, they form a rally neat and elegant solution, I think:

```csharp
#region Namespaces
using System;
using System.Collections.Generic;
using System.Data;
using System.Diagnostics;
using System.Drawing;
using System.IO;
using System.Windows.Forms;
using Autodesk.Revit.ApplicationServices;
using Autodesk.Revit.Attributes;
using Autodesk.Revit.DB;
using Autodesk.Revit.UI;
using Autodesk.Windows;
using TaskDialog = Autodesk.Revit.UI.TaskDialog;
#endregion

namespace GetRevisionData
{
  /// <summary>
  /// External command demonstrating how to use the
  /// Revit 2015 Revision API to retrieve and display
  /// all infoormation shown in the Revit
  /// 'Sheet Issues/Revisions' dialogue.
  /// </summary>
  [Transaction( TransactionMode.ReadOnly )]
  public class Command : IExternalCommand
  {
    /// <summary>
    /// A container for the revision data displayed in
    /// the Revit 'Sheet Issues/Revisions' dialogue.
    /// </summary>
    class RevisionData
    {
      public int Sequence { get; set; }
      public RevisionNumberType Numbering { get; set; }
      public string Date { get; set; }
      public string Description { get; set; }
      public bool Issued { get; set; }
      public string IssuedTo { get; set; }
      public string IssuedBy { get; set; }
      public RevisionVisibility Show { get; set; }

      public RevisionData( Revision r )
      {
        Sequence = r.SequenceNumber;
        Numbering = r.NumberType;
        Date = r.RevisionDate;
        Description = r.Description;
        Issued = r.Issued;
        IssuedTo = r.IssuedTo;
        IssuedBy = r.IssuedBy;
        Show = r.Visibility;
      }
    }

    /// <summary>
    /// Generate a Windows modeless form on the fly
    /// and display the revision data in it in a
    /// DataGridView.
    /// </summary>
    void DisplayRevisionData(
      List<RevisionData> revision\_data,
      IWin32Window owner )
    {
      System.Windows.Forms.Form form
        = new System.Windows.Forms.Form();

      form.Size = new Size( 680, 180 );
      form.Text = "Revision Data";

      DataGridView dg = new DataGridView();
      dg.DataSource = revision\_data;
      dg.AllowUserToAddRows = false;
      dg.AllowUserToDeleteRows = false;
      dg.AllowUserToOrderColumns = true;
      dg.Dock = System.Windows.Forms.DockStyle.Fill;
      dg.Location = new System.Drawing.Point( 0, 0 );
      dg.ReadOnly = true;
      dg.TabIndex = 0;
      dg.Parent = form;
      dg.AutoSize = true;
      dg.AutoSizeColumnsMode = DataGridViewAutoSizeColumnsMode.AllCells;
      dg.AutoResizeColumns( DataGridViewAutoSizeColumnsMode.AllCells );
      dg.AutoResizeColumns();

      form.ShowDialog( owner );
    }

    public Result Execute(
      ExternalCommandData commandData,
      ref string message,
      ElementSet elements )
    {
      IWin32Window revit\_window
        = new JtWindowHandle(
          ComponentManager.ApplicationWindow );

      UIApplication uiapp = commandData.Application;
      UIDocument uidoc = uiapp.ActiveUIDocument;
      Document doc = uidoc.Document;

      if( doc.IsFamilyDocument )
      {
        TaskDialog.Show( "Not a Revit RVT Project",
          "This command requires an active Revit RVT file." );

        return Result.Failed;
      }

      if( !( doc.ActiveView is ViewSheet ) )
      {
        TaskDialog.Show( "Current View is not a Sheet",
          "This command requires an active sheet view." );
        return Result.Failed;
      }

      IList<ElementId> ids
        = Revision.GetAllRevisionIds( doc );

      int n = ids.Count;

      List<RevisionData> revision\_data
        = new List<RevisionData>( n );

      foreach( ElementId id in ids )
      {
        Revision r = doc.GetElement( id ) as Revision;

        revision\_data.Add( new RevisionData( r ) );
      }

      DisplayRevisionData( revision\_data,
        revit\_window );

      return Result.Succeeded;
    }
  }
}
```

Enjoy!

Next time I write will be from Toronto,
  إن شاء الله
  ([insha'Allah](http://en.wikipedia.org/wiki/Insha%27Allah), God willing).