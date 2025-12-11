---
ai_optimized: true
author: Jeremy Tammik
complexity_score: 4.7
content_type: code_example
optimization_date: '2025-12-11T11:44:13.953816'
original_url: https://thebuildingcoder.typepad.com/blog/0443_idling_selection_watcher.html
post_number: '0443'
reading_time_minutes: 6
series: filtering
slug: idling_selection_watcher
source_file: 0443_idling_selection_watcher.htm
tags:
- csharp
- elements
- levels
- references
- revit-api
- rooms
- selection
- transactions
- filtering
title: Selection Watcher Using Idling Event
word_count: 1217
---

### Selection Watcher Using Idling Event

Here is a pretty cool example by Ken Goulding of using the Idling event to work around a little gap in the current Revit API.
Ken posted a
[comment](http://thebuildingcoder.typepad.com/blog/2010/04/element-level-events.html?cid=6a00e553e16897883301348655bd79970c#comment-6a00e553e16897883301348655bd79970c) which
triggered a pretty interesting conversation and culminated in the
[SelectionWatcher](zip/SelectionWatcher.zip) sample.

Another interesting item that cropped up was the possibility to use
[PasteBin](http://visualstudio.pastebin.com) to
share snippets of source code, as Ken did for parts of his initial implementation:

- [Class implementation](http://visualstudio.pastebin.com/X5rBXSA6)- [Sample usage](http://visualstudio.pastebin.com/aNwEFRkZ)

The code listed there is now obsolete, though.

Here are some excerpts from our conversation that led up to this sample:

**Ken:** When you mentioned "Element Level", I was hoping that included element selection events, but that does not seem to be the case.
As a work-around, I created a class that subscribes to the application Idling event and checks for selection changes. Given that the Idling event is called very frequently, I tried to be as efficient as possible at figuring out whether a change has happened, but there may be a simpler way?

**Jeremy:** Thank you for your query with the interesting idea and the source code.
Thank you also for the interesting PasteBin URLs. I was not previously aware of this possibility.

Regarding the element selection events, you are correct in your assessment that these are currently not provided by the Revit API.

Your use of the Idling event to implement a work-around sounds like a brilliant idea to me.

**Ken:** I'm glad you see potential for the selection watcher class.
I was using it to link additional data (e.g. photographs) to room objects, but I'm sure your readers can find far more interesting uses for context-sensitive functionality.
I added a few comments inline.
Please let me know if anything is not clear,

**Jeremy:** Thank you very much for the illuminating sample.
I like it very much!
I fixed two details which simplify things:

1. You need to set the 'Copy Local' flag to false on the references to the Revit API DLLs.- You can obtain the application object from the Idling event arguments, so there is no need for a global variable and a DocumentOpening event handler to set it.

**Ken:** Thanks for the suggestions.
It is good to know that you don't need to copy the Revit API DLLs locally, since they are big.

**Jeremy:** Here are some screen snapshots from an example run testing this.
The selection watcher is an external application and defines its own ribbon panel.
Here it is ripped off the ribbon bar and placed over the graphics area, so that it remains visible through a ribbon context switch:

![Selection Watcher ribbon panel](img/SelectionChanged0.png)

Here we have selected Room 5, which is displayed immediately by the selection watcher:

![One room selected](img/SelectionChanged1.png)

In this simple implementation, selection of multiple rooms is simply listed as such with no further details:

![Multiple rooms selected](img/SelectionChanged2.png)

For further details including some comments, please look at the source code in the archive file
[SelectionWatcher.zip](zip/SelectionWatcher.zip).
It includes the full source code, complete Visual Studio solution and an add-in manifest file to load the application.

Here are a few implementation details to whet your appetite:

- A standalone SelectionChangedWatcher class implementation which is completely separate from the external application.- The external application instantiates an instance of the SelectionChangedWatcher class and subscribes to its SelectionChanged event.- The external application does not need to know how the SelectionChanged event is implemented, or that it is internally triggered by the Idling event.

This allows the following minimalistic implementation of the external application class, which just needs to set up its ribbon panel and text box to display the results, instantiate and subscribe to the selection watcher, and update the text box when changes occur:
```csharp
[Transaction( TransactionMode.Manual )]
[Regeneration( RegenerationOption.Manual )]
[Journaling( JournalingMode.NoCommandData )]

public class SelectionChangedExampleApp
  : IExternalApplication
{
  private SelectionChangedWatcher
    \_selectionChangedWatcher;

  private TextBox \_tb;

  public Result OnShutdown(
    UIControlledApplication a )
  {
    return Result.Succeeded;
  }

  public Result OnStartup(
    UIControlledApplication a )
  {
    \_selectionChangedWatcher
      = new SelectionChangedWatcher( a );

    \_selectionChangedWatcher.SelectionChanged
      += new EventHandler(
        OnSelectionChanged );

    // it does not seem to be possible to add items
    // to context-sensitive ribbon panels.
    // however the user can detach this panel from
    // the main ribbon so that it is not hidden by
    // context-sensitive panels.

    RibbonPanel rpSelectionWatcher
      = a.CreateRibbonPanel( "Selection Watcher" );

    var t1 = new TextBoxData( "txtInfo" );

    \_tb = rpSelectionWatcher.AddItem( t1 ) as TextBox;

    return Result.Succeeded;
  }

  void OnSelectionChanged(
    object sender,
    EventArgs e )
  {
    if( null == \_selectionChangedWatcher.Selection )
    {
      ShowInfo( "No selection" );
      return;
    }

    // this example just reports the name of the
    // room that is selected. Obviously any other
    // kind of element can be handled instead.

    List<Room> rooms = new List<Room>(
      \_selectionChangedWatcher.Selection.OfType<Room>() );

    if( 0 == rooms.Count )
    {
      ShowInfo( "No rooms selected" );
    }
    else if( 1 == rooms.Count )
    {
      ShowInfo( "Room " + rooms[0].Number );
    }
    else
    {
      ShowInfo( "Multiple rooms selected" );
    }
  }

  private void ShowInfo( string p )
  {
    \_tb.PromptText = p;
  }
}
```

The SelectionChangedExample class is more interesting, of course.

- It takes the UIControlledApplication as an argument, so that it can subscribe to the Idling event.- It manages the subscription to the Idling event internally, with no need for the client to be aware of that.- It uses an auto-implemented property to store a list of all currently selected elements.- Every selection change is registered in the Idling event handler, updates the selection property, and triggers the SelectionChanged event.

Here it is in all its glory:
```csharp
public class SelectionChangedWatcher
{
  public event EventHandler SelectionChanged;

  /// <summary>
  /// Auto-implemented property storing a list
  /// of all currently selected elements.
  /// </summary>
  public List<Element> Selection
  {
    get;
    set;
  }

  private List<int> \_lastSelIds;

  public SelectionChangedWatcher(
    UIControlledApplication a )
  {
    a.Idling
      += new EventHandler<IdlingEventArgs>(
        OnIdling );
  }

  void OnIdling(
    object sender,
    IdlingEventArgs e )
  {
    // Idling events happen when the application has
    // nothing else to do,
    // They can happen very frequently and the user
    // will experience a lag if this code takes a
    // significant amount of time to execute.

    Application app = sender as Application;

    UIApplication uiApplication
      = new UIApplication( app );

    SelElementSet selected = uiApplication
      .ActiveUIDocument.Selection.Elements;

    if( 0 == selected.Size )
    {
      if( null != Selection && 0 < Selection.Count )
      {
        // if something was selected previously, and
        // now the selection is empty, report change

        HandleSelectionChange( selected );
      }
    }
    else // elements are selected
    {
      if( null == Selection )
      {
        // previous selection was null, report change

        HandleSelectionChange( selected );
      }
      else
      {
        if( Selection.Count != selected.Size )
        {
          // size has changed, no need to check
          // selection IDs, report the change

          HandleSelectionChange( selected );
        }
        else
        {
          // count is the same...
          // compare IDs to see if selection has changed
          if( SelectionHasChanged( selected ) )
          {
            HandleSelectionChange( selected );
          }
        }
      }
    }
  }

  private bool SelectionHasChanged(
    SelElementSet selected )
  {
    // we have already determined that the size of
    // "selected" is the same as the last selection...

    int i = 0;
    foreach( Element e in selected )
    {
      if( \_lastSelIds[i] != e.Id.IntegerValue )
      {
        return true;
      }
      ++i;
    }
    return false;
  }

  private void HandleSelectionChange(
    SelElementSet selected )
  {
    // store the current list of elements in the
    // Selection property and populate \_lastSelIds
    // with the current selection's ids

    Selection = new List<Element>();
    \_lastSelIds = new List<int>();

    foreach( Element e in selected )
    {
      Selection.Add( e );
      \_lastSelIds.Add( e.Id.IntegerValue );
    }
    Call\_SelectionChanged();
  }

  private void Call\_SelectionChanged()
  {
    if( SelectionChanged != null )
    {
      SelectionChanged( this, new EventArgs() );
    }
  }
}
```

Very many thanks to Ken for this idea and implementation and the fruitful discussion!