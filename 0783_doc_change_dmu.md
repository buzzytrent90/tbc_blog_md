---
ai_optimized: true
author: Jeremy Tammik
complexity_score: 6.1
content_type: code_example
optimization_date: '2025-12-11T11:44:14.583735'
original_url: https://thebuildingcoder.typepad.com/blog/0783_doc_change_dmu.html
post_number: 0783
reading_time_minutes: 7
series: transactions
slug: doc_change_dmu
source_file: 0783_doc_change_dmu.htm
tags:
- csharp
- elements
- family
- filtering
- python
- references
- revit-api
- transactions
- views
- walls
title: DocumentChanged versus Dynamic Model Updater
word_count: 1408
---

### DocumentChanged versus Dynamic Model Updater

Here is an interesting little overview and comparison of possibilities to react efficiently to the addition of certain elements to the database.
Two easy ways to achieve this are by subscribing to the unspecific [DocumentChanged event](#1"), or by registering a more specific DMU [dynamic model updater](#2").
If you subscribe to an event or register an updater, it is also important to keep in mind to [unsubscribe or unregister](#3") when you are done with it.

We discussed both of these repeatedly in the past, e.g. when looking at
[preventing element deletion](http://thebuildingcoder.typepad.com/blog/2011/11/lock-the-model-eg-prevent-deletion.html) and
[avoiding Idling](http://thebuildingcoder.typepad.com/blog/2012/01/avoid-idling.html).

In this case, the aim is to display a message box to the user when a new elevation view is added.

#### Subscribe to DocumentChanged to React to Elevation View Creation

It is easy to detect when a specific element is added, such as a new elevation view, for instance by subscribing to the DocumentChanged event. This can be done either in the OnStartup method of an external application or from an external command. In either case, you should pay attention to also unsubscribe from the event when it is no longer needed, for instance on application shutdown, at the end of the command, or from some other place.

In order to test the behaviour simply from within the standard framework of The Building Coder sample collection, I implemented a simple external command CmdElevationWatcher subscribing to the event.
The event handler receives a DocumentChangedEventArgs instance providing three separate collections of ids of added, deleted and modified elements.
We pass in the list of added ids to the FindElevationView method, which returns the first elevation view in the given element id collection if one exists, causing a message box listing it to be displayed.

Here is the entire code of this external command implementation:
```python
/// <summary>
/// React to elevation view creation subscribing to DocumentChanged event
/// </summary>
[Transaction( TransactionMode.ReadOnly )]
class CmdElevationWatcher : IExternalCommand
{
  /// <summary>
  /// Return the first elevation view found in the
  /// given element id collection or null.
  /// </summary>
  static View FindElevationView(
    Document doc,
    ICollection<ElementId> ids )
  {
    View view = null;

    foreach( ElementId id in ids )
    {
      view = doc.GetElement( id ) as View;

      if( null != view
        && ViewType.Elevation == view.ViewType )
      {
        break;
      }

      view = null;
    }
    return view;
  }

  /// <summary>
  /// DocumentChanged event handler
  /// </summary>
  static void OnDocumentChanged(
    object sender,
    DocumentChangedEventArgs e )
  {
    Document doc = e.GetDocument();

    View view = FindElevationView(
      doc, e.GetAddedElementIds() );

    if( null != view )
    {
      string msg = string.Format(
        "You just created an "
        + "elevation view '{0}'. Are you "
        + "sure you want to do that? "
        + "(Elevations don't show hidden line "
        + "detail, which makes them unsuitable "
        + "for core wall elevations etc.)",
        view.Name );

      TaskDialog.Show( "ElevationChecker", msg );
    }
  }

  public Result Execute(
    ExternalCommandData commandData,
    ref string message,
    ElementSet elements )
  {
    UIApplication uiapp = commandData.Application;
    Application app = uiapp.Application;

    // Subscribe to DocumentChanged event

    app.DocumentChanged
      += new EventHandler<DocumentChangedEventArgs>(
        OnDocumentChanged );

    return Result.Succeeded;
  }
}
```

After this command has been executed, the event is subscribed to.
Any subsequent elevation view insertion results in the message box being displayed, for instance when I duplicate the East elevation view:
![ElevationWatcher using DocumentChanged](img/ElevationWatcher1.png)

This approach has two significant disadvantages:

1. It reacts erroneously when a family is loaded.- Its performance is suboptimal.

**1.** When a family is loaded into a project, its views are also inserted with it.
This insertion currently triggers a DocumentChanged event on the elevation views as well.
For instance, if I load the standard rectangular column 'family M\_Rectangular Column.rfa', a similar message appears:
![ElevationWatcher reacting to family load](img/ElevationWatcher2.png)

This is unintentional and potentially confusing.

**2.** The performance is not optimal, since DocumentChanged reacts to each and every modification of the document.
Furthermore, every single added element needs to be examined to check whether it is an elevation view.

Both of these disadvantages can easily be avoided by using the dynamic model update mechanism DMU instead.

#### Install Dynamic Model Updater to React to Elevation View Creation

As said, the dynamic model updater mechanism can easily be used to determine that specific elements have been added to the database, and it avoids the DocumentChanged event disadvantages mentioned above, since it reacts only to certain well-defined modifications on a certain well-defined set of elements used to set up the updater trigger.

First we need to implement our updater class to display a message if an elevation view is added:
```csharp
/// <summary>
/// Updater notifying user if an
/// elevation view was added.
/// </summary>
public class ElevationWatcherUpdater : IUpdater
{
  static AddInId \_appId;
  static UpdaterId \_updaterId;

  public ElevationWatcherUpdater( AddInId id )
  {
    \_appId = id;

    \_updaterId = new UpdaterId( \_appId, new Guid(
      "fafbf6b2-4c06-42d4-97c1-d1b4eb593eff" ) );
  }

  public void Execute( UpdaterData data )
  {
    Document doc = data.GetDocument();
    Application app = doc.Application;
    foreach( ElementId id in
      data.GetAddedElementIds() )
    {
      View view = doc.GetElement( id ) as View;

      if( null != view
        && ViewType.Elevation == view.ViewType )
      {
        TaskDialog.Show( "ElevationWatcher Updater",
          string.Format( "New elevation view '{0}'",
            view.Name ) );
      }
    }
  }

  public string GetAdditionalInformation()
  {
    return "The Building Coder, "
      + "http://thebuildingcoder.typepad.com";
  }

  public ChangePriority GetChangePriority()
  {
    return ChangePriority.FloorsRoofsStructuralWalls;
  }

  public UpdaterId GetUpdaterId()
  {
    return \_updaterId;
  }

  public string GetUpdaterName()
  {
    return "ElevationWatcherUpdater";
  }
}
```

With this in place, we can implement a second external test command CmdElevationWatcherUpdater to instantiate and register our updater and define a trigger for it.
The trigger reacts only to the creation of new view elements:
```csharp
public Result Execute(
  ExternalCommandData commandData,
  ref string message,
  ElementSet elements )
{
  UIApplication uiapp = commandData.Application;
  Application app = uiapp.Application;

  // Register updater to react to view creation

  ElevationWatcherUpdater updater
    = new ElevationWatcherUpdater(
      app.ActiveAddInId );

  UpdaterRegistry.RegisterUpdater( updater );

  ElementCategoryFilter f
    = new ElementCategoryFilter(
      BuiltInCategory.OST\_Views );

  UpdaterRegistry.AddTrigger(
    updater.GetUpdaterId(), f,
    Element.GetChangeTypeElementAddition() );

  return Result.Succeeded;
}
```

After this command has been executed once, the updater is registered and its trigger installed.
Like before, any subsequent elevation view insertion results in its message box being displayed, for instance when I again duplicate the East elevation view:
![ElevationWatcher using DMU](img/ElevationWatcher3.png)

#### Unsubscribe from Event and Unregister Updater

Before letting this loose on the general public, I thought I would be a good citizen for the nonce and clean up after myself.

To do so, I modified both command implementations so that each call to the command toggles the event subscription or updater registration on and off.

In both cases, this was very easily achieved.
I simply implemented a static member variable to hold the event handler delegate or the updater and initialised it to null.
It looks like this for the DocumentChanged event handler:
```csharp
  /// <summary>
  /// Keep a reference to the handler, so we know
  /// whether we have already registered and need
  /// to unregister or vice versa.
  /// </summary>
  static EventHandler<DocumentChangedEventArgs>
    \_handler = null;
```

It is even simpler for the updater:
```csharp
  /// <summary>
  /// Keep a reference to the updater, so we know
  /// whether we have already registered and need
  /// to unregister or vice versa.
  /// </summary>
  static ElevationWatcherUpdater \_updater = null;
```

Each time the command is run, it either subscribes/registers and sets the reference if it was null previously, or unsubscribes/unregisters and nulls it in the other case.
Here is the relevant updated code snippet for the DocumentChanged event:
```csharp
  if( null == \_handler )
  {
    \_handler
      = new EventHandler<DocumentChangedEventArgs>(
        OnDocumentChanged );

    // Subscribe to DocumentChanged event

    app.DocumentChanged += \_handler;
  }
  else
  {
    app.DocumentChanged -= \_handler;
    \_handler = null;
  }
```

Here is the corresponding code for DMU:
```csharp
  if( null == \_updater )
  {
    \_updater = new ElevationWatcherUpdater(
      app.ActiveAddInId );

    // Register updater to react to view creation

    UpdaterRegistry.RegisterUpdater( \_updater );

    ElementCategoryFilter f
      = new ElementCategoryFilter(
        BuiltInCategory.OST\_Views );

    UpdaterRegistry.AddTrigger(
      \_updater.GetUpdaterId(), f,
      Element.GetChangeTypeElementAddition() );
  }
  else
  {
    UpdaterRegistry.UnregisterUpdater(
      \_updater.GetUpdaterId() );

    \_updater = null;
  }
```

Here is
[version 2013.0.99.0](zip/bc_13_99_0.zip) of
The Building Coder samples including the two new commands.

I hope you find this comparison useful, appreciate how simple both of these mechanisms are to use, and understand the advantages offered by the DMU.

**Addendum:** As Victor very correctly points out below, I omitted pointing out the main difference between the DocumentChanged event and the DMU mechanism, since we have discussed it several times in the past:

The DocumentChanged event is not raised until after the transaction causing it has been closed, so the changes made cannot be cancelled.
If elements were deleted, they are gone by the time you receive the notification; all you receive from the event handler argument is their element ids, and you cannot even find out what type they were.
The updater is called within the same transaction as the modification causing it, providing much more control.