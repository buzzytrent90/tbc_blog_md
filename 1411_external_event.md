---
ai_optimized: true
author: Jeremy Tammik
complexity_score: 9.9
content_type: qa
optimization_date: '2025-12-11T11:44:15.961864'
original_url: https://thebuildingcoder.typepad.com/blog/1411_external_event.html
post_number: '1411'
reading_time_minutes: 12
series: general
slug: external_event
source_file: 1411_external_event.md
tags:
- csharp
- elements
- family
- parameters
- python
- revit-api
- sheets
- transactions
- views
- windows
title: External Event
word_count: 2319
---

### Implementing the TrackChangesCloud External Event
Today, I'll discuss this morning's work:
- [TrackChanges enhancement idea](#2)
- [The TrackChangesCloud add-in](#3)
- [The TrackChangesCloud external event](#4)
- [Creating and raising an external event](#5)
- [Raising the external event from a separate thread](#6)
- [First test run](#7)
- [Trigger immediate execution by setting Revit foreground window](#8)
- [Complete external application module](#9)
- [Next steps](#10)
- [Download](#11)
I implemented a suggestion
for [tracking element modification](http://thebuildingcoder.typepad.com/blog/2016/01/tracking-element-modification.html) a
month ago and created
the [TrackChanges GitHub repository](https://github.com/jeremytammik/TrackChanges) to host its solution and source code.
That met with significant interest and triggered the following idea for an enhancement.
#### TrackChanges Enhancement Idea
I am pondering an enhancement of this external command that I suggested to Tim Corneliussen in
the [Revit API discussion forum](http://forums.autodesk.com/t5/revit-api/bd-p/160) thread
on [dynamic model update after loading family](http://forums.autodesk.com/t5/revit-api/dynamic-model-update-after-loading-family/m-p/6052310):
Tim says he needs to track changes, and wondered whether to use the DocumentChanged event or the dynamic model updater framework DMU to do so.
I suggested that he take a look at TrackChanges instead.
\*\*Response:\*\* Your solution looks really impressive. I haven't had the chance to implement the main fundamentals in my project yet. As a starting programmer, the concept of hash code is still new to me but it looks like the right way to go. My main concern is how it will affect the performance of the routine.
The main purpose of my tool is that each addition or modification will be registered by modifying some parameters including a "time-parameter" and "date-parameter". To do so, but correct me if I'm wrong, I need to trigger an event or use a DMU to determine when an element is added or modified.
Maybe I can use your snapshot technique combining it with a DMU. But doing so the DMU also collects a lot of data next to the snapshot routine. Is this necessary? Are there alternatives to avoid this sort of useless multiple data collecting?
Do elements themselves contain relevant information about their own creation or modification (perhaps a certain property that most people aren't aware of)? If so, I can use a single event (sort of like your suggestion on your blog), for example the DocumentSavingAs event. Last possible solution I can think of at the moment is a way to look even deeper in to the updaterdata/-information hoping it contains more general information about the addition or modification of the relevant elements.
Hoping that you'll understand the scenario I'm describing. For now I will try to use the snapshot routine combining it with a DMU and a viewactivating event. Last mentioned will be used to determine whether another document becomes active (when a user has opened multiple projects). I will place an update when I have successfully created a working solution to discuss the results with you fellow readers. If someone can tell me if this solution probably won't really work please do so.
\*\*Answer:\*\* Thank you very much for your appreciation.
I think the main characteristic of the modification tracker is simplicity, rather than impressiveness.
Of course, simplicity is much more impressive than impressiveness :-)
If you want to be notified on every single modification of an element and store that information immediately, then indeed you can and have to use either DMU or the DocumentChanged event.
The latter does not allow you to modify anything in the same transaction, though, whereas the former does.
If you want to guarantee that your date and time markers stored in Revit parameters are always up to date, immediately, then you need to use DMU.
But do you really need that?
You need to understand that DMU is complex and adds a significant burden to Revit, depending on how many elements trigger it, which in your case would be many.
Do you really need to keep track of the element modification on a split second-by-second basis?
Would it not be enough to track changes every minute, or every ten minutes?
If so, then you can vastly simplify your approach and vastly reduce the burden on Revit by using the modification tracker and completely avoiding DMU and the DocumentChanged event.
Just track changes based on snapshots taken every X minutes, for instance.
Regarding the issue of the hash code: that is a minor detail, and pretty irrelevant.
You can just store the full data. Depending on what criteria you use to define when an element has changed, you might need to store a lot of information for each element.
I suggested the hash code as a way to reduce and unify that data storage, but that has absolutely nothing to do with the fundamental concept.
To quote the original post: "We use the hash code to determine whether the state has been modified compared to a new element state snapshot made at a later time. We could obviously also store the entire original string representation instead of using a hash code. The hash code is small and handy, whereas the entire string contains all the original data. It is up to you to choose which you would like to use."
The hash code will not affect performance much, just reduce the memory used to cache the starting snapshot.
I would recommend thinking this through in depth and peace and quiet.
If you do not require split-second time slice data, I would avoid the DMU and DocumentChanged events, both, completely.
I am very much looking forward to hearing and discussing your further thoughts on this.
#### The TrackChangesCloud Add-In
I am thinking of testing the suggestions I propose above myself.
My idea is:
- Trigger automatic snapshots of element modifications every couple of minutes.
- Store the results in a cloud-based database.
That will also represent another step in my explorations on connecting the desktop and the cloud.
I created a new add-in [TrackChangesCloud](https://github.com/jeremytammik/TrackChangesCloud) to explore them in.
I prefer to leave the original TrackChanges unmodified, since it clearly and simply demonstrates the snapshot idea.
It would be a shame to make it more confusing by adding the complexity required by the external application and event handling.
One way to trigger the snapshots at regular intervals, and probably the cleanest approach, is by implementing an external event.
#### The TrackChangesCloud External Event
The Building Coder topic group
on [Idling and external events](http://thebuildingcoder.typepad.com/blog/about-the-author.html#5.28) is
growing pretty large and explains all the background information on this in full detail.
I implemented a skeleton for the external event named `ModificationLogger` that does nothing so far, except get raised when I say so.
Therefore, the implementation is completely trivial:

```
  class ModificationLogger : IExternalEventHandler
  {
    ///
    /// Required IExternalEventHandler interface
    /// method returning a descriptive name.
    ///
    public string GetName()
    {
      return "TrackChangesCloud ModificationLogger";
    }

    ///
    /// Execute method invoked by Revit via the
    /// external event as a reaction to a call
    /// to its Raise method.
    ///
    public void Execute( UIApplication a )
    {
      Util.Log( "ModificationLogger.Execute" );
    }
  }
```

Later on, the `Execute` method will do more, of course, e.g. create a modification tracking snapshot and store it in the cloud.
#### Creating and Raising an External Event
These two steps are both completely trivial one-liners, implemented in the external application module `App.cs`.
In the `OnStartup` method, I subscribe to the `ApplicationInitialized` event:

```
  a.ControlledApplication.ApplicationInitialized
    += OnApplicationInitialized;
```

In the event handler, I create the external event by instantiating my implementation:

```
  // Create our custom external event.

  _event = ExternalEvent.Create(
    new ModificationLogger() );
```

To trigger the event, causing Revit to wait for an opportune moment and then call its `Execute` method, I call its `Raise` method:

```
  _event.Raise();
```

That's all there is to it.
But how do we ensure that the `Raise` call is executed at the times we wish?
For that I implemented a separate driver thread, which is also very simple.
#### Raising the External Event from a Separate Thread
In order to control the regular raising of my external event, I can run an infinite triggering loop in a separate thread.
The loop is implemented in the following `TriggerModificationLogger` method:

```
  ///
  /// Trigger a modification tracker snapshot at
  /// regular intervals. Relinquish control and wait
  /// for the specified timeout period between each
  /// snapshot. This method runs in a separate thread.
  ///
  static void TriggerModificationLogger()
  {
    while( true )
    {
      ++_nSnapshots;

      Util.Log( string.Format(
        "TriggerModificationLogger snapshot {0}",
        _nSnapshots ) );

      _event.Raise();

      // Wait and relinquish control
      // before next snapshot.

      Thread.Sleep( _timeout );
    }
  }
```

It endlessly loops, raising the external event at regular intervals specified by the `_timeout` constant.
I start the looping in the `ApplicationInitialized` event handler, directly after instantiating my external event:

```
  void OnApplicationInitialized(
    object sender,
    ApplicationInitializedEventArgs e )
  {
    // Create our custom external event.

    _event = ExternalEvent.Create(
      new ModificationLogger() );

    // Start a thread to raise it regularly.

    _thread = new Thread(
      TriggerModificationLogger );

    _thread.Start();
  }
```

#### First Test Run
This works perfectly, with one teeny weeny little flaw: even though the external event is raised at the moment I specify, the `Execute` method is not called until the Revit window is activated:

```
TrackChanges 10:59:14.067 TriggerModificationLogger snapshot 1
TrackChanges 10:59:14.635 ModificationLogger.Execute
TrackChanges 11:00:14.074 TriggerModificationLogger snapshot 2
*manually activated Revit*
TrackChanges 11:00:44.050 ModificationLogger.Execute
TrackChanges 11:01:14.075 TriggerModificationLogger snapshot 3
*manually activated Revit*
TrackChanges 11:01:23.877 ModificationLogger.Execute
```

This may or may not be a problem, depending on the intended use.
Maybe you don't care if the external event is not triggered as long as Revit is in the background.
However, this is quite easy to rectify, so let's do so.
#### Trigger Immediate Execution by Setting Revit Foreground Window
I implemented the following helper method to check whether Revit is the current foreground window.
If it is not, it is temporarily brought to the foreground using the Windows API:

```
  // DLL imports from user32.dll to set focus to
  // Revit to force it to forward the external event
  // Raise to actually call the external event
  // Execute.

  ///
  /// The GetForegroundWindow function returns a
  /// handle to the foreground window.
  ///
  [DllImport( "user32.dll" )]
  static extern IntPtr GetForegroundWindow();

  ///
  /// Move the window associated with the passed
  /// handle to the front.
  ///
  [DllImport( "user32.dll" )]
  static extern bool SetForegroundWindow(
    IntPtr hWnd );

  static void SetFocusToRevit()
  {
    IntPtr hRevit = ComponentManager.ApplicationWindow;
    IntPtr hBefore = GetForegroundWindow();

    if( hBefore != hRevit )
    {
      SetForegroundWindow( hRevit );
      SetForegroundWindow( hBefore );
    }
  }
```

Now, if I add a call to `SetFocusToRevit` directly after calling `Raise`, the `Execute` method is invoked immediately, regardless of whether Revit is in the background or not:

```
TrackChanges 11:26:23.431 TriggerModificationLogger snapshot 1
TrackChanges 11:26:24.047 ModificationLogger.Execute
TrackChanges 11:27:23.445 TriggerModificationLogger snapshot 2
TrackChanges 11:27:23.499 ModificationLogger.Execute
TrackChanges 11:28:23.506 TriggerModificationLogger snapshot 3
TrackChanges 11:28:23.560 ModificationLogger.Execute
TrackChanges 11:29:23.569 TriggerModificationLogger snapshot 4
TrackChanges 11:29:23.623 ModificationLogger.Execute
TrackChanges 11:30:23.638 TriggerModificationLogger snapshot 5
TrackChanges 11:30:23.697 ModificationLogger.Execute
TrackChanges 11:31:23.706 TriggerModificationLogger snapshot 6
TrackChanges 11:31:23.790 ModificationLogger.Execute
```

No manual intervention required.
However, the screen will flash!
If the purpose of this tool is to track modifications made to the BIM by end users, then obviously they will have the Revit window up and visible, so the call to `SetFocusToRevit` can be removed again.
#### Complete External Application Module
To show how the external event and thread handling work together, here is the complete implementation of the external application module `App.cs`:

```
class App : IExternalApplication
{
  ///
  /// Store the external event.
  ///
  static ExternalEvent _event = null;

  ///
  /// Separate thread running a timer to trigger
  /// the modification tracker at regular intervals.
  ///
  static Thread _thread = null;

  ///
  /// Count total number of modification tracker
  /// snapshots taken so far in this session.
  ///
  static int _nSnapshots = 0;

  static int _timeout_minutes = 1;

  ///
  /// Number of milliseconds to wait and relinquish
  /// CPU control before next snapshot.
  ///
  static int _timeout = 1000 * 60 * _timeout_minutes;

  #region SetFocusToRevit
  // DLL imports from user32.dll to set focus to
  // Revit to force it to forward the external event
  // Raise to actually call the external event
  // Execute.

  ///
  /// The GetForegroundWindow function returns a
  /// handle to the foreground window.
  ///
  [DllImport( "user32.dll" )]
  static extern IntPtr GetForegroundWindow();

  ///
  /// Move the window associated with the passed
  /// handle to the front.
  ///
  [DllImport( "user32.dll" )]
  static extern bool SetForegroundWindow(
    IntPtr hWnd );

  static void SetFocusToRevit()
  {
    IntPtr hRevit = ComponentManager.ApplicationWindow;
    IntPtr hBefore = GetForegroundWindow();

    if( hBefore != hRevit )
    {
      SetForegroundWindow( hRevit );
      SetForegroundWindow( hBefore );
    }
  }
  #endregion // SetFocusToRevit

  ///
  /// Trigger a modification tracker snapshot at
  /// regular intervals. Relinquish control and wait
  /// for the specified timeout period between each
  /// snapshot. This method runs in a separate thread.
  ///
  static void TriggerModificationLogger()
  {
    while( true )
    {
      ++_nSnapshots;

      Util.Log( string.Format(
        "TriggerModificationLogger snapshot {0}",
        _nSnapshots ) );

      _event.Raise();

      // Set focus to Revit for a moment.
      // Without this, Revit will not forward the
      // event Raise to the external event handler
      // Execute method until the Revit window is
      // activated. This causes the screen to flash.

      SetFocusToRevit();

      // Wait and relinquish control
      // before next snapshot.

      Thread.Sleep( _timeout );
    }
  }

  public Result OnStartup( UIControlledApplication a )
  {
    a.ControlledApplication.ApplicationInitialized
      += OnApplicationInitialized;

    return Result.Succeeded;
  }

  void OnApplicationInitialized(
    object sender,
    ApplicationInitializedEventArgs e )
  {
    // Create our custom external event.

    _event = ExternalEvent.Create(
      new ModificationLogger() );

    // Start a thread to raise it regularly.

    _thread = new Thread(
      TriggerModificationLogger );

    _thread.Start();
  }

  public Result OnShutdown( UIControlledApplication a )
  {
    _thread.Abort();
    _thread = null;
    _event.Dispose();

    return Result.Succeeded;
  }
}
```

#### Next Steps
The next steps are obvious:
- Implement the cloud database to hold the modification tracking data.
- Implement the database upload.
Both of these are clearly and extensively demonstrated by the cloud storage approach developed for
the [FireRatingCloud project](https://github.com/jeremytammik/FireRatingCloud).
#### Download
This project is hosted in
the [TrackChanges GitHub repository](https://github.com/jeremytammik/TrackChanges),
and the version discussed above
is [release 2016.0.0.4](https://github.com/jeremytammik/TrackChangesCloud/releases/tag/2016.0.0.4).
![TrackChanges diff](img/track_changes_diff.png)