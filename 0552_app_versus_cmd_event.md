---
post_number: "0552"
title: "Application Versus Command Event"
slug: "app_versus_cmd_event"
author: "Jeremy Tammik"
tags: ['csharp', 'family', 'levels', 'revit-api']
source_file: "0552_app_versus_cmd_event.htm"
original_url: "https://thebuildingcoder.typepad.com/blog/0552_app_versus_cmd_event.html"
---

### Application Versus Command Event

Here is an interesting observation by Ning Zhou on the difference between registering a handler for the DialogBoxShowing event in an external application versus an external command.

Here is the initial query:

**Question:** We have an add-in which loads multiple families and suppresses the warning message which is displayed in case family instances already exist for some of them by registering a DialogBoxShowing event handler.

The problem with this is that the installation of this event handler also affects the normal Revit load family behaviour from the user interface.
The warning message is suppressed for load family using insert, but load family using reload still displays it.

If I uninstall our add-in, both load family methods display the warning message again.

Can you provide any solution or workaround for this?

Here are the relevant code snippets to explain what we are doing:
```csharp
public CmdResult OnStartup(
  UIControlledApplication application )
{
  application.DialogBoxShowing
    += new EventHandler<DialogBoxShowingEventArgs>(
      myDialogBoxShowing );
  // ...
}

void myDialogBoxShowing(
  object sender,
  DialogBoxShowingEventArgs e )
{
  TaskDialogShowingEventArgs e1
    = e as TaskDialogShowingEventArgs;

  if( e1 != null
    && e1.DialogId.Equals(
      "TaskDialog\_Family\_Already\_Exists" ) )
  {
    e1.OverrideResult( ( int )
      TaskDialogResult.CommandLink1 ); // 1001
  }
}
```

**Solution:** I fixed it myself by placing the DialogBoxShowing event at command level instead of application level.

**Response:** Yes, exactly, if you register the DialogBoxShowing event handler at the beginning of an external command and then unregister it again at the end of the command, the warning message will be suppressed only for the duration of your command.

After your command ends, the dialogue suppressing mechanism is deactivated again.
Without the double negation: the messages are displayed as normal again.
This explains why the warning dialogue box is showing again when you manually insert a family file.

If you register the DialogBoxShowing event handler in the external application OnStartup method, the dialogue suppressing mechanism is always active.
Then it can dismiss the warning dialogue both when your own external command is programmatically loading families and also when you manually insert a family file from disk through the user interface.