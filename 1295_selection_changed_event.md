---
ai_optimized: true
author: Jeremy Tammik
complexity_score: 5.6
content_type: code_example
optimization_date: '2025-12-11T11:44:15.704512'
original_url: https://thebuildingcoder.typepad.com/blog/1295_selection_changed_event.html
post_number: '1295'
reading_time_minutes: 5
series: filtering
slug: selection_changed_event
source_file: 1295_selection_changed_event.htm
tags:
- csharp
- elements
- levels
- revit-api
- selection
- transactions
- windows
- filtering
title: Element Selection Changed Event
word_count: 991
---

### Element Selection Changed Event

Many add-in developers are interested in being notified when the current selection changes in the Revit user interface.

Among other things, it led to the implementation of the
[selection watcher using the Idling event](http://thebuildingcoder.typepad.com/blog/2010/09/selection-watcher-using-idling-event.html) and
was one aspect of the discussion of
[element level events](http://thebuildingcoder.typepad.com/blog/2010/04/element-level-events.html).

#### Checking for Selection Changes in Idling Event Handler

Ken Goulding submitted a
[comment](http://thebuildingcoder.typepad.com/blog/2010/04/element-level-events.html?cid=6a00e553e16897883301348655bd79970c#comment-6a00e553e16897883301348655bd79970c) on
that, saying:

> When you mentioned "Element Level", I was hoping that included element selection events, but that does not seem to be the case.
>
> As a work-around, I created this class that subscribes to the application "Idling" event and checks for selection changes.
>
> Given that it is an "Idling" event, I have tried to be as efficient as possible at figuring out whether a change has happened, but there may be a simpler way?
>
> Usage:
>
> ```csharp
> public Result OnStartup( UIControlledApplication a )
> {
> selectionChangedWatcher = new SelectionChangedWatcher( a );
> selectionChangedWatcher.SelectionChanged
> += new EventHandler(
> selectionChangedWatcher\_SelectionChanged);
> // . . .
> ```
>
> Here are the PasteBin URLs to the sample implementation code:
>
> - [Class](http://visualstudio.pastebin.com/X5rBXSA6)
> - [Usage](http://visualstudio.pastebin.com/aNwEFRkZ)

#### Using a UI Automation Event to Notify Selection Changes

Vilo submitted a new
[comment](http://thebuildingcoder.typepad.com/blog/2010/04/element-level-events.html?cid=6a00e553e16897883301bb0807bd22970d#comment-6a00e553e16897883301bb0807bd22970d) with
an updated idea based on using UI Automation instead.

In Vilo's own words:

As many others Revit API users, I miss the "element selection changed" API access.

As mentioned in this post, there are several solutions, each having some caveats:

1. Use the OnIdling event to check current selection.
     – This may not be reliable, because there is no guarantee it will be fired by Revit.
   Secondly, this is an asynchronous method.
2. Use a Timer to raise an event at a specified interval.
     – More reliable, but still asynchronous and the more accurate it is (i.e. shorter timer delay), the more unneeded events are generated.
3. There is a third method that solves all the above "issues":

In Revit, when the user select/unselect an object, there is a special ribbon tab that dynamically changes, its name is "Modify".

Knowing that, and using the standard Autodesk assembly "AdWindows.dll", we can register an event that will be fired when this Tab's title changes.

Here it is the code to subscribe to the event:

```csharp
  foreach( Autodesk.Windows.RibbonTab tab in
    Autodesk.Windows.ComponentManager.Ribbon.Tabs )
  {
    if( tab.Id == "Modify" )
    {
      tab.PropertyChanged += PanelEvent;
      break;
    }
  }
```

The event handler can look like this:

```csharp
  void PanelEvent(
    object sender,
    System.ComponentModel.PropertyChangedEventArgs e )
  {
    if( sender is Autodesk.Windows.RibbonTab )
    {
      if( e.PropertyName == "Title" )
      {
        //selection changed !
      }
    }
  }
```

Not the cleanest way I admit, but until Autodesk adds the relevant API methods, this is (for me) the best.

Update:

- The test "if (sender is Autodesk.Windows.RibbonTab)" is not needed.
- The Tab's name is not dependent to current language.

My Revit is in French, and the tab's name is still "Modify" at start (then it changes according to current selection of course).

Limitation:

- There is actually a limitation I'm working on: the event is not fired after selecting the 3rd element (and 4th, 5th, ...) in a multiselection.

#### CmdSelectionChanged

Jeremy adds: I implemented a new external command
[CmdSelectionChanged](https://github.com/jeremytammik/the_building_coder_samples/blob/master/BuildingCoder/BuildingCoder/CmdSelectionChanged.cs) in
[The Building Coder samples](https://github.com/jeremytammik/the_building_coder_samples) exercising this:

```csharp
#region Namespaces
using System;
using System.Collections.Generic;
using System.Diagnostics;
using System.Linq;
using Autodesk.Revit.Attributes;
using Autodesk.Revit.DB;
using Autodesk.Revit.UI;
#endregion // Namespaces

namespace BuildingCoder
{
  [Transaction( TransactionMode.ReadOnly )]
  class CmdSelectionChanged : IExternalCommand
  {
    static UIApplication \_uiapp;
    static bool \_subscribed = false;

    void PanelEvent(
      object sender,
      System.ComponentModel.PropertyChangedEventArgs e )
    {
      Debug.Assert( sender is Autodesk.Windows.RibbonTab,
        "expected sender to be a ribbon tab" );

      if( e.PropertyName == "Title" )
      {
        ICollection<ElementId> ids = \_uiapp
          .ActiveUIDocument.Selection.GetElementIds();

        int n = ids.Count;

        string s = ( 0 == n )
          ? "<nil>"
          : string.Join( ", ",
            ids.Select<ElementId, string>(
              id => id.IntegerValue.ToString() ) );

        Debug.Print(
          "CmdSelectionChanged: selection changed: "
          + s );
      }
    }

    public Result Execute(
      ExternalCommandData commandData,
      ref string message,
      ElementSet elements )
    {
      \_uiapp = commandData.Application;

      foreach( Autodesk.Windows.RibbonTab tab in
        Autodesk.Windows.ComponentManager.Ribbon.Tabs )
      {
        if( tab.Id == "Modify" )
        {
          if( \_subscribed )
          {
            tab.PropertyChanged -= PanelEvent;
            \_subscribed = false;
          }
          else
          {
            tab.PropertyChanged += PanelEvent;
            \_subscribed = true;
          }
          break;
        }
      }

      Debug.Print( "CmdSelectionChanged: \_subscribed = {0}", \_subscribed );

      return Result.Succeeded;
    }
  }
}
```

Notice that this external command toggles the event subscription on and off on each call.

Obviously this technique would normally not be used in a command, but on startup in an external application.

Also, accessing the ActiveUIDocument.Selection.GetElementIds method in the PanelEvent event handler is absolutely forbidden, since we are not in a valid API context at that point.

A real application would handle this more elegantly and above all legally, e.g. by raising an external event in the PanelEvent event handler, and then accessing the selection set inside the external event handler instead.

Still, for a quick test this works fine.

Here is the result of launching the command, alternately selecting individual Revit elements in the model, clearing the selection set again by clicking into void space, and relaunching the command to terminate the event subscription again:

```
  $ grep CmdSel /tmp/sel.txt
  CmdSelectionChanged: _subscribed = True
  CmdSelectionChanged: selection changed: 121703
  CmdSelectionChanged: selection changed: <nil>
  CmdSelectionChanged: selection changed: 121736
  CmdSelectionChanged: selection changed: <nil>
  CmdSelectionChanged: selection changed: 121785
  CmdSelectionChanged: selection changed: <nil>
  CmdSelectionChanged: selection changed: 121812
  CmdSelectionChanged: selection changed: <nil>
  CmdSelectionChanged: selection changed: 121868
  CmdSelectionChanged: selection changed: <nil>
  CmdSelectionChanged: _subscribed = False
```

The new external command method CmdSelectionChanged lives in the module
[CmdSelectionChanged.cs](https://github.com/jeremytammik/the_building_coder_samples/blob/master/BuildingCoder/BuildingCoder/CmdSelectionChanged.cs) in
[release 2015.0.120.0](https://github.com/jeremytammik/the_building_coder_samples/releases/tag/2015.0.120.0) and later of
[The Building Coder samples](https://github.com/jeremytammik/the_building_coder_samples).