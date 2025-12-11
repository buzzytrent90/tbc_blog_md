---
ai_optimized: true
author: Jeremy Tammik
complexity_score: 6.7
content_type: code_example
optimization_date: '2025-12-11T11:44:15.692127'
original_url: https://thebuildingcoder.typepad.com/blog/1290_auto_open_project.html
post_number: '1290'
reading_time_minutes: 10
series: general
slug: auto_open_project
source_file: 1290_auto_open_project.htm
tags:
- elements
- family
- geometry
- parameters
- python
- revit-api
- vbnet
- walls
title: Automatically Open a Project on Startup
word_count: 1980
---

### Automatically Open a Project on Startup

Yesterday, I talked about the interesting activity, numerous answers and my participation in the
[Revit API discussion forum](http://forums.autodesk.com/t5/revit-api/bd-p/160) and
edited the thread and GitHub solution for
[opening and activating a Revit document in an event handler](http://thebuildingcoder.typepad.com/blog/2015/03/opening-and-activating-document-in-an-event-handler.html).

Today, we look at the related issue of
[loading a Revit RVT project file automatically on start-up](http://forums.autodesk.com/t5/revit-api/how-to-load-a-rvt-file-automatically-when-start-the-revit/m-p/5500878).

Before getting into that, here is a quick note on the deprecated SolidArray class:

#### SolidArray Removal

In the long distant past, the Revit API defined a considerable number of custom collection classes, many of them equipped with their own custom iterators and loads of other stuff.

We have been in the process of removing these custom collection classes for several years now.

They are being systematically replaced by generic .NET collection classes.

Some vestiges still remain, though.

One of them is the SolidArray.

The Revit 2015 API still defines the SolidArray class, and an example of using it is shown in the Element.Geometry property description sample code snippet.

However, this class is not used anywhere at all in the Revit API any longer.

Just like all other custom collection classes, it should be replaced by a generic .NET collection instead, e.g. `List<Autodesk.Revit.DB.Solid>`, since it will disappear in the near future.

Yet another little step to take in the efforts to
[future-proof your add-in](http://thebuildingcoder.typepad.com/blog/2014/11/determining-intersecting-elements-and-continued-futureproofing.html).

#### Loading a RVT Project File Automatically on Start-up

Let us continue with the main topic at hand, yet again based on a thread raised in the discussion forum and with a solution posted to GitHub, on
[loading a Revit RVT project file automatically on start-up](http://forums.autodesk.com/t5/revit-api/how-to-load-a-rvt-file-automatically-when-start-the-revit/m-p/5500878):

Yesterday's solution for
[opening and activating a Revit document in an event handler](http://thebuildingcoder.typepad.com/blog/2015/03/opening-and-activating-document-in-an-event-handler.html) made
use of an external event driven by VB.NET and XAML code, demonstrated by the
[VB.NET RevitOpenAndActivateDuringIdle sample on GitHub](https://github.com/LiFeleSs/RevitOpenAndActivateDuringIdle).

The solution to load a file on startup is simpler still, making use of the ApplicationInitialized event.

It took some iterations to arrive at that conclusion, though, and a few more to nail down the final solution.

I actually already published the beginning of this thread and the
[initial answers](http://thebuildingcoder.typepad.com/blog/2015/02/family-instance-area-and-auto-loading-a-project-file.html#3) by
Arnošt and me a month ago, but there was much more and the real solution to come later, so I need to pick it up again where I left off back then...

**Question:** I wonder how can I load a Revit file automatically when starting the Revit system.

Since the UIControlledApplication is the only parameter of the OnStartup function of the IExternalApplication class, while the OpenAndActivateDocument is the function of the UIApplication but not of the UIControlledApplication.

How can I invoke the UIApplication.OpenAndActivateDocument method in the OnStartup function? Or is there some other method to address the problem?

**Answer:** The UIControlledApplication controlled application provided in the OnStartup method cannot open a document, because Revit is still in the process of starting up and is not ready to open and process project files yet.

In the OnStartup method, you can set up a system to open the project document later on, when Revit has finished loading.

Before getting to that, though, let me point out that the easiest way to start up Revit with a project file loaded is to use
[Process.Start](http://thebuildingcoder.typepad.com/blog/2010/03/using-processstart-to-open-a-project-or-family.html).

Once Revit is up and running, as you have pointed out yourself, you can also use the UIApplication.OpenAndActivateDocument method, as demonstrated in this discussion on
[closing the active document](http://thebuildingcoder.typepad.com/blog/2012/12/closing-the-active-document.html).

You can prepare a call to this method from within the OnStartup method by temporarily subscribing to the Idling event in the OnStartup method, then unsubscribing from Idling in the Idling event handler, opening the document and doing whatever else you want at that point.

That should give you a couple of useful options to choose from.

**Answer** by Arnošt Löbel: The Revit API now provides another solution for the problem besides using the Idling event.
Three years ago, the Idling event may have been the only work-around, but there is a better solution now: a better way of opening a document on start-up is via the ApplicationInitialized event.
Like with the Idling event, an application subscribes to it during the OnStartup call.
Revit will raise this event a bit later, when the Revit application start-up has completed.
Remember, just as with the Idling event, the OpenAndActivateDocument method can be used only as long there is no active document in Revit yet.

**Response:**
Thank you very much for your prompt and detailed solution.

However, when I tested the System.Diagnostics.Process.Start method, the Revit file can be opened when start the Revit but disappear only straight away.

And I don't know how to test the Idling event and how to use the UIApplication.OpenAndActivateDocument method in OnStartup since I have no access to the UIApplication there.

**Answer** by Arnošt:
As I tried to explain but failed, apparently, please do not use the Idling event. Use the ApplicationInitialized event instead.
Subscribe to it during the OnStartup method of your external application, then from within the event's handler invoke the OpenAndActivateDocument method.

**Response:**
Thx! I'll try this solution...

When I used the event ApplicationInitialized and the System.Diagnostics.Process.Start in the event handler, it works now.

Thx for ur kindness direction!!

**Question:** by Matt:
I'm not sure if I understand your suggestion to use the ApplicationInitialized event.
While I agree that event is the right one to use for the timing of when it executes, it doesn't seem like we have access to the right contexts at the right time?

- If we register it during startup, we're registering it against the ControlledApplication class.
- When we are in the callback after, we're in the callback of the ControlledApplication class.

When we're in that callback – how would we have access to the UIApplication object so that we could call OpenAndActivate?

**Answer** by Arnošt:
I may be wrong, but I think it works differently than what you guessed. The object inside the OnStartup is indeed a ControlledApplication, but it ought to be an Application instance as the sender argument of the ApplicationInitialized event handler. With that you will be able to construct an instance of UIApplication and use its methods. (Unfortunately, most of us New Englanders are working from their homes today due to a storm, and I am trying to limit my debugging sessions to a minimum! If it turns out it is not working for you as I suggested, however, let me know and ’ll debug it myself by tomorrow or Wednesday.)

Just as a side note: it does not matter whether an event handler subscribes to an event using ControlledApplication or Application. The repository of application event delegates are always with the Application instance (internally).

**Answer:**
You make it sound so easy...

Here we are, an hour or two later:

```python
class App : IExternalApplication
{
  const string \_test\_project\_filepath
    = "Z:/a/rvt/CurvedWall.rvt";

  UIApplication \_uiapp;

  public Result OnStartup( UIControlledApplication a )
  {
    #region Retrieving UIApplication from UIControlledApplication
    // These attempts to access a UIApplication
    // or Application instance are all in vain:
    //
    //\_uiapp = (UIApplication) a;
    //\_uiapp = (UIApplication) a.ControlledApplication;
    //Application app = (Application) a;
    //Application app2 = (Application) a.ControlledApplication;
    //Application app3 = a.m\_application;

    // Using Reflection works, though:

    Type type = a.GetType();

    // Not useful in this case, but interesting:

    MemberInfo[] publicMembers = type.GetMembers();
    MemberInfo[] nonPublicMembers = type.GetMembers( BindingFlags.NonPublic );
    MemberInfo[] staticMembers = type.GetMembers( BindingFlags.Static );

    // This is the call that finally yields useful results:

    BindingFlags flags = BindingFlags.Public
      | BindingFlags.NonPublic
      | BindingFlags.GetProperty
      | BindingFlags.Instance;

    MemberInfo[] propertyMembers = type.GetMembers(
      flags );

    // Note that the field "m\_application" is listed
    // in the propertyMembers array, and also the
    // method "getUIApp"... let's grab the field:

    string propertyName = "m\_application";
    flags = BindingFlags.Public | BindingFlags.NonPublic
      | BindingFlags.GetField | BindingFlags.Instance;
    Binder binder = null;
    object[] args = null;

    object result = type.InvokeMember(
        propertyName, flags, binder, a, args );

    UIApplication \_uiapp;

    \_uiapp = (UIApplication) result;
    #endregion // Retrieving UIApplication from UIControlledApplication

    \_uiapp.ApplicationInitialized
      += OnApplicationInitialized;

    return Result.Succeeded;
  }

  void OnApplicationInitialized(
    object sender,
    ApplicationInitializedEventArgs e )
  {
    // This does not work, because the sender is
    // an Application instance, not UIApplication.

    //UIApplication uiapp = sender as UIApplication;

    // Sender is an Application instance:

    Application app = sender as Application;

    // However, UIApplication can be
    // instantiated from Application.

    UIApplication uiapp = new UIApplication( app );

    uiapp.OpenAndActivateDocument(
      \_test\_project\_filepath );
  }

  public Result OnShutdown( UIControlledApplication a )
  {
    return Result.Succeeded;
  }
}
```

It works!

Is that what you intended?

**Response:**
Thanks!
You are correct; the sender of the ApplicationInitialized event callback is the Application (not the controlled application).
So my quest of a non-hacky way to transition between the ControlledApplication and the Application is finally solved!

Jeremy – I'm not sure that the reflection was actually necessary? Was that just exploration?

**Answer** by Arnošt:
Perhaps I made it sound simple because it is in fact rather simple. Indeed, as Matt already noted, no reflection is necessary.
In the OnStartup you get UIControlledApplication, from which you can get ControlledApplication (the latter always exists if the former does), and on that you subscribe to the ApplicationInitialized.
Keep in mind you cannot cast a ControlledApplication to Application – that is the ControlledApplication’s very point of existence.

Then in the event handler the sender should be an instance of Application, which can be used to instantiate a UIApplication, on which the OpenAndActivateDocument method can be invoked. Like I said, quite simple. ;-) Each of the two methods is only a few lines of code.

**Answer:**
Oh dear. What a waste.

Yes, of course, given the UIApplication constructor taking an Application argument, all is clear and the Reflection is not needed. I did not notice that, and what I described was the first alternative I could find :-)

Well, well, apparently all roads lead to Rome after all... but it saves effort taking the easiest path.

For the sake of completeness and those that trust nothing but source code, here is the complete solution without using Reflection, showing just how short and simple it can be:

```python
class App : IExternalApplication
{
  const string \_test\_project\_filepath
    = "Z:/a/rvt/CurvedWall.rvt";

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
    // Sender is an Application instance:

    Application app = sender as Application;

    // However, UIApplication can be
    // instantiated from Application.

    UIApplication uiapp = new UIApplication( app );

    uiapp.OpenAndActivateDocument(
      \_test\_project\_filepath );
  }

  public Result OnShutdown( UIControlledApplication a )
  {
    return Result.Succeeded;
  }
}
```

The complete source code, Visual Studio solution and add-in manifest are provided in the
[OpenProject GitHub repository](https://github.com/jeremytammik/OpenProject),
and the version described here is
[release 2015.0.0.0](https://github.com/jeremytammik/OpenProject/releases/tag/2015.0.0.0).

It could also be noted, for the sake of completeness, that in the majority of application events, the sender is the Application object.
There are only a handful (if at all that much) of events that are raised by the UIApplication.
Those are more like exceptions to the rule, as documented in the Revit API help file RevitAPI.chm in the SDK.