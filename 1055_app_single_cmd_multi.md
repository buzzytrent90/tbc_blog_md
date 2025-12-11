---
ai_optimized: true
author: Jeremy Tammik
complexity_score: 5.5
content_type: code_example
optimization_date: '2025-12-11T11:44:15.208368'
original_url: https://thebuildingcoder.typepad.com/blog/1055_app_single_cmd_multi.html
post_number: '1055'
reading_time_minutes: 6
series: general
slug: app_single_cmd_multi
source_file: 1055_app_single_cmd_multi.htm
tags:
- elements
- python
- references
- revit-api
- rooms
- transactions
- windows
title: Singleton Application versus Multiple Command Instances
word_count: 1152
---

### Singleton Application versus Multiple Command Instances

I recently discussed the
[RoomEditorApp](http://thebuildingcoder.typepad.com/blog/2013/11/roomeditorapp-architecture-and-external-application.html) external
application, the Revit add-in part of my cloud-based real-time round-trip 2D Revit model editing application, while creating the
[RoomEditorApp GitHub repository](https://github.com/jeremytammik/RoomEditorApp) and
migrating it to Revit 2014.

I also mentioned that I was mysteriously
[unable to unsubscribe from the Idling event](http://thebuildingcoder.typepad.com/blog/2013/11/roomeditorapp-architecture-and-external-application.html#4).

I raised this [mystifying question](#2) with the development team and received a very [satisfying answer](#3), which includes some important information on how external applications and commands are handled and instantiated by the Revit API.

#### Unsubscribing from the Idling Event Fails

I created a simplified version of the external application named TestIdlingUnsubscribe to demonstrate how my attempt to unsubscribe from the Idling event fails.

It consists of an external application handling the subscription and an external command to toggle the subscription on and off.

The application defines a public method ToggleSubscription providing access to that functionality like this:

```python
class App : IExternalApplication
{
  /// <summary>
  /// Are we currently subscribed to the Idling event?
  /// </summary>
  public static bool Subscribed = false;

  /// <summary>
  /// Our one and only Revit-provided
  /// UIControlledApplication instance.
  /// </summary>
  static UIControlledApplication \_uiapp;

  /// <summary>
  /// Toggle on and off subscription to
  /// automatic cloud updates.
  /// </summary>
  public static void ToggleSubscription(
    EventHandler<IdlingEventArgs> handler )
  {
    if( Subscribed )
    {
      Debug.Print( "Unsubscribing..." );
      \_uiapp.Idling -= handler;
      Subscribed = false;
      Debug.Print( "Unsubscribed." );
    }
    else
    {
      Debug.Print( "Subscribing..." );
      \_uiapp.Idling += handler;
      Subscribed = true;
      Debug.Print( "Subscribed." );
    }
  }

  public Result OnStartup( UIControlledApplication a )
  {
    \_uiapp = a;
    return Result.Succeeded;
  }

  public Result OnShutdown( UIControlledApplication a )
  {
    if( Subscribed )
    {
      \_uiapp.Idling
        -= new EventHandler<IdlingEventArgs>(
          ( sender, ea ) => { } );
    }
    return Result.Succeeded;
  }
}
```

The external command defines the Idling event handler, reports the current status and makes calls to the App ToggleSubscription method:

```python
[Transaction( TransactionMode.ReadOnly )]
public class Command : IExternalCommand
{
  /// <summary>
  /// How many Idling calls to wait before reporting
  /// </summary>
  const int \_message\_interval = 100;

  /// <summary>
  /// Number of Idling calls received in this session
  /// </summary>
  static int \_counter = 0;

  void OnIdling(
    object sender,
    IdlingEventArgs ea )
  {
    ++\_counter;

    if( 0 == ( \_counter % \_message\_interval ) )
    {
      Debug.Print( "{0} OnIdling called {1} times",
        DateTime.Now.ToString( "HH:mm:ss.fff" ),
        \_counter );
    }

    ea.SetRaiseWithoutDelay();
  }

  public Result Execute(
    ExternalCommandData commandData,
    ref string message,
    ElementSet elements )
  {
    Debug.Print(
      "We are currently {0}subscribed to the Idling event.",
      App.Subscribed ? "" : "not " );

    App.ToggleSubscription( OnIdling );

    return Result.Succeeded;
  }
}
```

You can see how the command logs its activity and the Idling event handler logs the calls it receives to the Visual Studio debug output console window.

Here is a partial log of the command being executed twice, once to subscribe and once to unsubscribe:

```
  We are currently not subscribed to the Idling event.
  Subscribing...
  Subscribed.
  16:58:39.816 OnIdling called 100 times
  16:58:39.836 OnIdling called 200 times
  16:58:40.662 OnIdling called 300 times
  16:58:40.684 OnIdling called 400 times
  We are currently subscribed to the Idling event.
  Unsubscribing...
  Unsubscribed.
  16:58:42.230 OnIdling called 500 times
  16:58:42.250 OnIdling called 600 times
  16:58:42.270 OnIdling called 700 times
  ...
  16:58:44.465 OnIdling called 6200 times
```

Very strange!

The Idling event continues receiving calls even after unsubscribing.

The entire source code, Visual Studio solution and add-in manifest for this Revit add-in is published in the
[TestIdlingUnsubscribe GitHub repository](https://github.com/jeremytammik/TestIdlingUnsubscribe),
and the failing version described above is released as
[version 2014.0.0.0](https://github.com/jeremytammik/TestIdlingUnsubscribe/releases/tag/2014.0.0.0).

How can I unsubscribe successfully from the Idling event, and why is it failing?

#### Successful Unsubscription and Instantiation Details of the External Application and External Command

I discussed this with the Revit API development team, represented by Scott Conover, who replied:

What an interesting issue.

The good news – it has nothing to do with your machine configuration, using Parallels etc.

More good news – it is not related to any issues in the Revit API.

When a standalone ExternalCommand is added to Revit, Revit assumes that the command is entirely self-contained.
When the command is invoked – Revit instantiates an instance of the class implementing IExternalCommand, calls execute, and discards the reference.

Subsequent calls to the same command do the same thing – instantiate, invoke, discard.

Why it matters in your case – the Idling event handler you use is an instance method of the IExternalCommand class.
So the first invocation subscribes and it begins to run.
The second invocation instantiates a new IExternalCommand and attempts to unsubscribe – but to .NET this is a different method than was used to subscribe so the original keeps going.

To fix it, either:

- Make the event handler static.
- Hold your own reference to the command or other instance class that contains the handler, and use it to unsubscribe.

Note that Revit does not do this for ExternalApps, another reason to use them over ExternalCommands.
The app instance created to run OnStartup is held by Revit for event invocation of OnShutdown.
So another approach would be to use an instance method of the app (but this probably applies more to events which are not Idling, which should not remain active for the entire session anyway).

Many thanks to Scott for this important clarification.

I implemented a variation of the second approach he suggests, by storing the event handler method delegate that I use to subscribe with directly in a static variable in the external application.

I use that to redefine the Subscribe variable as a read-only property as well like this:

```python
class App : IExternalApplication
{
  /// <summary>
  /// Store the Idling event handler when subscribed.
  /// </summary>
  static EventHandler<IdlingEventArgs> \_handler = null;

  /// <summary>
  /// Are we currently subscribed to the Idling event?
  /// </summary>
  public static bool Subscribed
  {
    get
    {
      return null != \_handler;
    }
  }

  /// <summary>
  /// Our one and only Revit-provided
  /// UIControlledApplication instance.
  /// </summary>
  static UIControlledApplication \_uiapp;

  /// <summary>
  /// Toggle on and off subscription to
  /// automatic cloud updates.
  /// </summary>
  public static void ToggleSubscription(
    EventHandler<IdlingEventArgs> handler )
  {
    if( Subscribed )
    {
      Debug.Print( "Unsubscribing..." );
      \_uiapp.Idling -= \_handler;
      \_handler = null;
      Debug.Print( "Unsubscribed." );
    }
    else
    {
      Debug.Print( "Subscribing..." );
      \_uiapp.Idling += handler;
      \_handler = handler;
      Debug.Print( "Subscribed." );
    }
  }

  public Result OnStartup( UIControlledApplication a )
  {
    \_uiapp = a;
    return Result.Succeeded;
  }

  public Result OnShutdown( UIControlledApplication a )
  {
    if( Subscribed )
    {
      \_uiapp.Idling -= \_handler;
    }
    return Result.Succeeded;
  }
}
```

Since the external application is only instantiated as a singleton instance, I could basically remove all those 'static' qualifiers, but that would require an application instance variable for the external command to access them, so I left them in for the moment.

Note that the client code remains completely unaffected by this internal improvement, so the external command implementation is not changed at all.

The working solution above is provided in the repository as
[release 2014.0.0.1](https://github.com/jeremytammik/TestIdlingUnsubscribe/releases/tag/2014.0.0.1).

I hope you find this as interesting and illuminating as I do.