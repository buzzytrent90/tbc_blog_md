---
ai_optimized: true
author: Jeremy Tammik
complexity_score: 9.5
content_type: qa
optimization_date: '2025-12-11T11:44:14.334552'
original_url: https://thebuildingcoder.typepad.com/blog/0652_modeless_loose_connectors.html
post_number: '0652'
reading_time_minutes: 14
series: mep
slug: modeless_loose_connectors
source_file: 0652_modeless_loose_connectors.htm
tags:
- csharp
- doors
- elements
- references
- revit-api
- selection
- transactions
- windows
- mep
title: Yet Another Modeless Update
word_count: 2858
---

### Yet Another Modeless Update

Here is a rather exciting post for me that I spent quite some time on in the past few days.

I recently published an
[updated version for Revit 2012](http://thebuildingcoder.typepad.com/blog/2011/08/modeless-loose-connector-navigator-update.html)
of my
[Revit 2011 modeless loose connector navigator](http://thebuildingcoder.typepad.com/blog/2010/07/modeless-loose-connectors.html) and
fixed some flaws pointed out repeatedly by Arnošt Löbel at the same time.

Please refer to the original post for a full description of the application, since here I am focusing only on the modeless interaction with the Idling event.

The main fix I implemented was to instantiate an external application in addition to the original external command, and ensure that I subscribe only once to the Idling event, not every time the command is called.

I actually thought I had understood all the important aspect of modeless interaction with Revit, but was soon to discover how mistaken I was at the time.
Maybe there are some important lessons to be learned from the following analysis for you as well?

When writing a later note about the
[basics of interacting with Revit from an external context](http://thebuildingcoder.typepad.com/blog/2011/09/modeless-forms-in-revit.html),
I noticed a few other flaws remaining to be fixed as well.

Most pertinently, like any other call-back, Idling costs time and should be used sparingly.

I was of course already aware of this, but still, it prompted me to rethink and revisit my code once again, and subscribe to the Idling event only while the modeless form is actually being displayed.

The relevant changes that I ended up with in my first attempt were the following:

1. In the external application, do not subscribe to Idling on start-up, just keep a static pointer to the UIControlledApplication instance.- In the external command, subscribe to the form Load and FormClosing events.- In the form Load and FormClosing events, subscribe to and unsubscribe from the Idling event.

In my naivety, that seemed like a clean and pretty optimal solution to me, and also not overly complex.

Here are some lines of code illustrating this, first subscribing to the form events from the external command Execute method:
```csharp
  LooseConnectorNavigator navigator
    = new LooseConnectorNavigator(
      data,
      new SetElementId( SetPendingElementId ) );

  navigator.Load += new EventHandler(
    navigator\_Load );

  navigator.FormClosing
    += new FormClosingEventHandler(
      navigator\_FormClosing );

  navigator.Show( \_hWndRevit );
```

Within the form events, I subscribe to and unsubscribe from the application events:
```csharp
/// <summary>
/// The application subscribes to the Idling event
/// when the modeless loose connector navigator is
/// displayed.
/// </summary>
void navigator\_Load( object sender, EventArgs e )
{
  App.UIControlledApplication.Idling
    += new EventHandler<IdlingEventArgs>( OnIdling );
}

/// <summary>
/// The application unsubscribes from the Idling
/// event as soon as the modeless loose connector
/// navigator is closed.
/// </summary>
void navigator\_FormClosing(
  object sender,
  FormClosingEventArgs e )
{
  App.UIControlledApplication.Idling
    -= new EventHandler<IdlingEventArgs>( OnIdling );
}
```

**Don't do this!**

Here is the explanation why.

Luckily, I asked Arnošt for his opinion again, and he replied:

1. Static pointer to the UIControlledApplication instance: I do not think you need the pointer until your external command is called to be executed, if you need it at all (I do not think you do).- Subscribe to the form events: you need to subscribe to Idling from the external command Execution method. I do not care about the form events – those are all yours; do whatever you want in them **except calling Revit API**.- Subscribe to and unsubscribe from the Idling event in the form events: you cannot subscribe to or unsubscribe from Revit events from your internal form events. They mean nothing to Revit. You can only make the subscription when Revit calls your app – that is from OnStartup, command.Execute, event handlers (**Revit** event handlers), etc.

If I am correctly guessing what your goal is, the following should be the workflow:

- Do nothing in OnStartup- From your Command.Execute subscribe to Idling. The handler should be a method of your external application. Set up an internal Boolean to flag (in the app) that you have done this.- When the end user clicks a command (a control, I assume) in your dialog:
      - Set an internal variable (let's call it requestId) indicating that it happened.- Disable the controls in your dialog including the one the user just clicked (Cancel should still be available though).- In every OnIdling call, check:
        - If the dialog is no longer visible:
          - Unsubscribe from Idling.- Reset the Boolean flag you set in Command.Execute so you know you are not subscribed anymore (this could be done by other means, of course; I am just showing one possible way).
        - If the dialog is still visible:
          - If your requestId is not set, do nothing.- If your requestId has been set:
              - Make Revit to do what the user requested.- Clear the requestId variable.- Enable the controls in your dialog again.- From OnShutdown you should unsubscribe from Idling if you are still listening to it, and I think you should close your dialog if it has not been closed by the end user yet. I believe it is just a nice thing to do and I would implement it that way.

That feedback was quite a shock to me and took a while to digest.

It also led me to raise a couple of supplementary questions:

**Question:** Why do you think that I do not need the pointer to the UIControlledApplication instance?

Because I can use the Application property passed in by the command data instead?

Yes, that would simplify things a bit.

**Answer:** Yes, that is what I meant.
You do need to set it in OnStartup, because you get it later again in your Command.Execute and in the OnIdling as well (if you need it there).

**Question:** Ah yes, I see the difference between subscribing to the Idling event from within the command and within the form Load event.
That is something I should improve.
I made a mistake there.
And you are saying that I cannot even unsubscribe from Idling from the form events?

**Answer:** No, you cannot.
Your form events are completely unattached from Revit.
They happen on a different thread.
They are like any other methods running on a different thread (not the main thread on which Revit calls out).
I believe I've seen this mistake made several times.
I also
[mentioned some of these flaws](http://thebuildingcoder.typepad.com/blog/2011/01/modeless-door-lister-flaws.html) after
looking at the
[modeless door lister](http://thebuildingcoder.typepad.com/blog/2010/12/modeless-door-lister-and-deleter.html).

**Question:** This all seems a bit strange and extreme to me.
I mean, these are application events, and the application is there forever, as long as the session is active.
I understand that officially I cannot touch the Revit API outside of the specified Revit call-back notifications, but still...

**Answer:** Yes, but not our application (Revit).
It is your application – except for the time Revit explicitly calls it, it does not have anything to do in it (and that is mutual).

**Question:** I was under the assumption that subscribing to an event does not make use of the Revit API.
I thought it uses pure .NET functionality and that the application object is available for me to attach an event handler to it, with no use of the Revit API at all.

**Answer:** That is not correct.
All public events have their corresponding framework internally in Revit.
In order to subscribe or unsubscribe, something needs to be modified somewhere in Revit, which means allocations/reallocations are likely to be made.
That is not going to be safe in a multithreaded environment (I mean, not that it couldn't be in theory, but it might be not in Revit, currently).

**Question:** Initially I did not see the point of the additional complexity and strict separation between the form and Revit events that you require in your suggestion.
After working through these answers of yours, I reread you suggested new algorithm, and it makes complete sense.

**Answer:** Yes, it does sound a bit complex when you read it first, but like you learned, it is not as complex once you take time to think about it and especially once you start using it that way.
I believe the complexity is just due to the multi-threading nature, which is what any modeless interaction is essentially based upon.

**Question:** I still do not understand the importance of unsubscribing in OnShutdown, since Revit will shut down anyway at that point.
Is that for future-proofing, for some future situation in which possibly the add-in may be shut down but the Revit session remains active?

**Answer:** It is really not critical, but I consider it a nice thing to do and we do log unsubscribed events in journals.
You can look at it this way: while it is not essentially important to close all your open files (and other resources) in your application (Windows will close them when the process get killed), an application is considered well behaved if it does close everything, and I always do so.

**Question:** Does it makes any difference subscribing to the Idling event from the UIApplication class that I obtain from the Execute command data, or using a static variable initialised from the OnStartup UIControlledApplication instead?

**Answer:** It does matter from where you subscribe and it does not matter if you use UIApplication or UIControlledApplication.
Those two application objects share the event repository internally, so subscribing through one is technically identical to subscribing through the other one.
In your particular case, I recommended subscribing from your command, because that is when you start needing it, actually; you did not need it in OnStartup yet.

You should not need to store a pointer (a tracking handle in .NET, to be exact) to the application object (to neither one), because it should always be available to you when you are actually permitted to use it – you get it, directly or indirectly, in OnStartup, in command.execute, in Events, in updaters, etc; there is no need to store it anywhere.
If you think it simplifies you code, you can keep the UIControlledApplication variable that you initialised in OnStartup, but my implementation would be different (of course, all programmers have different habits, it does not mean that one is better).
I would pass the application as an argument to ShowForm, although it seems you do not need to call Subscribe from ShowForm.
It looks like you can call Subscribe from your Execute after ShowForm returned success.
I would pass the application object to both Subscribe and Unsubscribe as an argument. (It is simply my preferred way of using arguments over having global variables.)

**Question:** I will follow every single piece of your advice, except passing the arguments to the subscribe methods.
I also almost fanatically try to avoid global variables wherever possible.
I tried my very best to do that here as well, but it requires me to call two static methods on the App class instead of one from the command Execute method, and to create at least three different overloads of the Subscribe and Unsubscribe methods taking either UIApplication or UIControlledApplication arguments.
It also forces the application Subscribe method called from the command to be public instead of private.
All in all, that turns out to be just too much.

**Answer:** Your code is good.
I did not realize the need for overloading Unsubscribe – I think you have a point there.

In the end, I improved my implementation to follow Arnošt's advice as follows:

- Moved the OnIdling handler from the external command to the external application.- Moved the pending element id handling to the external application too.- Implemented a static singleton LooseConnectorNavigator instance.- Implemented a static Show method to initialise it in the LooseConnectorNavigator class.- Subscribe to the FormClosing event to set it to null when the modeless form is closed.- Implemented a static IsShowing predicate in the LooseConnectorNavigator class to check whether the modeless form is displayed.- Subscribe to the Idling event from the external command Execute method.- Unsubscribe from the Idling event in the OnIdling event if the modeless dialog has been closed.- Ensure that I am unsubscribed from the Idling event and the modeless form is closed in the application OnShutdown handler.

Once this is done, there is actually no reason any longer for the command to communicate with the modeless form anymore.
It is better to limit the knowledge of and connection between the various classes as much as possible, so I removed all references of the modeless form from the command class to the application class.
This means that I also:

- Moved the static Revit window handle variable and its initialisation to the external application and its OnStartup method.- Implemented three methods in the application class to show the modeless form, subscribe to and unsubscribe from the Idling event.

The OnStartup method also stores a pointer to the UIControlledApplication instance in a static variable, so that the UIApplication instance does not need to be passed in from the external command when needed:
```csharp
#region Revit window handle
/// <summary>
/// Revit application window handle, used
/// as parent for modeless dialogue.
/// </summary>
static JtWindowHandle \_hWndRevit = null;
#endregion // Revit window handle

static UIControlledApplication \_a;

public Result OnStartup( UIControlledApplication a )
{
  ProductType pt = a.ControlledApplication.Product;

  if( ProductType.MEP == pt )
  {
    #region Revit window handle
    // Set up IWin32Window instance encapsulating
    // main Revit application window handle:

    Process process = Process.GetCurrentProcess();

    IntPtr h = process.MainWindowHandle;

    \_hWndRevit = new JtWindowHandle( h );
    #endregion // Revit window handle

    \_a = a;

    return Result.Succeeded;
  }
  else
  {
    return Result.Cancelled;
  }
}
```

The external command implementation is vastly simplified and disconnected from both the modeless form and the Idling event handling.
It no longer even worries about the Revit window handle needed to display the form properly associated with the Revit main window.
All it does now is call one single static method ShowForm on the application class before terminating to pass in the current data to display in the form:
```csharp
  // Cleaned up all the complexity and encapsulated
  // it in one single call to the external application:

  App.ShowForm( data );

  return Result.Succeeded;
```

The ShowForm method handles and hides all the complexities of managing the form and the subscription to the Idling event:
```csharp
public static void ShowForm(
  SortableBindingList<ConnectorData> data )
{
  LooseConnectorNavigator.Show( data,
    new SetElementId( SetPendingElementId ),
    \_hWndRevit );

  Subscribe();
}
```

The methods to subscribe and unsubscribe can even remain private to the application class and ensure that we only subscribe once:
```csharp
/// <summary>
/// Keep track of our subscription status.
/// </summary>
static bool \_subscribing = false;

/// <summary>
/// Subscribe to the Idling event if not yet already done.
/// </summary>
static void Subscribe()
{
  if( !\_subscribing )
  {
    \_a.Idling
      += new EventHandler<IdlingEventArgs>(
        OnIdling );

    \_subscribing = true;
  }
}

/// <summary>
/// Unsubscribe from the Idling event
/// if we are currently subscribed.
/// </summary>
static void Unsubscribe()
{
  if( \_subscribing )
  {
    \_a.Idling
      -= new EventHandler<IdlingEventArgs>(
        OnIdling );

    \_subscribing = false;
  }
}
```

Except for the additional check to unsubscribe from the Idling event if the modeless form has been closed, the Idling event handler is unchanged from the previous implementation:
```csharp
/// <summary>
/// Revit Idling event handler.
/// Whenever the user has selected an element to
/// zoom to in the modeless dialogue, the pending
/// element id is set. The event handler picks it
/// up and zooms to it. We are not modifying the
/// Revit document, so it seems we can get away
/// with not starting a transaction.
/// </summary>
public static void OnIdling(
  object sender,
  IdlingEventArgs ea )
{
  if( !LooseConnectorNavigator.IsShowing )
  {
    Unsubscribe();
  }

  int id = \_pending\_element\_id;

  if( 0 != id )
  {
    // Support both 2011, where sender is an
    // Application instance, and 2012, where
    // it is a UIApplication instance:

    UIApplication uiapp
      = sender is UIApplication
        ? sender as UIApplication
        : new UIApplication(
          sender as Application );

    UIDocument uidoc
      = uiapp.ActiveUIDocument;

    Document doc
      = uidoc.Document;

    ElementId eid = new ElementId( id );
    Element e = doc.get\_Element( eid );

    Debug.Print(
      "Element id {0} requested --> {1}",
      id, new ElementData( e, doc ) );

    // Look, mom, no transaction required!

    uidoc.Selection.Elements.Clear();
    uidoc.Selection.Elements.Add( e );
    uidoc.ShowElements( e );

    \_pending\_element\_id = 0;
  }
}
```

Finally, in the OnShutdown method, we make sure that we both close the modeless form, if it is open, and unsubscribe from the Idling event, if we are subscribed:
```csharp
public Result OnShutdown( UIControlledApplication a )
{
  LooseConnectorNavigator.Shutdown();

  Unsubscribe();

  return Result.Succeeded;
}
```

The code itself did not change that much from my original implementation.
Most of these changes just involved juggling existing snippets around to satisfy the recommendations listed above, and adding some trivial logic.

As far as I can tell, it all still works, and it feels better than ever before :-)

I think this has been nagging at me subconsciously ever since I did not dive in deep into the
[door lister flaws](http://thebuildingcoder.typepad.com/blog/2011/01/modeless-door-lister-flaws.html) highlighted
by Arnošt, so it is a real relief to have it finally sorted out now.

For comparison purposes, here are two separate downloads,
<loose_connectors_9.zip> containing
the flawed version 2012.0.0.9 from my initial attempt before receiving and finally understanding the additional advice, and
<loose_connectors_11.zip> containing
the fixed and hopefully final version 2012.0.0.11.
Don't mix them up, please!