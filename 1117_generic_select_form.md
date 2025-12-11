---
ai_optimized: true
author: Jeremy Tammik
complexity_score: 4.0
content_type: qa
optimization_date: '2025-12-11T11:44:15.336292'
original_url: https://thebuildingcoder.typepad.com/blog/1117_generic_select_form.html
post_number: '1117'
reading_time_minutes: 6
series: general
slug: generic_select_form
source_file: 1117_generic_select_form.htm
tags:
- csharp
- elements
- filtering
- revit-api
- rooms
- selection
- transactions
- views
- windows
title: Using Generic Collections with Filters and Forms
word_count: 1161
---

### Using Generic Collections with Filters and Forms

Today I discuss some uses of generics to elegantly and efficiently handle lists of Revit elements and .NET Windows forms, specifically:

- One line of code to retrieve all printable floor plan views from the BIM.
- One line of code to retrieve all selected floor plan views from a Windows check box list.

This is my first step starting to implement the new RoomEditorApp features for my
[generic 2D simplified BIM editor](http://thebuildingcoder.typepad.com/blog/2014/03/back-from-desert-and-two-happy-events.html#3) Tech
Summit presentation.

The current tentative planned workflow steps to upload the database information look like this:

1. Launch the RoomEditorApp export plan views command.
2. Display a list of plan views in a popup window.
3. Select views to be exported and click OK.
4. Display a list of categories in a popup window.
5. Select categories to be exported and click OK.
6. Store relevant graphical and non-graphical information in a cloud database.

The following editing and downloading steps will remain similar to the original implementation:

7. Navigate and display simplified model on mobile device.
8. Edit graphical and non-graphical information.
9. Update Revit BIM either interactively or real-time.

I will be looking at the first three steps today.

Yesterday, I mentioned a nice use of the generic ToDictionary method to
[convert a filtered element collector to a dictionary](http://thebuildingcoder.typepad.com/blog/2014/03/adding-new-materials-from-list-updated.html)

Today I would like to present related examples, e.g. to populate a CheckListBox in a Windows form and extract the resulting user selection back to a generic list in a single call.

I implemented a new RoomEditorApp command named [CmdUploadViews](#3).

It makes use of a form to select the plan views to export named [FrmSelectViews](#2).

Let's look at the latter first.

#### FrmSelectViews

This .NET Windows form supports interactive user selection of plan views.

Here is a snapshot of the floor plans in the project browser in a very simple Revit model:
![Floor plans in project browser](img/select_views_browser.png)

Using RevitLookup to examine the ViewPlan instances stored in the Revit database, we note that there are many more than the ones of interest:

![ViewPlan instances in RevitLookup](img/select_views_snoop.png)

Some of these are ceiling plans, not floor plans.
Others are unknown objects that we cannot even explain.

We can filter out the ones we actually want by checking their ViewType and CanBePrinted properties.

This is done in a LINQ Where statement added to the filtered element collector filtering.

Please note that the filtered element collector itself is enumerable, and we can make good use of that to combine it with other generic filters to post-process the results returned by Revit.

In the discussion on
[FindElement and collector optimisation](http://thebuildingcoder.typepad.com/blog/2012/09/findelement-and-collector-optimisation.html),
I already mentioned that it is often not necessary and even wasteful to convert a filtered element collector to a list of elements or element ids.

The following code performs all that filtering and generates a generic list of views that can be used to directly populate a Windows .NET CheckedListBox via its DataSource and DisplayMember properties:

```csharp
  List<ViewPlan> views = new List<ViewPlan>(
    new FilteredElementCollector( \_doc )
      .OfClass( typeof( ViewPlan ) )
      .Cast<ViewPlan>()
      .Where<ViewPlan>( v => v.CanBePrinted
        && ViewType.FloorPlan == v.ViewType ) );
```

Similarly, the floor plans selected by the user in the form by checking the associated boxes can be retrieved in one single line from the checked list box CheckedItems property by adding LINQ statements to cast the items and convert them to a generic List of ViewPlan instances.

To cut a long story short – or rather, to keep the short story as short as it is, without adding yet more unnecessary verbosity – here is the source code of this form – from the horse's mouth, so to say:

```csharp
using System;
using System.Collections.Generic;
using System.Linq;
using System.Windows.Forms;
using Autodesk.Revit.DB;
using Form = System.Windows.Forms.Form;
/// <summary>
/// Interactive user selection of plan views.
/// </summary>
public partial class FrmSelectViews : Form
{
  /// <summary>
  /// The Revit input project document.
  /// </summary>
  Document \_doc;

  /// <summary>
  /// Constructor initialises document
  /// and nothing else.
  /// </summary>
  /// <param name="doc"></param>
  public FrmSelectViews( Document doc )
  {
    InitializeComponent();

    \_doc = doc;
  }

  /// <summary>
  /// Candidate views are retrieved by a filtered
  /// element collector on loading the form.
  /// </summary>
  private void FrmSelectViews\_Load(
    object sender,
    EventArgs e )
  {
    List<ViewPlan> views = new List<ViewPlan>(
      new FilteredElementCollector( \_doc )
        .OfClass( typeof( ViewPlan ) )
        .Cast<ViewPlan>()
        .Where<ViewPlan>( v => v.CanBePrinted
          && ViewType.FloorPlan == v.ViewType ) );

    checkedListBox1.DataSource = views;
    checkedListBox1.DisplayMember = "Name";
  }

  /// <summary>
  /// Selected views are accessible after the
  /// form has been successfully completed.
  /// </summary>
  /// <returns></returns>
  public List<ViewPlan> GetSelectedViews()
  {
    return checkedListBox1.CheckedItems
      .Cast<ViewPlan>().ToList<ViewPlan>();
  }
}
```

#### CmdUploadViews

The command is still just a skeleton, doing nothing at all yet except to exercise and test the FrmSelectViews view selection.

Two noteworthy details:

- Determine the Revit main application window handle and use that to parent the forms we display, so that they behave correctly on minimising and restoring Revit etc.
- Once again use generic collection functionality to avoid touching the individual selected view element names, using the string Join method to create a list to display in a message box.

Here is the entire command implementation:

```csharp
using System;
using System.Linq;
using System.Collections.Generic;
using Autodesk.Revit.ApplicationServices;
using Autodesk.Revit.Attributes;
using Autodesk.Revit.DB;
using Autodesk.Revit.UI;
using ComponentManager = Autodesk.Windows.ComponentManager;
using IWin32Window = System.Windows.Forms.IWin32Window;
using DialogResult = System.Windows.Forms.DialogResult;
[Transaction( TransactionMode.ReadOnly )]
public class CmdUploadViews : IExternalCommand
{
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
    Application app = uiapp.Application;
    Document doc = uidoc.Document;

    if( null == doc )
    {
      Util.ErrorMsg( "Please run this command in a valid"
        + " Revit project document." );
      return Result.Failed;
    }

    FrmSelectViews form = new FrmSelectViews( doc );

    if( DialogResult.OK == form.ShowDialog(
      revit\_window ) )
    {
      List<ViewPlan> views = form.GetSelectedViews();

      int n = views.Count;

      string caption = string.Format(
        "{0} Plan View{1} Selected",
        n, Util.PluralSuffix( n ) );

      string list = string.Join( ", ",
        views.Select<Element,string>(
          e => e.Name ) );

      TaskDialog.Show( caption, list );

      //List<Category> categories = new List<Category>();
    }
    return Result.Succeeded;
  }
}
```

The view selection form is displayed like this:

![View selection form](img/select_views_form.png)

The resulting user selection is reported in a standard Revit task dialogue:

![View selection report](img/select_views_report.png)

#### Download

For the complete source code, Visual Studio solution and add-in manifest, please refer to the
[RoomEditorApp GitHub repository](https://github.com/jeremytammik/RoomEditorApp).

The version discussed above is stored as
[release 2014.0.2.1](https://github.com/jeremytammik/RoomEditorApp/releases/tag/2014.0.2.1).

I hope this helps you make use of generics and compiling them with Revit API objects as well, and avoid implementing code handling individual collection member unnecessarily.

Wish me luck for the next steps.