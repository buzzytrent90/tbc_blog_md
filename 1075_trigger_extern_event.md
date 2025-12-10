---
post_number: "1075"
title: "Triggering Immediate External Event Execution"
slug: "trigger_extern_event"
author: "Jeremy Tammik"
tags: ['csharp', 'python', 'references', 'revit-api', 'rooms', 'windows']
source_file: "1075_trigger_extern_event.htm"
original_url: "https://thebuildingcoder.typepad.com/blog/1075_trigger_extern_event.html"
---

### Triggering Immediate External Event Execution

We held our first West European Devday in Paris on Monday and I am now sitting in the airport headed for Milano.

En route to Milano, I worked at testing Joe Ye's
[trick to trigger the Idling event](http://adndevblog.typepad.com/aec/2013/07/tricks-to-force-trigger-idling-event.html)
([potential SetRaiseWithoutDelay workaround](http://thebuildingcoder.typepad.com/blog/2013/07/curve-length-idling-units-and-revitpythonshell.html#3))
to force an
[immediate execution of my external event](http://thebuildingcoder.typepad.com/blog/2013/12/replacing-an-idling-event-handler-by-an-external-event.html#8).

I also added a comment from Guy Robinson on
[Revit requiring focus](http://thebuildingcoder.typepad.com/blog/2013/12/replacing-an-idling-event-handler-by-an-external-event.html#10) that
had me worried for a moment.

However, the solution is easy!

Before we get to that, here is our one and only photo from the DevDay event in Paris, a shot of beautiful evening sky at the Charles de Gaulle airport:

![Evening sky at Charles de Gaulle airport](file:///j/photo/jeremy/2013/2013-12-09_paris/sky_at_cdg.jpg)

#### Triggering External Event Execute by Setting Focus

Let me recapitulate the situation: I can call the external event Raise method from an external thread, giving me complete control over timing and performance overhead.

I can see from my debug output log messages that Revit has received and taken account of the Raise and the external event is now marked as pending.

However, Revit does not call the external event Execute method until some time later, often when the mouse cursor is moved over its window area.

Guy mentions
[Revit requiring focus](http://thebuildingcoder.typepad.com/blog/2013/12/replacing-an-idling-event-handler-by-an-external-event.html#10).

That can easily be achieved using the Windows API SetForegroundWindow provided by user32.dll.

I use it in conjunction with its sibling GetForegroundWindow method, so I can restore focus back to the original window after activating Revit for an instant.

They can both be referenced from user32.dll using p/invoke like this:

```csharp
  // DLL imports from user32.dll to set focus to
  // Revit to force it to forward the external event
  // Raise to actually call the external event
  // Execute.

  /// <summary>
  /// The GetForegroundWindow function returns a
  /// handle to the foreground window.
  /// </summary>
  [DllImport( "user32.dll" )]
  static extern IntPtr GetForegroundWindow();

  /// <summary>
  /// Move the window associated with the passed
  /// handle to the front.
  /// </summary>
  [DllImport( "user32.dll" )]
  static extern bool SetForegroundWindow(
    IntPtr hWnd );
```

With those references in place, I can momentarily shift focus to Revit in the CheckForPendingDatabaseChanges method, immediately after raising the external event, and Revit forwards that call to the external event Execute method right away, just as I had hoped:

```csharp
    if( rdb.LastSequenceNumberChanged(
      DbUpdater.LastSequence ) )
    {
      \_event.Raise();

      ++\_nUpdatesRequested;

      Util.Log( string.Format(
        "database update pending event raised {0} times",
        \_nUpdatesRequested ) );

      // Set focus to Revit for a moment.
      // Otherwise, it may take a while before
      // Revit forwards the event Raise to the
      // event handler Execute method.

      IntPtr hBefore = GetForegroundWindow();

      SetForegroundWindow(
        ComponentManager.ApplicationWindow );

      SetForegroundWindow( hBefore );
    }
```

With these three extra lines in place, Revit reacts immediately to my external event Raise.
This is also reflected in the log file (compare with the previous version
[waiting for Revit to react](http://thebuildingcoder.typepad.com/blog/2013/12/replacing-an-idling-event-handler-by-an-external-event.html#8)):

```
01:16:56.820 CheckForPendingDatabaseChanges loop 1 - check for changes 1
01:16:57.012 CheckForPendingDatabaseChanges loop 2 - check for changes 2
01:16:57.217 CheckForPendingDatabaseChanges loop 3 - check for changes 3
. . .
01:17:09.058 CheckForPendingDatabaseChanges loop 63 - check for changes 63
01:17:09.076 database update pending event raised 1 times
01:17:09.115 UpdateBim begin
01:17:09.216 UpdateBim end
01:17:09.283 CheckForPendingDatabaseChanges loop 64 - check for changes 64
01:17:09.457 CheckForPendingDatabaseChanges loop 65 - check for changes 65
01:17:09.562 CheckForPendingDatabaseChanges loop 66 - check for changes 66
. . .
01:17:19.144 CheckForPendingDatabaseChanges loop 153 - check for changes 153
01:17:19.249 CheckForPendingDatabaseChanges loop 154 - check for changes 154
01:17:19.305 database update pending event raised 2 times
01:17:19.333 UpdateBim begin
01:17:19.400 UpdateBim end
01:17:19.515 CheckForPendingDatabaseChanges loop 155 - check for changes 155
01:17:19.652 CheckForPendingDatabaseChanges loop 156 - check for changes 156
. . .
```

**Much** shorter, **instantaneous** reaction.

Yay!

Success!

Perfection!

As already explained, I much prefer this use of an external event over the Idling event, due to the detailed timing control over the polling frequency and possibility to run my polls in a separate thread, as opposed to the Idling all-or-nothing approach.

#### Download

I am quite happy about this and committed the changes implementing this version in the
[RoomEditorApp GitHub repository](https://github.com/jeremytammik/RoomEditorApp)
as
[release 2014.0.0.19](https://github.com/jeremytammik/RoomEditorApp/releases/tag/2014.0.0.19).

I hope that solves this issue for good.

Hardly likely, though.

Anyway, I urgently need to get some sleep now before the Milano DevDay conference, due to begin in just a very few hours...

Again, wish me luck, guys.