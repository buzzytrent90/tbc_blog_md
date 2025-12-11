---
ai_optimized: true
author: Jeremy Tammik
complexity_score: 5.0
content_type: qa
optimization_date: '2025-12-11T11:44:13.449516'
original_url: https://thebuildingcoder.typepad.com/blog/0151_auto_confirm_save.html
post_number: '0151'
reading_time_minutes: 5
series: general
slug: auto_confirm_save
source_file: 0151_auto_confirm_save.htm
tags:
- csharp
- family
- parameters
- revit-api
- walls
- windows
title: Auto-Confirm Save Using DialogBoxShowing Event
word_count: 958
---

### Auto-Confirm Save Using DialogBoxShowing Event

Simon Del Faveo of
[RevitTV Limited](http://www.revittv.com)
asked an interesting question which I decided to implement a sample application for, since it may be of general interest and even general use.
It demonstrates a specific use of the
[new events](http://thebuildingcoder.typepad.com/blog/2009/04/new-2010-events.html)
provided by the Revit 2010 API.

**Question:**
We are trying to save loaded family files after some parameters have been added.
When using the Save or Close functions, Revit always prompts you with the confirm message box asking 'Do you want to save changes to filename.rfa?' and offering Yes, No, and Cancel options.
We are processing a whole series of RFA files, so we don't want that message box appearing hundreds of times.
Is there any way via the API to prevent the message box from appearing?

**Answer:**
You can request notification on the display of certain dialogue boxes by registering to the Application.DialogBoxShowing event. Its event handler takes two arguments, the sender and a DialogBoxShowingEventArgs instance. The sender argument is actually the Revit application object. The event args parameter provides a HelpId property to enable you to identify the dialogue, and an OverrideResult method to define how the dialogue should be handled.

Note that not all dialogues in Revit raise a DialogBoxShowing event. Only dialogs derived from a specific internal class do so, unless a specific derived class overrides that default behaviour. I do not have a list of exceptions, but the at least the file open dialogs are derived from a different class, so these events are not raised for the SaveAs dialog. Luckily, this does not affect your case.

The event argument DialogBoxShowingEventArgs class is actually a parent class for more specialised subclasses MessageBoxShowingEventArgs and TaskDialogShowingEventArgs. The actual instance class depends on the dialogue being handled.

The confirm message box is a task dialogue, so the latter subclass is used and the help id is set to -1.

The task dialogue event args provides additional properties to access the dialogue id and message being displayed. For the confirm message box, these are TaskDialog\_Save\_File and something similar to 'Do you want to save changes to filename.rvt?'

Therefore, we can install an external application which registers for this event, keeps its eyes open for this task dialogue being displayed, and then uses the OverrideResult method to answer Yes to the confirmation message.

The OverrideResult method takes an integer argument to specify the dialogue result, and is described a follows in the API documentation:

The range of valid result values depends on the type of dialog as follows:

- DialogBox: Any non-zero value will cause a dialog to be dismissed.- MessageBox: Standard Message Box IDs, such as IDOK and IDCANCEL, are accepted. For all possible IDs, please refer to the Windows API documentation. Naturally, the ID used must be relevant to the buttons in a message box.- TaskDialog: Either standard Message Box IDs or Revit Custom IDs are accepted depending on the buttons used in a dialog. Standard buttons, such as OK and Cancel, have standard IDs as described in Windows API documentation. Buttons with custom text have custom IDs with incremental values starting at 1001 for the left-most or top-most button in a task dialog.

The options displayed by the confirm message are Yes, No and Cancel, which correspond to standard Windows API message box ids. These are accessible through the DialogResult enumeration in the System.Windows.Forms namespace.

To explore and test this issue, I implemented an external application named AutoConfirmSave for you. It implements the IExternalApplication interface as follows, registering an event handler named a\_DialogBoxShowing to react to Revit dialogues popping up:

```csharp
public IExternalApplication.Result OnShutdown(
  ControlledApplication a )
{
  return IExternalApplication.Result.Succeeded;
}

public IExternalApplication.Result OnStartup(
  ControlledApplication a )
{
  a.DialogBoxShowing
    += new EventHandler<DialogBoxShowingEventArgs>(
      a\_DialogBoxShowing );
  return IExternalApplication.Result.Succeeded;
}
```

Here is the implementation of the event handler:

```csharp
void a\_DialogBoxShowing(
  object sender,
  DialogBoxShowingEventArgs e )
{
  TaskDialogShowingEventArgs e2
    = e as TaskDialogShowingEventArgs;

  string s = string.Empty;

  if( null != e2 )
  {
    s = string.Format(
      ", dialog id {0}, message '{1}'",
      e2.DialogId, e2.Message );

    bool isConfirm = e2.DialogId.Equals(
      "TaskDialog\_Save\_File" );

    if ( isConfirm )
    {
      e2.OverrideResult(
        (int) WinForms.DialogResult.Yes );

      s += ", auto-confirmed.";
    }
  }

  Debug.Print(
    "DialogBoxShowing: help id {0}, cancellable {1}{2}",
    e.HelpId,
    e.Cancellable ? "Yes" : "No",
    s );
}
```

This method checks whether the dialogue being displayed is a task dialogue by attempting to cast the event arguments to TaskDialogShowingEventArgs.
If this succeeds, it is a task dialogue and we can use the event argument DialogId to check whether it is the confirm message.
If so, we specify DialogResult.Yes as the result of the dialogue, which causes it not to be displayed to the user at all.
Finally, we print out some of the event argument properties, regardless of which dialogue it was.

Opening and closing a project named wall\_footing.rvt on my system displays the following message in the Visual Studio debug output window and automatically confirms the save, so the dialogue box is not displayed to the user:

```
DialogBoxShowing:
  help id -1,
  cancellable No,
  dialog id TaskDialog_Save_File,
  message 'Do you want to save changes to wall_footing.rvt?',
  auto-confirmed.
```

So all you need to do to achieve your task is to install the AutoConfirmSave external application alongside any existing applications, and the question whether to save your project will always be automatically answered with Yes.
You obviously need to be careful to remove the external application from Revit.ini again once you are finished processing your files, or you will never again see the message box asking you to confirm changes.

Here is the complete
[AutoConfirmSave](zip/AutoConfirmSave.zip)
Visual Studio solution implementing this external application.