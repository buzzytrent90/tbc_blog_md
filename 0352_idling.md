---
ai_optimized: true
author: Jeremy Tammik
complexity_score: 5.5
content_type: qa
optimization_date: '2025-12-11T11:44:13.793439'
original_url: https://thebuildingcoder.typepad.com/blog/0352_idling.html
post_number: '0352'
reading_time_minutes: 6
series: general
slug: idling
source_file: 0352_idling.htm
tags:
- csharp
- doors
- elements
- revit-api
- transactions
- walls
- windows
title: Idling Event
word_count: 1108
---

### Idling Event

One of the most exciting of the many
[new Revit 2011 API features](http://thebuildingcoder.typepad.com/blog/2010/03/revit-2011-is-coming.html) which
opens up completely new possibilities for applications to interact with Revit is the Idling event.
The What's New section of the Revit API help file RevitAPI.chm describes it as follows:

The new event

- UIApplication.Idling

is raised when it is safe for the API application to access the active document between user interactions.
This event is raised only when the Revit UI is in a state where the user could successfully click on an API command button.
The event allows changes to a document if a new transaction is opened.

Because this event is invoked between user actions in the Revit UI, if the handler for this event requires a significant amount of processing time, users will perceive a slowdown in the responsiveness of Revit.
If the execution for updates can be safely split across multiple calls to this event, the user perception of Revit responsiveness will be improved.

In the detailed description of the event itself, it says further:

Subscribe to the Idling event to be notified when Revit is not in an active tool or transaction.

This event is raised when it is safe for the API application to access the active document between user interactions. The event is raised only when the Revit UI is in a state where the user could successfully click on an API command button.

Handlers of this event are permitted to make modifications to any document (including the active document), except for documents that are currently in read-only mode.

In order to change a document, you must begin a new transaction for that document. This transaction will appear in the Revit undo stack and may be undone by the Revit user.

This event is invoked between user actions in the Revit UI. If the handler for this event requires a significant amount of processing time, users will perceive a slowdown in the responsiveness of Revit. If the execution for updates can be safely split across multiple calls to this event, the user perception of Revit responsiveness will be improved.

Event is not cancellable.

As far as I can tell, there is no Revit SDK sample demonstrating its use.

The reason why this is so exciting is because it makes it a little bit more feasible to
[control Revit from an external application](http://thebuildingcoder.typepad.com/blog/2008/12/driving-revit-from-outside.html) or a
[modeless dialogue](http://thebuildingcoder.typepad.com/blog/2009/12/modeless-dialogues-in-revit.html),
topics that have always been very high on the developer wish list and frequently discussed in this blog.
The fact still remains in Revit 2011 that Revit insists on being the boss and cannot be driven from outside by any direct API calls, but at least this new event makes it possible to set up a pair of an external application wishing to do something from outside and an internal Revit plug-in that supports it in doing so.

The scenario that I envision would be that the external application raises some kind of signal letting the plug-in know that it would like to do something and possibly provide detailed data on what that is.
The internal plug-in registers to the Idling event, and every time it is raised, it checks whether the external application has queued up some task for it and then performs it.

I implemented a new little minimal Building Coder sample command CmdIdling to demonstrate how to subscribe to this event.

The event handler for this event takes two arguments, the sender and the event arguments, as defined by the
[standard .NET event model](http://msdn.microsoft.com/en-us/library/db0etb8x).
The Revit API defines a new class IdlingEventArgs for it, derived from the PreEventArgs and RevitEventArgs classes without adding any extra functionality, so it does not provide any useful members for us to access the application or documents through.

Happily, the sender argument is in fact the Revit application instance, a fact that is currently not documented anywhere, as far as I can tell.
This obviously provides access to the documents and the active document, enabling us to open a transaction and perform any required actions on the Revit model.

In my minimalistic first sample, I do nothing but demonstrate the access to the active document and log a message with a time stamp using the following little helper method:
```csharp
void Log( string msg )
{
  string dt = DateTime.Now.ToString( "u" );
  Debug.Print( dt + " " + msg );
}
```

Here is my event handler implementation demonstrating how to access the Revit application instance from the sender argument, obtain and query the active document, and log a message to the Visual Studio debug output window:
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

The command implementation is simpler still, basically just one single line of code to register my event handler and subscribe to the UI application event:
```csharp
public Result Execute(
  ExternalCommandData commandData,
  ref string message,
  ElementSet elements )
{
  Log( "Execute begin" );

  UIApplication uiapp = commandData.Application;

  uiapp.Idling
    += new EventHandler<IdlingEventArgs>(
      OnIdling );

  Log( "Execute end" );

  return Result.Succeeded;
}
```

Since I am not modifying the database in any way whatsoever, I use the most restrictive
[regeneration option](http://thebuildingcoder.typepad.com/blog/2010/04/regeneration-option-best-practices.html) and
transaction mode:
```csharp
[Transaction( TransactionMode.ReadOnly )]
[Regeneration( RegenerationOption.Manual )]
```

As soon as I launch this command, it prints out its initial two log messages specified in the Execute method.
From that moment onwards, the Idling event fires almost continuously as long as no other Revit commands are invoked, so my event handler method is called with great frequency:

```
2010-04-22 12:04:18Z Execute begin
2010-04-22 12:04:18Z Execute end
2010-04-22 12:04:18Z OnIdling with active document wall.rvt
2010-04-22 12:04:18Z OnIdling with active document wall.rvt
2010-04-22 12:04:18Z OnIdling with active document wall.rvt
. . .
```

This is yet another addition to the Revit API which really opens up completely new doors, so it will be very exciting to see what use can be made of this in the coming months!

Here is
[version 2011.0.0.65](zip/bc_11_65.zip)
of the complete Visual Studio solution including the new command.