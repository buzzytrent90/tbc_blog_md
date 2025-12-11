---
ai_optimized: true
author: Jeremy Tammik
complexity_score: 4.0
content_type: qa
optimization_date: '2025-12-11T11:44:16.816321'
original_url: https://thebuildingcoder.typepad.com/blog/1822_placing_monitor.html
post_number: '1822'
reading_time_minutes: 5
series: general
slug: placing_monitor
source_file: 1822_placing_monitor.md
tags:
- elements
- family
- references
- revit-api
- sheets
- transactions
- windows
title: Placing Monitor
word_count: 999
---

### Multi-Threading Family Instance Placement Monitor
In the last post, I mentioned
some [undocumented UIFrameworkService utility methods](https://thebuildingcoder.typepad.com/blog/2020/02/lat-long-to-metres-and-duplicate-legend-component.html#4)
pointed out by Kennan Chen of Shanghai in
the [Revit API discussion forum](http://forums.autodesk.com/t5/revit-api-forum/bd-p/160) thread
on [getting notified when a family type is about to be placed](https://forums.autodesk.com/t5/revit-api-forum/get-notified-when-a-family-type-is-about-to-place/m-p/9327282).
He made use of them to implement a family instance placement monitor that could not be achieved using the pure documented Revit API alone.
After the initial publication, he perfected it by elegantly combining the Revit API with additional .NET timer and multi-threading functionality in a novel fashion seldom seen in a Revit add-in.
Let's return to the original question and start fresh from there:
\*\*Question:\*\* Is there an event which can notify when Revit is about to place a family type?
There are events like `Application` `FamilyLoadedIntoDocument` and `FamilyLoadingIntoDocument`.
Is it possible to have another event `FamilyTypePlacingIntoDocument` for this?
Or is there a workaround?
\*\*Answer:\*\* As recently discussed, you
can [use the DocumentChanged event to detect the launching of a command](https://thebuildingcoder.typepad.com/blog/2020/01/torsion-tools-command-event-and-info-in-da4r.html#3).
\*\*Response:\*\* It works great to catch the placing FamilyType event triggered by placing type directly from Revit UI.
After hours of struggle to improve
the [initial solution](https://thebuildingcoder.typepad.com/blog/2020/02/lat-long-to-metres-and-duplicate-legend-component.html#4),
I finally completed this by simply using a `Timer` to constantly check the currently placing type until the UI refreshes and the API call returns correctly.
It may not be the best solution, but at least it works.
I wrapped the logic up in the following class to make it easier.
Hope this can be helpful.
Pay more attention to the potential multi-thread risk and UI performance impact if someone wants to use the code.
It is my pleasure to share this to people who are in need.
As a Chinese, I'm not quite good at explaining things in English. I'll try my best to be accurate :-)
I'll ignore the part how to get notified about a family symbol placing event.
Once the placing event is caught, the next job is to get the placing family symbol.
The core method is the `UIFrameworkServices` `TypeSelectorService` `getCurrentTypeId` method from UIFrameworkServices.dll. This DLL ships with Revit, and can be easily found in the Revit root folder.
I believe this method is designed for the Revit UI framework to get data for the Properties panel, since every time this method is invoked, the returned value is always the element id of the `ElementType` that is currently displayed in the Properties panel.
The biggest problem is, when a family symbol is to be placed, by API call or by UI click, this method doesn't return correctly.
I guess the state doesn't change before Revit really enters placing mode.
But when Revit enters placing mode, no code can be run since the command loop in Revit is stuck.
That's frustrating!
Luckily, another loop is still running: the message loop in every Windows UI application running in STA mode.
In WPF, the message loop is started by Dispatcher in the main UI thread.
UI updates must be queued by the dispatcher to be executed in the main UI thread synchronously.
That's exactly the same mechanism adopted by Revit known as `ExternalEvent`.
In the `FamilyPlacingMonitorService` class, the `DispatcherInvoke` method tests whether the execution is currently in main UI thread.
If not, it queues the delegate method to the UI thread.
To run code after Revit entering placing mode, a `Timer` is created ahead of time to constantly try to resolve the currently placing symbol.
But the Timer doesn't run code in the UI thread, which means it's not safe to call the Revit API.
The trick is to queue the Timer callback to the main UI thread using Dispatcher.
Every UI object (DispatcherObject specifically) holds a reference to the Dispatcher instance; it's easy to get that instance from the Ribbon object (Autodesk.Windows.ComponentManager.Ribbon).
I added some comments to the code to explain it more clearly:
```csharp
public class FamilyPlacingMonitorService
{
#region Constructors
public FamilyPlacingMonitorService( Application app )
{
app.DocumentChanged += App_DocumentChanged;
}
#endregion
#region Others
public event EventHandler
FamilySymbolPlacingIntoDocument;
private void App_DocumentChanged(
object sender,
DocumentChangedEventArgs e )
{
var transactionName = e.GetTransactionNames()
.FirstOrDefault();
if( transactionName == "Modify element attributes" )
{
// treat transactions with the name "Modify
// element attributes" as element placing
// maybe not accurate, but enough to cover
// most scenes
var document = e.GetDocument();
// try to get the current placing family symbol
if( ResolveCurrentlyPlacingFamilySymbol( document )
is FamilySymbol familySymbol )
{
// got ya, notify via event
OnFamilySymbolPlacingIntoDocument( familySymbol );
}
else
{
// current type doesn't refreshed, create
// a Timer to constantly try
Timer timer = null;
// only queue one resolving logic to the main thread
var checking = false;
timer = new Timer( s =>
{
if( !checking )
{
checking = true;
// try to queue the resolving logic to
// main thread to avoid multi-thread risk
DispatcherInvoke( () =>
{
if( ResolveCurrentlyPlacingFamilySymbol(
document ) is FamilySymbol symbol )
{
// got ya, notify via event
OnFamilySymbolPlacingIntoDocument( symbol );
// release the timer
timer?.Change( 0, Timeout.Infinite );
timer?.Dispose();
}
else
{
checking = false;
}
} );
}
}, null, 0, 100 );
}
}
}
private void DispatcherInvoke( Action action )
{
if( ComponentManager.Ribbon.Dispatcher?
.CheckAccess() ?? false )
{
// currently on main thread, execute directly
action?.Invoke();
}
else
{
// not on main thread, queue the delegate
// to main ui thread
ComponentManager.Ribbon.Dispatcher?
.Invoke( action );
}
}
private void OnFamilySymbolPlacingIntoDocument(
FamilySymbol symbol )
{
FamilySymbolPlacingIntoDocument?.Invoke( this, symbol );
}
private FamilySymbol ResolveCurrentlyPlacingFamilySymbol(
Document document )
{
var id = TypeSelectorService.getCurrentTypeId();
if( id > 0 )
{
if( document.GetElement( new ElementId( id ) )
is FamilySymbol symbol )
{
return symbol;
}
}
return null;
}
#endregion
}
```
Very great thanks to Kennan Chen for this extremely knowledgeable, clear and illuminating explanation and elegant juggling of the different threads and contexts!
![Surveillance](img/surveillance.jpg "Surveillance")