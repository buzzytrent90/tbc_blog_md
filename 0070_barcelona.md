---
post_number: "0070"
title: "Barcelona Questions"
slug: "barcelona"
author: "Jeremy Tammik"
tags: ['csharp', 'elements', 'family', 'filtering', 'levels', 'parameters', 'revit-api', 'rooms', 'transactions', 'views', 'walls', 'windows']
source_file: "0070_barcelona.htm"
original_url: "https://thebuildingcoder.typepad.com/blog/0070_barcelona.html"
---

### Barcelona Questions

Thanks to Alex Vila Ortega of Autodesk Spain, I spent the last couple of days here in Barcelona giving a training class on the Revit API, with lots of professional and enthusiastic participants.
Here are some of the questions that we explored.

**Question:**
How can I retrieve all the windows hosted by a specific given wall?

**Answer:**
Well, one possibility is to make use of the
[relationship inverter](http://thebuildingcoder.typepad.com/blog/2008/10/relationship-in.html)
that we already discussed.
This will provide a global dictionary mapping each wall in the entire model to a list of all the windows it hosts.
Here is an idea for a more direct and localised approach using API filtering:

We can make use of the fact that each window's host id is available not only from the FamilyInstance Host property, but also through the built-in parameter HOST\_ID\_PARAM.
This means that we can build a filter checking for the following combination of properties:

- Category Windows.
- Type FamilyInstance.
- Built-in parameter value for HOST\_ID\_PARAM equal to the given wall element id.

The resulting Boolean 'and' combination should return all the windows hosted by a specific wall.

**Question:**
I create new levels through the API and that works fine.
However, no views are generated for these levels.
How can I generate views for the new levels?

**Answer:**
You might get some ideas by searching globally in the SDKSamples2009 solution for the NewLevel method, which is used to create a new level.
One of the places it is used is the FrameBuilder SDK sample, in FrameData.cs:

```csharp
Level newLevel
= createDoc.NewLevel( elevation );
createDoc.NewViewPlan( newLevel.Name,
newLevel, ViewType.FloorPlan );
```

This appears to be creating a level and immediately creating a new plan view for it as well, which seems to be just what you are looking for.

**Question:**
We need to update some parameter values on certain elements from the document closed and saved events on the application object.
To do so, we implemented an external application class App in order to hook up automatic synchronisation to document save and close events.
We created a method SynchroniseAreaParameters to update the appropriate element parameters, which is registered as an event handler for the OnDocumentClosed, OnDocumentSaved and OnDocumentSavedAs events:

```csharp
public class App : IExternalApplication
{
  void SynchroniseAreaParameters(
    Document doc )
  {
    ElementIterator i = doc.get\_Elements(
      typeof( Room ) );

    while( i.MoveNext() )
    {
      Command.UpdateAreaParameter(
        i.Current as Room );
    }
  }

  public IExternalApplication.Result OnStartup(
    ControlledApplication app )
  {
    app.OnDocumentClosed
      += new DocumentClosedEventHandler(
        SynchroniseAreaParameters );

    app.OnDocumentSaved
      += new DocumentSavedEventHandler(
        SynchroniseAreaParameters );

    app.OnDocumentSavedAs
      += new DocumentSavedAsEventHandler(
        SynchroniseAreaParameters );

    return IExternalApplication.Result.Succeeded;
  }

  public IExternalApplication.Result OnShutdown( ControlledApplication app )
  {
    app.OnDocumentClosed
      -= new DocumentClosedEventHandler(
        SynchroniseAreaParameters );

    app.OnDocumentSaved
      -= new DocumentSavedEventHandler(
        SynchroniseAreaParameters );

    app.OnDocumentSavedAs
      -= new DocumentSavedAsEventHandler(
        SynchroniseAreaParameters );

    return IExternalApplication.Result.Succeeded;
  }
}
```

Unfortunately, this does not work, although we can see in the debugger that the notifications are dispatched as expected and each step executes correctly with no error.
Why are the parameters not updated accordingly?

**Answer:**
The reason why no changes are applied to the document is that you need to encapsulate the modification in a transaction.
In the context of an external command, the Revit API will automatically begin a transaction for you.
Depending on the IExternalCommand.Result value returned from the external command's Execute method, the transaction will be either closed or aborted.
Within the context of an event handler, however, no such transaction will be automatically opened for you, so you are responsible for doing so yourself.
Adding two lines to SynchroniseAreaParameters to begin and end a transaction solved the problem:

```csharp
  void SynchroniseAreaParameters(
    Document doc )
  {
    if( doc.BeginTransaction() )
    {
      ElementIterator i = doc.get\_Elements(
        typeof( Room ) );

      while( i.MoveNext() )
      {
        Command.UpdateAreaParameter(
          i.Current as Room );
      }
      doc.EndTransaction();
    }
  }
```

I am looking forward to more exciting topics to explore and things to learn in our remaining day together.