---
ai_optimized: true
author: Jeremy Tammik
complexity_score: 5.5
content_type: qa
optimization_date: '2025-12-11T11:44:16.034958'
original_url: https://thebuildingcoder.typepad.com/blog/1442_roomedit3d.html
post_number: '1442'
reading_time_minutes: 8
series: general
slug: roomedit3d
source_file: 1442_roomedit3d.md
tags:
- csharp
- elements
- revit-api
- rooms
- selection
- sheets
- transactions
- views
- windows
title: Roomedit3d
word_count: 1577
---

### Roomedit3d Live Real-Time BIM Update Recording
I completed the first running version of my [roomedit3d](https://github.com/jeremytammik/roomedit3d) project
connecting BIM and the cloud demonstrating two cool possibilities to enhance interaction with the View and Data API:
- A viewer extension enabling interactive model modification, i.e., translation of selected elements
- Real-time communication of the modification back to the source CAD system using a direct socket.io connection to broadcast from the web server to any number of desktop clients
Let's take a closer look at the implementation and a test run recording:
- [Overview](#1)
- [Roomedit3d implementation](#2)
- [External command to toggle broadcast subscription](#3)
- [BimUpdater external event handler](#4)
- [External application managing socket.io and external event](#5)
- [Test run video recording](#6)
- [Download](#7)
- [To do](#8)
- [iTerm2](#9)
#### Overview
All the background information and full discussions of the implementation details are provided in my mention of
the [initial idea](http://thebuildingcoder.typepad.com/blog/2016/05/idea-station-and-textnote-bounding-box.html), the
working [proof of concept with a C# .NET console test application](http://the3dwebcoder.typepad.com/blog/2016/05/roomedit3d-viewer-translation-extension-post-and-socket.html) and
the overview of
the [Revit-independent implementation aspects](http://thebuildingcoder.typepad.com/blog/2016/05/roomedit3d-console-test-and-rendering-assets.html#2).
Here is a quick recapitulation:
- We start off with a Revit BIM
- The BIM model is translated for sharing in the [Forge](http://forge.autodesk.com) [View and Data API](https://developer.autodesk.com/api/view-and-data-api) viewer
- The [roomedit3d](https://github.com/jeremytammik/roomedit3d) node.js web server hosts the viewer and provides a REST API for it to request authorisation tokens from
- The viewer is equipped with an extension enabling manual selection and translation of individual elements
- The viewer extension detects and reports the selected element external id and translation vector back to the web server via a REST API call
- The web server opens a [socket.io](http://socket.io) channel and broadcasts the translation data
- The [Roomedit3dApp](https://github.com/jeremytammik/Roomedit3dApp) C# .NET Revit API add-in subscribes to the socket.io broadcast
- The add-in implements an external event to update the BIM when the translation data is received
- The BimUpdater class holds the requested translation tasks in a queue and processes them in its `Execute` method
#### Roomedit3d Implementation
All the interesting aspects are implemented by just two modules, the BIM updater and the external application managing the socket.io subscription and external event.
In fact, the BIM updater is kind of trivial, too, so really `App.cs` is the only really interesting part :-)
Let's look at both, though, and the external command as well, for the sake of completeness:
- [External command to toggle broadcast subscription](#3)
- [BimUpdater external event handler](#4)
- [External application managing socket.io and external event](#5)
#### External Command to Toggle Broadcast Subscription
```csharp
[Transaction( TransactionMode.ReadOnly )]
public class Command : IExternalCommand
{
public Result Execute(
ExternalCommandData commandData,
ref string message,
ElementSet elements )
{
bool subscribed = App.ToggleSubscription();
TaskDialog.Show( "Toggled Subscription",
( subscribed ? "S" : "Uns" ) + "ubscribed." );
return Result.Succeeded;
}
}
```
#### BimUpdater External Event Handler
```csharp
/// BIM updater, driven both via external
/// command and external event handler.
/// summary>
class BimUpdater : IExternalEventHandler
{
///
/// The queue of pending tasks consisting
/// of UniqueID and translation offset vector.
/// summary>
Queue> _queue = null;
public BimUpdater()
{
_queue = new Queue>();
}
///
/// Execute method invoked by Revit via the
/// external event as a reaction to a call
/// to its Raise method.
/// summary>
public void Execute( UIApplication a )
{
Document doc = a.ActiveUIDocument.Document;
using ( Transaction t = new Transaction( doc ) )
{
t.Start( GetName() );
while ( 0 < _queue.Count )
{
Tuple task = _queue.Dequeue();
Debug.Print( "Translating {0} by {1}",
task.Item1, Util.PointString( task.Item2 ) );
Element e = doc.GetElement( task.Item1 );
ElementTransformUtils.MoveElement(
doc, e.Id, task.Item2 );
}
t.Commit();
}
}
///
/// Required IExternalEventHandler interface
/// method returning a descriptive name.
/// summary>
public string GetName()
{
return App.Caption + " " + GetType().Name;
}
///
/// Enqueue a BIM update action to be performed,
/// consisting of UniqueID and translation
/// offset vector.
/// summary>
public void Enqueue( string uid, XYZ offset )
{
_queue.Enqueue(
new Tuple(
uid, offset ) );
}
}
```
#### External Application Managing Socket.io and External Event
```csharp
class App : IExternalApplication
{
///
/// Caption
/// summary>
public const string Caption = "Roomedit3d";
///
/// Socket broadcast URL.
/// summary>
const string _url = "https://roomedit3d.herokuapp.com:443";
#region External event subscription and handling
///
/// Store the external event.
/// summary>
static ExternalEvent _event = null;
///
/// Store the external event.
/// summary>
static BimUpdater _bimUpdater = null;
///
/// Store the socket we are listening to.
/// summary>
static Socket _socket = null;
///
/// Provide public read-only access to external event.
/// summary>
public static ExternalEvent Event
{
get { return _event; }
}
///
/// Enqueue a new BIM updater task.
/// summary>
static void Enqueue( object data )
{
JObject data2 = JObject.FromObject( data );
string s = string.Format(
"transform: uid={0} ({1:0.00},{2:0.00},{3:0.00})",
data2["externalId"], data2["offset"]["x"],
data2["offset"]["y"], data2["offset"]["z"] );
Util.Log( "Enqueue task " + s );
string uid1 = data2["externalId"].ToString();
XYZ offset1 = new XYZ(
double.Parse( data2["offset"]["x"].ToString() ),
double.Parse( data2["offset"]["y"].ToString() ),
double.Parse( data2["offset"]["z"].ToString() ) );
string uid = (string) data2["externalId"];
XYZ offset = new XYZ(
(double) data2["offset"]["x"],
(double) data2["offset"]["y"],
(double) data2["offset"]["z"] );
_bimUpdater.Enqueue( uid, offset );
_event.Raise();
}
///
/// Toggle on and off subscription to automatic
/// BIM update from cloud. Return true when subscribed.
/// summary>
public static bool ToggleSubscription()
{
if ( null != _event )
{
Util.Log( "Unsubscribing..." );
_socket.Disconnect();
_socket = null;
_bimUpdater = null;
_event.Dispose();
_event = null;
Util.Log( "Unsubscribed." );
}
else
{
Util.Log( "Subscribing..." );
_bimUpdater = new BimUpdater();
var options = new IO.Options()
{
IgnoreServerCertificateValidation = true,
AutoConnect = true,
ForceNew = true
};
_socket = IO.Socket( _url, options );
_socket.On( Socket.EVENT_CONNECT, ()
=> Util.Log( "Connected" ) );
_socket.On( "transform", data
=> Enqueue( data ) );
_event = ExternalEvent.Create( _bimUpdater );
Util.Log( "Subscribed." );
}
return null != _event;
}
#endregion // External event subscription and handling
public Result OnStartup( UIControlledApplication a )
{
return Result.Succeeded;
}
public Result OnShutdown( UIControlledApplication a )
{
return Result.Succeeded;
}
}
```
That's all there is too it!
Are you surprised how short and easy it is?
La perfection est atteinte, non pas lorsqu'il n'y a plus rien à ajouter, mais lorsqu'il n'y a plus rien à retirer.
\*Perfection is achieved, not when there is nothing more to add, but when there is nothing left to take away.\*

*Antoine de Saint-Exupéry*

#### Test Run Video Recording
Now the time has come to show the full solution running live, connecting the [View and Data API](https://developer.autodesk.com/api/view-and-data-api) viewer and the Revit add-in updating the BIM live in real-time.
Here is a [five-minute video recording](https://youtu.be/EbtyAZPX8Bc) showing the system up and running:

To my great surprise, I do not have to do anything at all to convert or transform the viewer coordinates back into the Revit model.
In other words, the viewer seems to be using the same units as Revit does, i.e., imperial feet, and swapping the X, Y and Z axes appropriately too.
I had expected to have implement and apply some kind of transformation myself.
#### Download
The version up and running in the recording above is the Revit
add-in [Roomedit3dApp release 2017.0.0.4](https://github.com/jeremytammik/Roomedit3dApp/releases/tag/2017.0.0.4) with
the web server and viewer hosted by [roomedit3d release 0.0.4](https://github.com/jeremytammik/roomedit3d/releases/tag/0.0.4) running
on [Heroku](https://www.heroku.com).
The most up-to-date versions are always provided by
the [Roomedit3dApp](https://github.com/jeremytammik/Roomedit3dApp)
and [roomedit3d](https://github.com/jeremytammik/roomedit3d) master
branches, respectively, and the main documentation is in the latter.
I hope you find this useful and wish you much fun and success connecting the desktop and the cloud, and BIM with many powerful [Forge](http://forge.autodesk.com) and own custom web services.
By the way, I very much hope you can make your way to the [Forge DevCon](http://forge.autodesk.com/conference) coming up real soon now, in just two weeks time!
I look forward to seeing you there!
Many thanks to Philippe Leefsma for all his help in Barcelona getting the basics up and running! Philippe implemented most of the code in the web server, including the viewer management, viewer extension and socket.io broadcast.
#### To Do
This completes the old first item in my [previous to do list](http://the3dwebcoder.typepad.com/blog/2016/05/roomedit3d-viewer-translation-extension-post-and-socket.html#13).
Here is an update with a new first entry:
- Explore how to update the Revit BIM automatically immediately, without having to click into the window first. This may be as easy as simply briefly setting the Windows focus on the Revit main window.
- Besides translation, I would also like the View and Data extension to handle rotation in the XY plane.
- Since this runs so well here, I would like to update the [FireRatingCloud](https://github.com/jeremytammik/FireRatingCloud) sample to use the same technology and implement a socket.io connection between the FireRatingCloud C# .NET Revit add-in and its [fireratingdb](https://github.com/jeremytammik/firerating) node.js web and MongoDB server.
#### iTerm2
I just installed and started using [iTerm2](http://iterm2.com),
again based on Philippe's recommendation last week.
So far, the switch from the standard Mac terminal to iTerm2 is completely seamless, and now I am looking forward to discovering and exploring the [numerous advantages](http://iterm2.com/features.html) one by one as they in handy.
Thanks again, Philippe!