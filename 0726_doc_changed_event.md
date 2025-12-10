---
post_number: "0726"
title: "Build Your Own Document Changed Event"
slug: "doc_changed_event"
author: "Jeremy Tammik"
tags: ['elements', 'levels', 'python', 'revit-api', 'views', 'walls', 'windows']
source_file: "0726_doc_changed_event.htm"
original_url: "https://thebuildingcoder.typepad.com/blog/0726_doc_changed_event.html"
---

### Build Your Own Document Changed Event

> **P.S. (pre scriptum)** As Arnošt Löbel very kindly pointed out, I gave this post a rather misleading title.
>
> This post does **not** discuss the Revit API DocumentChanged event, which reacts when elements are modified.
> 'Build your own Document Activated event' would actually more closely advertise my intentions, at least to those of you who already familiar with the DocumentChanged event.
>
> Read on to see what it **does** discuss :-)

Welcome to this extra day of the year, the bisestile invented by Julius Caesar and his friends, bis sexto die, the repeated sixth day after the year's end on February 24th, sexto die ante Kalendas Martias, the sixth day before the month of March.
An event occuring every fourth year, except at the turn of each century.

The Revit API also provides a number of events.
They mostly occur more frequently than the leap year day, but there are still not as many as some would like.
The ones provided may actually cover a larger number of situations than you initially expect, though.
Here is an example of making use of the ViewActivated event to react on document changes.

**Question:** I can't find an event to notify me when the active document changes in Revit.

For example, if I have two RVT projects open in Revit, I would like an event to trigger when the user switches back and forth between them.

**Answer:** One brute force approach to this would be subscribing to the Idling event and checking the active document property in your event handler.
This would be a vastly inefficient overkill, however.

A much more elegant solution is to subscribe to the ViewActivated event instead.
Since there is only one active view for the session, it also triggers when the active document changes.

I implemented a vastly simplified external application named ViewActivatedOnDocChange providing a proof of concept of this idea and nothing more.

It omits a number of important steps such as unsubscribing from the event at shutdown and managing the event subscription to minimize the impact on performance.

Here is the entire external application implementation:
```python
static string ViewDescription( View v )
{
  return string.Format(
    "view '{0}' in document '{1}'",
    v.Name, v.Document.Title );
}

void OnViewActivated(
  object sender,
  ViewActivatedEventArgs e )
{
  View vPrevious = e.PreviousActiveView;
  View vCurrent = e.CurrentActiveView;

  string s = ( null == vPrevious )
    ? "no view at all"
    : "previous " + ViewDescription( vPrevious );

  Debug.Print( string.Format(
    "Switching from {0} to new {1}.",
    s, ViewDescription( vCurrent ) ) );
}

public Result OnStartup( UIControlledApplication a )
{
  a.ViewActivated
    += new EventHandler<ViewActivatedEventArgs>(
      OnViewActivated );

  return Result.Succeeded;
}

public Result OnShutdown( UIControlledApplication a )
{
  return Result.Succeeded;
}
```

This is the output produced in the Visual Studio debug output window by starting up Revit, loading the ViewActivatedOnDocChange add-in, and switching back and forth between a few different documents:

```
Switching from no view at all to
  new view 'Level 1' in document 'walls.rvt'.

Switching from previous view 'Level 1'
  in document 'walls.rvt' to new view
  'Unnamed' in document 'Project1.rvt'.

Switching from previous view 'Unnamed'
  in document 'Project1.rvt' to new view
  'From Parking Area' in document
  'rac_advanced_sample_project.rvt'.
```

Here is
[ViewActivatedOnDocChange.zip](zip/ViewActivatedOnDocChange.zip) containing
the complete source code, Visual Studio solution and add-in manifest for this external application.