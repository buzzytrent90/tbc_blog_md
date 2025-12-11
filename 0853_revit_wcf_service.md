---
ai_optimized: true
author: Jeremy Tammik
complexity_score: 5.0
content_type: code_example
optimization_date: '2025-12-11T11:44:14.738896'
original_url: https://thebuildingcoder.typepad.com/blog/0853_revit_wcf_service.html
post_number: 0853
reading_time_minutes: 6
series: general
slug: revit_wcf_service
source_file: 0853_revit_wcf_service.htm
tags:
- csharp
- family
- python
- revit-api
- views
- walls
title: Drive Revit through a WCF Service
word_count: 1117
---

﻿

### Drive Revit through a WCF Service

Today, a special titbit for you:

How can I access Revit functionality from an external process?

As we have underlined many times in the past, the Revit API does not support any kind of
[asynchronous access](http://thebuildingcoder.typepad.com/blog/2010/04/asynchronous-api-calls-and-idling.html).

The
[Idling event](http://thebuildingcoder.typepad.com/blog/idling),
however, provides a possibility to work around this limitation.

Daren Thomas, the author or
[RevitPythonShell](http://code.google.com/p/revitpythonshell),
even presented a neat and clean
[pattern for semi asynchronous Idling API access](http://thebuildingcoder.typepad.com/blog/2010/11/pattern-for-semi-asynchronous-idling-api-access.html).

Now Victor Chekalin, aka Виктор Чекалин, presents another very neat sample providing such an access.
He says:

I have an idea how to access Revit from an external application.

The only official way to achieve this is through the Idling event and external events, of course.

The main idea:

- Inside Revit I create a service which receives queries from a stand-alone external process.- The service receives a command from the external process and pushes it to the command pool.- The Idling event handler looks at the command pool; if a command is waiting in the pool, it will be executed
      in the Idling event handler.

There is no risk.
All commands are executed in the one and only Revit thread.

Here is the scheme of work:
![Revit external access WCF service workflow](img/RevitExternalAccess_workflow_2.png)

In this implementation, I am using a WCF service.
It would be equally possible to use any other technology or framework to handle the inter-application communication.

Everything you need to know about creating and using a WCF service is described in the
[WCF Getting Started Tutorial](http://msdn.microsoft.com/en-us/library/ms734712.aspx).

I created a simple demonstration project to test this idea.
It implements the following minimal service demonstrating that we can access the Revit API, both to query it for information and execute and action that modifies the current BIM:
```csharp
namespace RevitExternalAccessDemo
{
  [ServiceContract]
  public interface IRevitExternalService
  {
    [OperationContract]
    string GetCurrentDocumentPath();

    [OperationContract]
    bool CreateWall( XYZ startPoint, XYZ endPoint );
  }
}
```

It further requires:

- An external application to manage the Idling event subscription.- An implementation of the service defined above.- Two helper classes:
      - One to contain the queued up tasks and handle locking issues.- The other to redefine an own representation of the Revit API XYZ point class.

Here is a
[short video](http://www.youtube.com/embed/uFHoUC6Met0) showing how it works:

I am happy to make the project available for everyone in its current state.

The source code is available on
[Github](https://github.com/chekalin-v/RevitExternalAccessDemo),
both directly and as a Github
[zip archive file](https://github.com/chekalin-v/RevitExternalAccessDemo/zipball/master).

It is just a demo and provided as is.

To launch it on your computer, you must run Revit as administrator, because this is required by the WCF service.

Remember, Revit must be idling to launch a command from the external application.

In this kind of situation, calling the
[SetRaiseWithoutDelay](http://thebuildingcoder.typepad.com/blog/2012/04/idling-enhancements-and-external-events.html#2) method
is of utmost importance.
Without the call, the demo pauses.
Even with it, some delay exists, and the CPU load grows to over 25% even if no operations are active in Revit.

A better solution exists – follow Arnošt Löbel suggestion: subscribe to the Idling event only when you really need it, and unsubscribe immediately afterwards, as soon as possible.

Obviously, we need to handle Idling only when we have a request queued up from the WCF service.
The idea would be to subscribe to the Idling event when we receive a request from the WCF Service and unsubscribe when it has been processed.

Unfortunately, I have not yet implemented this idea in the code presented here.

Here are some sample code snippets from the current implementation.

First, here is the main external application implementation including:

- OnStartup, which creates the WCF service and subscribes to the Idling event.- OnIdling, the Idling event handler, which dequeues and executes the next task.- OnShutdown, unsubscribing and cleaning up.

```python
class App : IExternalApplication
{
  private const string serviceUrl =
    "http://localhost:56789/RevitExternalService";

  private ServiceHost serviceHost;

  public Result OnStartup(
    UIControlledApplication a )
  {
    a.Idling += OnIdling;

    Uri uri = new Uri( serviceUrl );

    serviceHost = new ServiceHost(
      typeof( RevitExternalService ), uri );

    try
    {
      serviceHost.AddServiceEndpoint(
        typeof( IRevitExternalService ),
        new WSHttpBinding(),
        "RevitExternalService" );

      ServiceMetadataBehavior smb
        = new ServiceMetadataBehavior();

      smb.HttpGetEnabled = true;

      serviceHost.Description.Behaviors.Add( smb );

      serviceHost.Open();
    }
    catch( Exception ex )
    {
      a.ControlledApplication.WriteJournalComment(
        "Could not start WCF service.\r\n"
        + ex.ToString(),
        true );
    }
    return Result.Succeeded;
  }

  private void OnIdling(
    object sender,
    IdlingEventArgs e )
  {
    var uiApp = sender as UIApplication;

    Debug.Print( "OnIdling: {0}",
      DateTime.Now.ToString( "HH:mm:ss.fff" ) );

    // Be careful! This loads the CPU:

    e.SetRaiseWithoutDelay();

    if( !TaskContainer.Instance.HasTaskToPerform )
      return;

    try
    {
      Debug.Print( "Start execute task: {0}",
        DateTime.Now.ToString( "HH:mm:ss.fff" ) );

      var task = TaskContainer.Instance.DequeueTask();

      task( uiApp );

      Debug.Print( "Ending execute task: {0}",
        DateTime.Now.ToString( "HH:mm:ss.fff" ) );
    }
    catch( Exception ex )
    {
      uiApp.Application.WriteJournalComment(
        "RevitExternalService. An error occured "
        + "while executing the OnIdling event:\r\n"
        + ex.ToString(), true );

      Debug.WriteLine( ex );
    }
  }

  public Result OnShutdown( UIControlledApplication a )
  {
    a.Idling -= OnIdling;

    if( serviceHost != null )
    {
      serviceHost.Close();
    }
    return Result.Succeeded;
  }
}
```

Secondly, here is part of the RevitExternalService definition including one of its method implementations:
```python
class RevitExternalService : IRevitExternalService
{
  private string currentDocumentPath;

  private static readonly object \_locker
    = new object();

  private const int WAIT\_TIMEOUT = 10000; // 10 seconds timeout

  public string GetCurrentDocumentPath()
  {
    Debug.Print( "Push task to the container: {0}",
      DateTime.Now.ToString( "HH:mm:ss.fff" ) );

    lock( \_locker )
    {
      TaskContainer.Instance.EnqueueTask( GetDocumentPath );

      // Wait when the task is completed

      Monitor.Wait( \_locker, WAIT\_TIMEOUT );
    }

    Debug.Print( "Finish task: {0}",
      DateTime.Now.ToString( "HH:mm:ss.fff" ) );

    return currentDocumentPath;
  }

  private void GetDocumentPath(
    UIApplication uiapp )
  {
    try
    {
      currentDocumentPath
        = uiapp.ActiveUIDocument.Document.PathName;
    }
    finally
    {
      // Always release locker in finally block
      // to ensure to unlock locker object.

      lock( \_locker )
      {
        Monitor.Pulse( \_locker );
      }
    }
  }

  public bool CreateWall(
    XYZ startPoint,
    XYZ endPoint )
  {
    // . . .
  }
}
```

For completeness' sake, here is also
[VCRevitExternalAccessDemo.zip](zip/VCRevitExternalAccessDemo.zip) containing
my personal version of the code, mainly for my own convenience :-)

Many thanks to Victor for very clear and efficient sample implementation!

By the way, don't miss the additional neat little feature demonstrated by the sample code above, the use of the Application WriteJournalComment method to add a comment to the Revit journal file.

#### Programmatic Creation of a Structural Engineering Plan

Saikat Bhattacharya discusses the details and provides sample code to
[create a new Revit EngineeringPlan view](http://adndevblog.typepad.com/aec/2012/10/creation-of-engineeringplan-using-revit-api.html) using
the static ViewPlan.Create method and a ViewFamily.StructuralPlan ViewFamilyType.