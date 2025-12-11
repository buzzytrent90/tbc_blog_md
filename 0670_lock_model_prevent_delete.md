---
ai_optimized: true
author: Jeremy Tammik
complexity_score: 5.7
content_type: code_example
optimization_date: '2025-12-11T11:44:14.366574'
original_url: https://thebuildingcoder.typepad.com/blog/0670_lock_model_prevent_delete.html
post_number: '0670'
reading_time_minutes: 5
series: general
slug: lock_model_prevent_delete
source_file: 0670_lock_model_prevent_delete.htm
tags:
- csharp
- elements
- filtering
- python
- references
- revit-api
- selection
- views
- walls
title: Lock the Model, e.g. Prevent Deletion
word_count: 1094
---

### Lock the Model, e.g. Prevent Deletion

I recently discussed two common and interesting issues with developers which ended up being easily resolved:

- Locking the state of certain programmatically generated parts of the model to prevent the user from messing them up.- Simpler still, prevent deletion of certain elements.

Both of these issues can be efficiently addressed using the
[Failure API](http://thebuildingcoder.typepad.com/blog/2010/04/failure-api.html).
It allows us to

- Define our own failure conditions.- Post pre-defined or custom failures.- Handle failures, e.g.
      [suppress unwanted messages](http://thebuildingcoder.typepad.com/blog/2010/08/suppress-unwanted-dialogue.html) or
      completely replace the standard failure processing.

The Revit SDK includes the ErrorHandling sample to demonstrate all of these features,
and we later returned to this topic to discuss various
[failure API aspects in more depth](http://thebuildingcoder.typepad.com/blog/2010/11/failure-api-take-two.html).

So here is the first query that came up:

#### Lock Down Part of the Model

**Question:** I am creating certain AssemblyInstance elements in the model programmatically and would like to prevent the user from being able to modify these.

How can I use the API to lock these elements?

It should be impossible to disassemble them and edit or remove elements.
How can I achieve this?

**Answer:** This can be achieved using the Failure API.
Section 26.1.1 'Defining and registering a failure' in the developer guide shows how you can define a new failure.
If you maintain a cache keeping track of the original state of all your assembly instances, you can define a failure that is triggered when anything is changed in any assembly instance.

The failure API is linked with the Dynamic Model Update framework DMU.

The simpler DocumentChanged event is not cancellable, unfortunately, or you could simply use that.

Below, I demonstrate the detailed implementation steps for the simpler task of simply preventing deletion of certain elements, which does not require keeping track of their previous state.

#### Prevent Deletion of Elements

**Question:** How do I prevent the deletion of an element?
I tried to use DocumentChanged and IUpdater, but both of those seem to be "after the fact" type interfaces.

**Answer:** As said, DocumentChanged is indeed after the fact.

The IUpdater is on the right track.
It is part of the Dynamic Model Update framework DMU and ties in with the failure API, which enables you to easily achieve what you need.

To prove it, I implemented a small sample application PreventDeletion.
It implements three things:

- An external command allowing the user to specify which elements to protect from deletion.- A class DeletionUpdater derived from IUpdater which prevents deletion of selected elements.- An external application to register the updater and its trigger.

All three components are extremely simple.
Let's start with the external command.
It maintains a list of protected element ids and implements an externally accessible IsProtected method:
```csharp
  static List<ElementId> \_protectedIds
    = new List<ElementId>();

  static public bool IsProtected( ElementId id )
  {
    return \_protectedIds.Contains( id );
  }
```

The command itself asks the user to select elements to protect and adds their ids to the list:
```python
public Result Execute(
  ExternalCommandData commandData,
  ref string message,
  ElementSet elements )
{
  UIApplication uiapp = commandData.Application;
  UIDocument uidoc = uiapp.ActiveUIDocument;

  IList<Reference> refs = null;

  try
  {
    Selection sel = uidoc.Selection;

    refs = sel.PickObjects( ObjectType.Element,
      "Please pick elements to prevent from deletion. " );
  }
  catch( OperationCanceledException )
  {
    return Result.Cancelled;
  }

  if( null != refs && 0 < refs.Count )
  {
    foreach( Reference r in refs )
    {
      ElementId id = r.ElementId;

      if( !\_protectedIds.Contains( id ) )
      {
        \_protectedIds.Add( id );
      }
    }
    int n = refs.Count;

    TaskDialog.Show( Caption, string.Format(
      "{0} new element{1} selected and protected "
      + " from deletion, {2} in total.",
      n, ( 1 == n ? "" : "s" ),
      \_protectedIds.Count ) );
  }
  return Result.Succeeded;
}
```

The deletion updater is just as simple.
The only tricky thing about it is that it defines a custom failure definition in its constructor.
The severity of the failure is specified as error, so the user really cannot ignore it, which will prevent the elements from being deleted.
Whatever action that caused the deletion will have to be undone:
```csharp
public DeletionUpdater( AddInId addInId )
{
  \_appId = addInId;

  \_updaterId = new UpdaterId( \_appId, new Guid(
    "6f453eba-4b9a-40df-b637-eb72a9ebf008" ) );

  \_failureId = new FailureDefinitionId(
    new Guid( "33ba8315-e031-493f-af92-4f417b6ccf70" ) );

  FailureDefinition failureDefinition
    = FailureDefinition.CreateFailureDefinition(
      \_failureId, FailureSeverity.Error,
      "PreventDeletion: sorry, this element cannot be deleted." );
}
```

The updater Execute method simply checks whether any of the deleted element ids are protected from deletion and posts the failure if that is the case:
```csharp
public void Execute( UpdaterData data )
{
  Document doc = data.GetDocument();
  Application app = doc.Application;
  foreach( ElementId id in data.GetDeletedElementIds() )
  {
    if( Command.IsProtected( id ) )
    {
      FailureMessage failureMessage
        = new FailureMessage( \_failureId );

      failureMessage.SetFailingElement(id);
      doc.PostFailure(failureMessage);
    }
  }
}
```

In the external application, all we have to do is register the updater and its trigger in the start-up method:
```csharp
public Result OnStartup( UIControlledApplication a )
{
  DeletionUpdater deletionUpdater
    = new DeletionUpdater( a.ActiveAddInId );

  UpdaterRegistry.RegisterUpdater(
    deletionUpdater );

  ElementClassFilter filter
    = new ElementClassFilter(
      typeof( Wall ) );

  UpdaterRegistry.AddTrigger(
    deletionUpdater.GetUpdaterId(), filter,
    Element.GetChangeTypeElementDeletion() );

  return Result.Succeeded;
}
```

If I specify that an element is protected from deletion using the command above and then try to delete it anyway, the standard Revit error message is displayed:

![Prevent deletion error message](img/deletion_prevention_error.png)

If this message should not be seen by the user, you can easily use the failure handling functionality mentioned above to suppress it.

Here is
[PreventDeletion.zip](zip/PreventDeletion.zip) including
the complete source code and Visual Studio solution of this command.

Finally, my first personal use of the failure API :-)

#### Entry Points to Point Clouds in Revit

Here is a short note on a non-API-related issue, an overview of some introductory material on point clouds in Revit from a user point of view:

**Question:** I am interested in retrofit projects and would like to understand better what I can do in Revit using point clouds.
Where should I start, please?

**Answer:** Here are a couple of starting points for you:

- There are videos on the
  [AutodeskBuilding](http://www.youtube.com/user/AutodeskBuilding) youtube
  channel, where the
  [overview video](http://www.youtube.com/watch?v=1NAzX5xcfO4) is
  probably the best place to start.- An interesting web site to look at is
    [eBIM](http://www.ebim.co.uk),
    of a company in the UK that built an entire business model around as-built BIM authoring in Revit.- The very nice data set to understand the point cloud integration is provided by the
      [Revit model and point cloud of the 210 King office in Toronto](http://revit.downloads.autodesk.com/download/2012Datasets/210 King.zip).