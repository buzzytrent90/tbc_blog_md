---
ai_optimized: true
author: Jeremy Tammik
complexity_score: 11.1
content_type: code_example
optimization_date: '2025-12-11T11:44:15.250736'
original_url: https://thebuildingcoder.typepad.com/blog/1074_roomedit_extern_event.html
post_number: '1074'
reading_time_minutes: 15
series: general
slug: roomedit_extern_event
source_file: 1074_roomedit_extern_event.htm
tags:
- csharp
- elements
- python
- revit-api
- rooms
- schedules
- views
- windows
title: Replacing an Idling Event Handler by an External Event
word_count: 3086
---

### Replacing an Idling Event Handler by an External Event

I arrived back in Europe safe and sound from America.

Healthy, as well, in spite of air conditioning and the freezing temperatures in some of the Autodesk University conference rooms.

Thank God, I went well prepared with long-sleeved woollen underwear and thick heavy sweaters to face the arctic challenges of the desert town.

As I hinted at when describing my
[last day at AU](http://thebuildingcoder.typepad.com/blog/2013/12/au-day-3-recap-cloud-based-round-trip-model-editing.html),
I realised that I can significantly improve the implementation of the RoomEditorApp subscription to real-time database changes by replacing the Idling event I was using previously by an external event:

- [Idling versus an external event](#2)
- [Implementing the RoomEditorApp external event](#3)
- [Implementing the IExternalEventHandler interface](#4)
- [External event creation and disposal](#5)
- [Creating and managing the external event](#6)
- [Checking for pending database changes in a separate thread](#7)
- [Waiting for Revit to react](#8)
- [Differences, download and conclusion](#9)
- [Revit requires focus](#10)

I implemented that during the trip home, actually, in a very few hours.

I spent significantly more time over the weekend putting together the following detailed documentation, so I really hope you enjoy and appreciate it.

That obviously means that I had less time to prepare for the series of European DevDays conferences starting Monday morning in Paris.

I'll have to wing it somehow, I guess.

Here is the sequence of stops I will attend:

- 2013-12-09 – DevDay, Paris, France
- 2013-12-10 – DevDay, Milan, Italy
- 2013-12-11 – DevDay, Farnborough, United Kingdom
- 2013-12-12 – DevDay, Munich, Germany
- 2013-12-13 – DevLab, Munich, Germany
- 2013-12-16 – DevDay, Gothenburg, Sweden

If you are interested in the full list of events, please look at the complete
[DevDays schedule](http://thebuildingcoder.typepad.com/blog/2013/09/back-and-preparing-the-au-and-devdays-conferences.html#2).
As said,
[you are welcome to join](http://thebuildingcoder.typepad.com/blog/2013/10/invitation-to-autodesk-devdays-2013.html),
regardless of ADN membership.

By the time you read this, the DevDay conference in Paris will already be in full tilt.

Back to the topic at hand:

#### Idling versus an External Event

In fact, maybe it is best to avoid using the Idling event completely when you require a continuous check for external changes to react to, since it only gives you the choice between either calling SetRaiseWithoutDelay or not.

If you call it, Revit will hog the processor and bring down the system performance.

If you do not, you will only be called once every time Revit enters the Idling state.

A new Idling state can be triggered in Revit by just moving the cursor across its graphics screen.
That effect is described and used by Joe Ye to implement a programmatic
[trick to trigger the Idling event](http://adndevblog.typepad.com/aec/2013/07/tricks-to-force-trigger-idling-event.html),
as briefly mentioned as a
[potential SetRaiseWithoutDelay workaround](http://thebuildingcoder.typepad.com/blog/2013/07/curve-length-idling-units-and-revitpythonshell.html#3).

Use of an external event instead of Idling gives an add-in much more control over the timing behaviour.

You can run your external check in a separate thread, make full use of the various .NET or Windows threading options to make regular checks for pending updates at specified intervals and totally relinquish control of the CPU in between, avoiding the performance hit of using SetRaiseWithoutDelay and yet activating the check as often as desired.

I rewrote the RoomEditorApp check for database updates using an external event instead of Idling as described below.

After completing that and getting it to work perfectly, just the way I want, I noticed one unexpected little drawback:

When I raise the external event with nothing else happening in the system, Revit does not react immediately.
It just sits there doing nothing, although I can see from my debugging messages that the external event I registered and raised is pending.

If I move the cursor over the Revit screen, the pending external event is processed almost immediately.

Sometimes Revit decides to process it without moving the cursor, but hardly ever right away.

This prompted me to remember the trick described above, and the next step I plan is to update the RoomEditorApp to trigger immediate external event processing by simulating a mouse move event in the Revit graphics screen.

Let's look at the individual steps in detail.

#### Implementing the RoomEditorApp External Event

An external event is created by the Revit API by calling the static ExternalEvent.Create method and passing in the custom event handler to it, represented by an instance of a class implementing the IExternalEventHandler interface.

The previous RoomEditorApp implementation working with the Idling event handler already defined a DbUpdater class providing a UpdateBim method that was called by the handler when it determined the presence of pending database updates.

That represents an obvious candidate for implementing the new external event handler interface, so I added IExternalEventHandler as a new DbUpdater base class:

```csharp
  class DbUpdater : IExternalEventHandler
```

The Idling version of the DbUpdater class kept track of the current Revit document in a dedicated member variable \_doc.

The new version is more or less document independent – I hope!
It stores the Revit UI application instead and retrieves the currently active document from that when needed.
This will certainly work fine when you are only working with one single document, as is the case in my testing scenario:

```csharp
  /// <summary>
  /// Current Revit project document.
  /// </summary>
  //Document \_doc = null;

  /// <summary>
  /// Revit UI application.
  /// </summary>
  UIApplication \_uiapp = null;
```

Except for that modification, the **existing** DbUpdater class implementation remains unchanged.

The new **additional** methods handle two aspects:

- [Implementing the IExternalEventHandler interface](#4)
- [Checking for pending database changes in a separate thread](#6)

#### Implementing the IExternalEventHandler Interface

The IExternalEventHandler interface is pretty minimal.

All it requires is the implementation of two methods:

- The **Execute** method invoked by Revit via the external event as a reaction to a call to its Raise method.
- The **GetName** method returning a descriptive name.

This is what they look like in my case:

```csharp
  /// <summary>
  /// Execute method invoked by Revit via the
  /// external event as a reaction to a call
  /// to its Raise method.
  /// </summary>
  public void Execute( UIApplication a )
  {
    // As far as I can tell, the external event
    // should work fine even when switching between
    // different documents. That, however, remains
    // to be tested in more depth (or at all).

    //Document doc = a.ActiveUIDocument.Document;

    //Debug.Assert( doc.Title.Equals( \_doc.Title ),
    //  "oops ... different documents ... test this" );

    UpdateBim();
  }

  /// <summary>
  /// Required IExternalEventHandler interface
  /// method returning a descriptive name.
  /// </summary>
  public string GetName()
  {
    return string.Format(
      "{0} DbUpdater", App.Caption );
  }
```

#### External event creation and disposal

The creation and disposal of the external event is more trivial still.

Just like the Idling event subscription, I decided to handle it in the main add-in external application implementation.

Instead of keeping track of the Idling event handler, it now holds on to an instance of the external event it creates:

```csharp
  /// <summary>
  /// Store the Idling event handler when subscribed.
  /// </summary>
  //static EventHandler<IdlingEventArgs> \_handler = null;

  /// <summary>
  /// Store the external event.
  /// </summary>
  static ExternalEvent \_event = null;
```

The actual creation and disposal in achieved by the ToggleSubscription that was already in place for the Idling event subscription:

```python
  /// <summary>
  /// Toggle on and off subscription to
  /// automatic cloud updates.
  /// </summary>
  public static ExternalEvent ToggleSubscription(
    // EventHandler<IdlingEventArgs> handler
    IExternalEventHandler handler )
  {
    if( Subscribed )
    {
      Debug.Print( "Unsubscribing..." );
      //\_uiapp.Idling -= \_handler;
      //\_handler = null;
      \_event.Dispose();
      \_event = null;
      \_buttons[3].ItemText = \_subscribe;
      \_timer.Stop();
      \_timer.Report( "Subscription timing" );
      \_timer = null;
      Debug.Print( "Unsubscribed." );
    }
    else
    {
      Debug.Print( "Subscribing..." );
      //\_uiapp.Idling += handler;
      //\_handler = handler;
      \_event = ExternalEvent.Create( handler );
      \_buttons[3].ItemText = \_unsubscribe;
      \_timer = new JtTimer( "Subscription" );
      Debug.Print( "Subscribed." );
    }
    return \_event;
  }
```

#### Creating and Managing the External Event

OK, OK, so all the above is obvious and trivial, you say?

So where is the challenge, what is the interesting part, where is the use and big advantage of all this?

Well, that is the part we address now:

By escaping from the Idling event invoked by Revit internally as it sees fit, with the minimal timing control provided by the SetRaiseWithoutDelay method, we gain several advantages:

- We have full control over the timing of our calls to the external event Raise method.
- We have full access to the methods provided by the Windows and .NET APIs, e.g. Sleep, timers, threads etc.
- We can run the check for external changes that need to be reacted on in a separate thread!

Instead of calling the main external application's ToggleSubscription method directly and passing in the Idling event handler, the external command to subscribe to database changes now calls an intermediate ToggleSubscription method implemented by the DbUpdater class that forwards it to App.ToggleSubscription and simultaneously launches the separate thread.

The entire read-only CmdSubscribe implementation thus now looks like this:

```csharp
  public Result Execute(
    ExternalCommandData commandData,
    ref string message,
    ElementSet elements )
  {
    if( !App.Subscribed
      && -1 == DbUpdater.LastSequence )
    {
      DbUpdater.SetLastSequence();
    }

    //App.ToggleSubscription( OnIdling );

    DbUpdater.ToggleSubscription(
      commandData.Application );

    return Result.Succeeded;
  }
```

The DbUpdater.ToggleSubscription method thus achieves the following:

- Toggle subscription to automatic database updates.
- Forward the call to the external application that creates the external event.
- Store the external event.
- Launch a separate thread checking for database updates.
- On Detecting pending changes, invoke the external event Raise method.

The code to achieve this is actually more succinct than the verbal description:

```csharp
  /// <summary>
  /// Toggle subscription to automatic database
  /// updates. Forward the call to the external
  /// application that creates the external event,
  /// store it and launch a separate thread checking
  /// for database updates. When changes are pending,
  /// invoke the external event Raise method.
  /// </summary>
  public static void ToggleSubscription(
    UIApplication uiapp )
  {
    \_event = App.ToggleSubscription(
      new DbUpdater( uiapp ) );

    if( null == \_event )
    {
      \_thread.Abort();
      \_thread = null;
    }
    else
    {
      // Start a new thread to regularly check the
      // database status and raise the external event
      // when updates are pending.

      \_thread = new Thread(
        CheckForPendingDatabaseChanges );

      \_thread.Start();
    }
  }
```

#### Checking for Pending Database Changes in a Separate Thread

So, we started a new thread to check for database changes.

That was awfully simple, wasn't it?

When creating it, we pass in a method to execute.

It is not complicated either.

In fact, it is completely trivial as well, and just slightly bloated by debug and log messages for monitoring its behaviour.

It currently looks like this, including a couple of bookkeeping and tweaking variables that might come in handy for measuring and optimising performance:

```csharp
  /// <summary>
  /// Count total number of checks for
  /// database updates made so far.
  /// </summary>
  static int \_nLoopCount = 0;

  /// <summary>
  /// Count total number of checks for
  /// database updates made so far.
  /// </summary>
  static int \_nCheckCount = 0;

  /// <summary>
  /// Count total number of database
  /// updates requested so far.
  /// </summary>
  static int \_nUpdatesRequested = 0;

  /// <summary>
  /// Number of milliseconds to wait and relinquish
  /// CPU control before next check for pending
  /// database updates.
  /// </summary>
  static int \_timeout = 100;

  /// <summary>
  /// Repeatedly check database status and raise
  /// external event when updates are pending.
  /// Relinquish control and wait for timeout
  /// period between each attempt. Run in a
  /// separate thread.
  /// </summary>
  static void CheckForPendingDatabaseChanges()
  {
    while( true )
    {
      ++\_nLoopCount;

      Debug.Assert( null != \_event,
      "expected non-null external event" );

      if( \_event.IsPending )
      {
        Util.Log( string.Format(
          "CheckForPendingDatabaseChanges loop {0} - "
          + "database update event is pending",
          \_nLoopCount ) );
      }
      else
      {
        using( JtTimer pt = new JtTimer(
          "CheckForPendingDatabaseChanges" ) )
        {
          ++\_nCheckCount;

          Util.Log( string.Format(
            "CheckForPendingDatabaseChanges loop {0} - "
            + "check for changes {1}",
            \_nLoopCount, \_nCheckCount ) );

          RoomEditorDb rdb = new RoomEditorDb();

          if( rdb.LastSequenceNumberChanged(
            DbUpdater.LastSequence ) )
          {
            \_event.Raise();

            ++\_nUpdatesRequested;

            Util.Log( string.Format(
              "database update pending event raised {0} times",
              \_nUpdatesRequested ) );
          }
        }
      }

      // Wait a moment and relinquish control before
      // next check for pending database updates.

      Thread.Sleep( \_timeout );
    }
  }
```

Most significantly, of course, the \_timeout variable controls the number of milliseconds to wait between each repetition of the database check for pending updates.

#### Waiting for Revit to React

As I mentioned in the beginning of this discussion, when the external event has been raised, Revit does not actually call the external event Execute method until a cursor movement or some other system event wakes it up.

Here is an extract from a sample log file demonstrating this.
I generated it by making changes to the cloud database via the browser, observing that the external event is raised immediately, waiting for a while and then moving the mouse to trigger Revit to forward the notification by calling the external event (copy and paste somewhere or view source to see truncated lines in full):

```
19:12:16.796 CheckForPendingDatabaseChanges loop 1 - check for changes 1
19:12:17.075 CheckForPendingDatabaseChanges loop 2 - check for changes 2
19:12:17.224 CheckForPendingDatabaseChanges loop 3 - check for changes 3
. . .
19:12:28.227 CheckForPendingDatabaseChanges loop 105 - check for changes 105
19:12:28.332 CheckForPendingDatabaseChanges loop 106 - check for changes 106
19:12:33.189 database update pending event raised 1 times
19:12:33.291 CheckForPendingDatabaseChanges loop 107 - database update event is pending
19:12:33.394 CheckForPendingDatabaseChanges loop 108 - database update event is pending
19:12:33.496 CheckForPendingDatabaseChanges loop 109 - database update event is pending
. . .
19:12:34.825 CheckForPendingDatabaseChanges loop 122 - database update event is pending
19:12:34.927 CheckForPendingDatabaseChanges loop 123 - database update event is pending
19:12:35.067 CheckForPendingDatabaseChanges loop 124 - check for changes 107
19:12:35.177 CheckForPendingDatabaseChanges loop 125 - check for changes 108
19:12:35.283 CheckForPendingDatabaseChanges loop 126 - check for changes 109
. . .
19:12:51.476 CheckForPendingDatabaseChanges loop 276 - check for changes 259
19:12:51.581 CheckForPendingDatabaseChanges loop 277 - check for changes 260
19:12:51.584 database update pending event raised 2 times
19:12:51.686 CheckForPendingDatabaseChanges loop 278 - database update event is pending
19:12:51.789 CheckForPendingDatabaseChanges loop 279 - database update event is pending
19:12:51.891 CheckForPendingDatabaseChanges loop 280 - database update event is pending
. . .
19:13:11.498 CheckForPendingDatabaseChanges loop 470 - database update event is pending
19:13:11.613 CheckForPendingDatabaseChanges loop 471 - database update event is pending
19:13:11.725 CheckForPendingDatabaseChanges loop 472 - check for changes 261
19:13:11.833 CheckForPendingDatabaseChanges loop 473 - check for changes 262
19:13:11.938 CheckForPendingDatabaseChanges loop 474 - check for changes 263
. . .
19:13:32.217 CheckForPendingDatabaseChanges loop 660 - check for changes 449
19:13:32.222 database update pending event raised 3 times
19:13:32.326 CheckForPendingDatabaseChanges loop 661 - database update event is pending
19:13:32.427 CheckForPendingDatabaseChanges loop 662 - database update event is pending
19:13:32.530 CheckForPendingDatabaseChanges loop 663 - database update event is pending
. . .
19:13:37.188 CheckForPendingDatabaseChanges loop 708 - database update event is pending
19:13:37.291 CheckForPendingDatabaseChanges loop 709 - database update event is pending
19:13:37.470 CheckForPendingDatabaseChanges loop 710 - check for changes 450
19:13:37.587 CheckForPendingDatabaseChanges loop 711 - check for changes 451
19:13:37.692 CheckForPendingDatabaseChanges loop 712 - check for changes 452
. . .
19:13:45.076 CheckForPendingDatabaseChanges loop 780 - check for changes 520
19:13:45.184 CheckForPendingDatabaseChanges loop 781 - check for changes 521
19:13:45.188 database update pending event raised 4 times
19:13:45.291 CheckForPendingDatabaseChanges loop 782 - database update event is pending
19:13:45.444 CheckForPendingDatabaseChanges loop 783 - database update event is pending
19:13:45.838 CheckForPendingDatabaseChanges loop 784 - database update event is pending
. . .
19:13:47.967 CheckForPendingDatabaseChanges loop 789 - database update event is pending
19:13:48.387 CheckForPendingDatabaseChanges loop 790 - database update event is pending
19:13:48.989 CheckForPendingDatabaseChanges loop 791 - check for changes 522
19:13:49.613 CheckForPendingDatabaseChanges loop 792 - check for changes 523
```

As you can see, there is sometimes a significant delay between the moments when the database update pending event is raised and when it is actually forwarded by Revit to the external event Execute method, at which point the code switches back to check for new changes again.

In most cases, it was possible to trigger the forwarding simply by moving the cursor on the screen.

A similar effect is described and used by Joe Ye to implement the programmatic
[trick to trigger the Idling event](http://adndevblog.typepad.com/aec/2013/07/tricks-to-force-trigger-idling-event.html),
as briefly mentioned as a
[potential SetRaiseWithoutDelay workaround](http://thebuildingcoder.typepad.com/blog/2013/07/curve-length-idling-units-and-revitpythonshell.html#3).

That is obviously the next step I will explore to complete my understanding, perfect this application and make optimal use of the external event mechanism.

For now, though, I think this is enough for today.

#### Differences, Download and Conclusion

I hope this pretty exhaustive description is clear.

As you can see, I think this is a very important and interesting topic, so I really want to get the message across.

I committed the changes implementing this version in the
[RoomEditorApp GitHub repository](https://github.com/jeremytammik/RoomEditorApp)
([description](http://thebuildingcoder.typepad.com/blog/2013/10/roomeditorapp-for-revit-2014-on-github.html)) as
[release 2014.0.0.18](https://github.com/jeremytammik/RoomEditorApp/releases/tag/2014.0.0.18).

If you are interested in the exact differences between the versions using the Idling event versus the new external event, here is an
[edited version](zip/roomedit_extern_event_diff_edited.txt) and the
[full list of differences](zip/roomedit_extern_event_diff_original.txt) reported
by the git diff tool.

I hope you have fun with all this, and please wish me lots of luck for the DevDays conferences.

#### Addendum – Revit Requires Focus

Guy Robinson has some comments on this and says:

Here are a few more things to consider:

- External events won't fire if Revit doesn't have focus.
- Because they're managed by the OS if you try raising external events continuously the period grows increasingly large as it tries to ensure the external events aren't hogging the system.
  At least that's my explanation of the result.

Admittedly it's been a while since I was experimenting with this.
But I think the behaviour will be the same.
I found I had to keep using Idling because external events couldn't replicate the high frequency firing possible with Idling.

Many thanks to Guy for this valuable input!

It matches my findings.
I still hope to find a way to stick with external events, though, to avoid the Idling event hogging the system.

I also noticed that Revit needs focus to actually forward the external event Raise invocation to call its Execute method.
I am still hoping to find an easy clean way to trigger that programmatically from outside...