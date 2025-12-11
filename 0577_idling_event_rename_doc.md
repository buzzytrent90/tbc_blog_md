---
ai_optimized: true
author: Jeremy Tammik
complexity_score: 4.5
content_type: code_example
optimization_date: '2025-12-11T11:44:14.203521'
original_url: https://thebuildingcoder.typepad.com/blog/0577_idling_event_rename_doc.html
post_number: '0577'
reading_time_minutes: 7
series: general
slug: idling_event_rename_doc
source_file: 0577_idling_event_rename_doc.htm
tags:
- csharp
- doors
- levels
- python
- revit-api
- transactions
title: Cascaded Events and Renaming the Active Document
word_count: 1433
---

### Cascaded Events and Renaming the Active Document

I waxed philosophical on the topic of
[change](http://thebuildingcoder.typepad.com/blog/2011/05/iteration-and-springtime-change-is-the-only-constant.html) yesterday,
and here is another issue that can be related to that.
In fact, almost anything can be associated with change somehow or other, can't it?

A developer trying to trap the SaveAs command and rename the active document in order to force users to comply with certain naming conventions was attempting a convoluted solution and running into problems with that scenario.

Scott Conover, Software Development Manager in the Revit API team, came up with an elegant and effective alternative solution making use of "cascaded events", i.e. an event handler which in turn temporarily subscribes to another event.

In this case, we obviously make use of the SavingAs and SavedAs pre- and post-event notifications.
They cannot be used to rename a document.
The pre-event can however cancel the event if desired, and subscribe temporarily to the Idling event, in which you have full freedom to make any changes you like.

We already discussed a number of aspects of the Idling event, which was one of the most exciting new features provided by the Revit 2011 API:

- The [Idling event](http://thebuildingcoder.typepad.com/blog/2010/04/idling-event.html).- [Asynchronous API Calls and Idling](http://thebuildingcoder.typepad.com/blog/2010/04/asynchronous-api-calls-and-idling.html).- [Modeless loose connectors](http://thebuildingcoder.typepad.com/blog/2010/07/modeless-loose-connectors.html) Idling sample.- A [pattern for Idling access](http://thebuildingcoder.typepad.com/blog/2010/11/pattern-for-semi-asynchronous-idling-api-access.html).- Another [modeless sample using Idling](http://thebuildingcoder.typepad.com/blog/2010/12/modeless-door-lister-and-deleter.html) and a [critique of its flaws](http://thebuildingcoder.typepad.com/blog/2011/01/modeless-door-lister-flaws.html).

The reason the convoluted workaround failed and this succeeds is the fact that save can be invoked while an editor transaction is open.
That is one of the useful features of the Idling event, to be told by Revit when it is OK to proceed because the session is not in an unsupported state.

Before we look at the steps involved, and because it is short and sweet, here is the entire source code, to make it easier to understand the description.
Scott's original implementation was based on the Revit 2011 API, since that is what the developer was asking about.
Here is a Revit 2012 version, with a few small interesting differences which I will discuss further down:
```python
class App : IExternalApplication
{
  public Result OnShutdown( UIControlledApplication a )
  {
    return Result.Succeeded;
  }

  public Result OnStartup( UIControlledApplication a )
  {
    a.ControlledApplication.DocumentSavingAs
      += new EventHandler<DocumentSavingAsEventArgs>(
        OnDocumentSavingAs );

    a.ControlledApplication.DocumentSavedAs
      += new EventHandler<DocumentSavedAsEventArgs>(
        OnDocumentSavedAs );

    return Result.Succeeded;
  }

  bool reentering = false;

  void OnDocumentSavingAs(
    object sender,
    DocumentSavingAsEventArgs e )
  {
    try
    {
      if( !reentering )
      {
        Document doc = e.Document;

        e.Cancel();

        reentering = true;

        UIApplication uiApp
          = new UIApplication( doc.Application );

        uiApp.Idling
          += new EventHandler<IdlingEventArgs>(
            OnIdling );
      }
    }
    catch( System.Exception ex )
    {
      TaskDialog.Show( "Exception in SavingAs",
        ex.ToString() );
    }
  }

  void OnDocumentSavedAs(
    object sender,
    DocumentSavedAsEventArgs e )
  {
    try
    {
      if( e.Status != RevitAPIEventStatus.Cancelled )
      {
        reentering = false;
      }
    }
    catch( System.Exception ex )
    {
      TaskDialog.Show( "Exception in SavedAs",
        ex.ToString() );
    }
  }

  void OnIdling( object sender, IdlingEventArgs e )
  {
    //Application app = (Application) sender; // 2011
    //UIApplication uiApp = new UIApplication( app ); // 2011

    UIApplication uiApp = sender as UIApplication; // 2012

    Document doc = uiApp.ActiveUIDocument.Document;

    string filename = @"C:\foo.rvt";

    // warning CS0618:
    // Autodesk.Revit.DB.Document.SaveAs(string, bool) is obsolete:
    // This method is obsolete, use SaveAs(String, SaveAsOptions) instead.
    //doc.SaveAs( filename, true );

    // SaveAsOptions.Rename essentially does what the old option did
    // – it either kept the document in memory with the original name,
    // or renamed it in memory.  If the latter, the UI is one place
    // where the new name appears.

    SaveAsOptions options = new SaveAsOptions();
    options.OverwriteExistingFile = true;
    options.Rename = true;

    doc.SaveAs( filename, options );

    uiApp.Idling -= OnIdling;
  }
}
```

Here is a top-level description of the functionality and execution sequence of the code:

- We implement an external application subscribing to the SavingAs and SavedAs pre- and post-event notification.- Within the pre-event notification, we cancel the save to the user selected file name, set a flag named 'reentering', and register to the Idling event.- The post-event is used to reset the 'reentering' flag, but only if the save operation has not been cancelled.- The document cannot be renamed in either of these events, nor can an additional SaveAs call be made, which is why we need to register to the Idling event instead.- When the Idling event handler is notified, Revit has terminated the cancelled attempt to save the file and is in quiescent state.- Now SaveAs can be called with a valid filename that cannot be modified by the user.- We can also immediately unsubscribe from the Idling event right there in the Idling event handler.- The two pre- and post-event handlers will both be called again.- Because 'reentering' is still true, the pre-event knows that this save event should not be cancelled and it has nothing to do.- The post-event handler determines that in this call the operation is not being cancelled and resets the 'reentering' flag.

I made the following additional notes while running the application and stepping through these steps one by one in the debugger:

- Select Big R > SaveAs > file naming dialogue > specify a new name > OK.- OnDocumentSavingAs is called, cancels the save operation, sets the 'reentering' flag, and subscribes to the Idling event.- OnDocumentSavedAs is called; the status is already cancelled, so it has nothing to do.- A 'file not saved' dialogue is displayed:

        ![File not saved](img/file_not_saved.png)

        ... that needs to be caught and dismissed, I guess.- On clicking OK, the save operation is terminated and the Idling event fires, saving the document to a new name.- OnDocumentSavingAs is called again; this time, 'reentering' is true, so the operation is not cancelled and the save as operation completes.- OnDocumentSavedAs is called; the status is not cancelled this time, and the reentering flag is reset to false.- The document is renamed as desired.

Here are two interesting little differences between the code for Revit 2011 and Revit 2012:

- In Revit 2011, the sender argument to the Idling event handler was an Application instance and we had to instantiate our own UIApplication instance from it.- In Revit 2011, we used the SaveAs overload taking a Boolean argument 'changeDocumentFilename' to change the document title in Revit as well as specify the name of the external file on the hard disk; in 2012, we use the SaveAsOptions.Rename property to achieve the same thing.

Both SaveAsOptions.Rename and the obsolete 'changeDocumentFilename' argument choose between either keeping the document in memory with the original name, or renaming it in memory.

For completeness' sake, here is the original Revit 2011 version
[RenDocOnSave2011.zip](zip/RenDocOnSave2011.zip)
and the cleaned up Revit 2012 migration
[RenDocOnSave2012.zip](zip/RenDocOnSave2012.zip).

#### Idling Event Handler Sender Argument Changed

The second change above actually affects the migration of every single Revit 2011 add-in subscribing to the Idling event and using the event handler 'sender' argument to access the Revit application, for example to retrieve the active document from it.

The Building Coder sample command
[CmdIdling](http://thebuildingcoder.typepad.com/blog/2010/04/idling-event.html) used the following code in the Revit 2011 API:
```csharp
void OnIdling( object sender, IdlingEventArgs e )
{
  // access active document from sender:

  Application app = sender as Application;

  Debug.Assert( null != app,
    "expected a valid Revit application instance" );

  if( app != null )
  {
    UIApplication uiapp = new UIApplication( app );
    UIDocument uidoc = uiapp.ActiveUIDocument;
    Document doc = uidoc.Document;

    Log( "OnIdling with active document "
      + doc.Title );
  }
}
```

This will fail in Revit 2012, since the sender argument is no longer an Application instance, but an UIApplication one.
The updated code is much simpler and looks like this:
```csharp
void OnIdling( object sender, IdlingEventArgs e )
{
  // access active document from sender:

  UIApplication uiapp = sender as UIApplication;
  Document doc = uiapp.ActiveUIDocument.Document;

  Log( "OnIdling with active document "
    + doc.Title );
}
```

Here is an updated
[version 2012.0.87.2](zip/bc_12_87_2.zip) of
The Building Coder sample code including this change.

**Disclaimer:** Please note that the code presented here is just a test showing one possible workaround for the originally stated problem with the customized SaveAs.
It is by no means a production-level solution, and even the concept of using the Idling event in the suggested way is not optimal, as it could lead to potentially undesired results.
A more detailed discussion and analysis of the above presented workaround will be published in a future post.