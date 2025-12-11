---
ai_optimized: true
author: Jeremy Tammik
complexity_score: 6.5
content_type: qa
optimization_date: '2025-12-11T11:44:14.502886'
original_url: https://thebuildingcoder.typepad.com/blog/0743_external_event.html
post_number: '0743'
reading_time_minutes: 7
series: general
slug: external_event
source_file: 0743_external_event.htm
tags:
- csharp
- elements
- revit-api
- transactions
- views
title: Idling Enhancements and External Events
word_count: 1448
---

### Idling Enhancements and External Events

Revit 2013 has been released, and I presented an overview of the
[Revit 2013 API](http://thebuildingcoder.typepad.com/blog/2012/03/revit-2013-and-its-api.html) and the
[new SDK samples](http://thebuildingcoder.typepad.com/blog/2012/03/new-revit-2013-sdk-samples.html) last week.

One of the areas of enhancement is the
[Idling event](http://thebuildingcoder.typepad.com/blog/idling) and
interaction with modeless dialogues, which also was the subject of Arnošt Löbel's Autodesk University 2011 class
[CP5381](http://au.autodesk.com/?nd=event_class&session_id=9879&jid=1763185) on
asynchronous interactions and managing modeless UI.
As I mentioned, the enhancements and new functionality are demonstrated by the new
[ModelessForm\_ExternalEvent and ModelessForm\_IdlingEvent](http://thebuildingcoder.typepad.com/blog/2012/03/new-revit-2013-sdk-samples.html#1) SDK
samples.

Here is an overview of these enhancements from the recent ADN newsletter:

#### Overview

A number of features introduced in the Revit 2012 API have been enhanced in the Revit 2013 API based on the initial real-world experience gathered since the initial release of Revit 2012. For instance, this has led to the following added features related to the Idling and other events:

- Control of the [Idling event repetition](#2)- Idling event with [no active document](#3)- New [external events framework](#4)- [RevitWebcam revisited](#5)- Use of [OpenAndActivateDocument in event handlers](#6)- [Further information](#7)

#### Idling Event Repetition

The Idling event has been changed to offer two different modes of behaviour. The default behaviour has changed, so pre-existing code that needs the original behaviour will have to explicitly select the non-default behaviour.

In the **default mode**, the event will be raised one single time each time Revit begins an idle session. Note that when the user is active in the Revit user interface, idle sessions begin whenever the mouse stops moving for a moment or when a command completes. However, if the user is not active in the user interface at all, Revit may not invoke additional idling sessions for quite some time; this means that an add-in may not be able to take advantage of time when the user leaves the machine completely idle for a period of time.

In the **non-default mode**, Revit keeps the idling session open and makes repeated calls to the event subscriber. In this mode, Revit will continue to make Idling calls even if the user is totally inactive. However, this can result in system performance degradation, because the CPU may remain fully engaged in serving Idling events.

You can specify the non-default Idling behaviour by calling the new **SetRaiseWithoutDelay** method on the IdlingEventArgs instance passed in to the event handler. This call has to be made each time the Idling event call-back is made. Revit will revert to the default single-call Idling behaviour as soon as this method is not called.

#### Idling Event with no Active Document

In contrast to the Revit 2012 behaviour, the Idling event now is invoked even when there is no document active in Revit.

#### External Events Framework

The Idling event enables an interaction between Revit and an external asynchronous process. The external process cannot directly invoke any Revit API methods, but it can send a request to be processed by the Idling event handler next time it is raised. One common use of this has turned out to be supporting the interaction of a modeless dialogue with Revit. In this kind of scenario, the Idling event handler never has anything to do except when such a request has been submitted, so most of the calls to it will simply return immediately.

This interaction is supported much simpler and much more efficiently by the new external events framework. It operates similarly to the Idling event with default frequency. Instead of implementing an event handler and setting up the handler to do nothing when there is no activity in your dialog, you do the following instead:

- Implement an external event, derived from the IExternalEvent interface.- Create an instance of your external event by calling the ExternalEvent.Create method.- When the external asynchronous process triggers an event that requires a Revit action to be taken, call the Raise method on the external event instance.- Revit will wait for an available Idling time cycle and call the external event Execute method.

This framework is useful for modeless dialogue developers, because you can skip the default no-op implementation when nothing has happened, and just tell Revit when there is some work to do. It saves on the time required for Revit to make repeated calls to the Idling event when there is nothing to be done.

This is the complete specification of the external event interface:
```csharp
  // An interface to be executed when an external
  // event is raised.
  //
  // An instance of a class implementing this
  // interface will be registered with Revit first,
  // and every time the corresponding external event
  // is raised, the Execute method of this interface
  // will be invoked.

  public interface IExternalEventHandler
  {
    // Method called to handle the external event.

    void Execute( UIApplication app );

    // String identification of the event handler.

    string GetName();
  }
```

Here is an example snippet of code creating an instance of an external event and starting a new thread to drive it from the same application:
```csharp
  \_event = ExternalEvent.Create(
    new WebcamEventHandler() );

  Thread thread = new Thread(
    new ThreadStart( Run ) );

  thread.Start();
```

Here is part of the implementation of the Run method making use of the external event and calling its Raise method to request its Execute method to be called, in which the Revit API can be used again:
```csharp
    /// <summary>
    /// External webcam event driver.
    /// Check regularly whether the webcam image has
    /// been updated. If so, update the spatial field
    /// primitive with the new image data.
    /// Currently, we only display a grey scale image.
    /// Colour images could be handled as well by
    /// defining a custom colour palette.
    /// </summary>
    static void Run()
    {
      \_running = true;

      while( \_running )
      {
        \_data = new GreyscaleBitmapData(
          \_width, \_height, \_url );

        byte[] hash = \_data.HashValue;

        if( null == \_lastHash
          || 0 != CompareBytes( hash, \_lastHash ) )
        {
          \_lastHash = hash;
          \_event.Raise();
        }
        Thread.Sleep( \_intervalMs );
      }
    }
```

#### RevitWebcam Revisited

I implemented the
[RevitWebcam](http://thebuildingcoder.typepad.com/blog/2010/06/display-webcam-image-on-building-element-face.html) sample
created to demonstrate use of the Idling event when it was introduced in the Revit 2011 API, and recently
[migrated it to Revit 2012](http://thebuildingcoder.typepad.com/blog/2012/02/revit-webcam-2012.html) in
preparation for making use of this new 2013 functionality.

I now performed two further steps on it:

1. Enhance it with the new Idling functionality, and- Convert it to make use of an external event instead of the Idling event.

Here is
[RevitWebcam\_2013.zip](zip/RevitWebcam_2013.zip) containing
the following hopefully self-descriptive subfolders, each of which in turn contains a complete Visual Studio solution:

- RevitWebcam2011- RevitWebcam2012\_1\_initial\_migration- RevitWebcam2012\_2\_removed\_warnings- RevitWebcam2012\_3\_final\_cleanup- RevitWebcam2013\_1\_initial\_migration- RevitWebcam2013\_2\_removed\_warnings- RevitWebcam2013\_3\_idling\_repetition- RevitWebcam2013\_4\_external\_event- RevitWebcam2013\_4\_external\_event\_2

If you are interested in the evolution of the project, you can compare these with each other.

Please be aware that these projects are for demonstration purposes only and skip a lot of error checking.
For instance, they do nothing protect themselves against the user switching or closing the active project or making other changed to the environment while they are running, so they should not be used as examples of real world application implementations.

The
[ModelessForm\_ExternalEvent and ModelessForm\_IdlingEvent](http://thebuildingcoder.typepad.com/blog/2012/03/new-revit-2013-sdk-samples.html#1) SDK
samples are probably a bit simpler and more safely built (by Arnošt) to handle these kind of issues :-)

#### Calling OpenAndActivateDocument during Events

In the Revit 2012 API, the OpenAndActivateDocument method cannot be called from inside any event handler at all. Related to the new Idling and external event functionality, it may now be invoked from within event handlers under most circumstances, specifically under the following conditions:

- From an external event if the original active document has no open transactions or transaction groups.- From any regular event handler including Idling and the new ApplicationInitialized event if
    - There is no active document open yet in Revit, and- The event is not nested in any other event or in the execution of an external command.

In general, the new ApplicationInitialized event is more suitable than the Idling one for opening and activating the first document.

#### Further Information

All the information above and more is provided in the What's New section of the Revit API help file RevitAPI.chm included with the Revit SDK.

Download Revit 2013 today and see for yourself!