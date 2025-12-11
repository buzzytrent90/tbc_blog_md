---
ai_optimized: true
author: Jeremy Tammik
complexity_score: 8.7
content_type: qa
optimization_date: '2025-12-11T11:44:16.010192'
original_url: https://thebuildingcoder.typepad.com/blog/1431_firerating_2017.html
post_number: '1431'
reading_time_minutes: 18
series: general
slug: firerating_2017
source_file: 1431_firerating_2017.md
tags:
- doors
- elements
- filtering
- parameters
- references
- revit-api
- rooms
- sheets
- transactions
- windows
title: Firerating 2017
word_count: 3590
---

### Real-Time BIM Update with FireRatingCloud 2017
Yesterday,
I [migrated RoomEditorApp to Revit 2017](http://thebuildingcoder.typepad.com/blog/2016/04/room-editor-first-revit-2017-addin-migration.html#3) and
mentioned
the [BIM and cloud related projects](http://thebuildingcoder.typepad.com/blog/2016/04/room-editor-first-revit-2017-addin-migration.html#1) I am working on.
Next, let's focus on
the [FireRatingCloud](https://github.com/jeremytammik/FireRatingCloud) sample.
The main goal there is to implement fully automatic real-time BIM update from the cloud.
Now, 24 hours after writing the previous sentence, I can tell you that I succeeded.
Well, add another six hours to edit this post...
And yet another six hours to struggle with Typepad, which is blocking me from publishing this...
Here is the updated FireRatingCloud custom ribbon tab with its new set of commands:
![FireRatingCloud ribbon tab and commands in Revit 2017](img/firerating_2017_ribbon_tab.png)
The commands are now:
- Cmd_0_About.cs – display an About... box.
- Cmd_1_CreateAndBindSharedParameter – I won't ever tell.
- Cmd_2a_ExportSharedParameterValues – upload data to cloud database using individual REST API calls for each record.
- Cmd_2b_ExportSharedParameterValuesBatch – batch upload data to cloud database using one single REST API call.
- Cmd_3_ImportSharedParameterValues – interactively download data and update BIM from cloud database.
- Cmd_4_Subscribe – subscribe to automatic real-time BIM updates.
Achieving that required the following steps:
- [Already done and yet to do](#2)
- [Migration to Revit 2017](#3)
- [Reusing the RoomEditorApp infrastructure](#4)
- [Redesign](#5)
- [Typepad blocking source code in the blog post](#5.0)
- [App.cs](#5.1)
- [BimUpdater.cs](#5.2)
- [Cmd_4_Subscribe.cs](#5.3)
- [DbAccessor.cs](#5.4)
- [FireRatingCloud video recording](#9)
- [Download](#10)
- [To do](#11)
#### Already Done and Yet To Do
As said, our main goal right now is the support of a round-trip data exchange with the cloud with an optional automatic real-time update of the BIM.
FireRatingCloud will require almost exactly the same functionality as our trusty old [RoomEditorApp](https://github.com/jeremytammik/RoomEditorApp) sample to support that.
We can reuse a lot of the RoomEditorApp implementation, e.g., its external application managing the database polling in a separate thread, the external event to update the BIM when the polling returns modified records, the ribbon user interface for the growing number of commands, the toggle button to turn subscription on and off, etc.
Some tasks that I already addressed for this project include:
- [REST API batch upload and Windows client](http://the3dwebcoder.typepad.com/blog/2016/03/fireratingcloud-rest-api-batch-upload-and-windows-client.html) – the original FireRatingCloud sample made a separate REST API call for each door firerating shared parameter value that it exported to the cloud database. Reimporting modified values uses one single REST call for the whole batch. This enhancement exports in batch as well.
- [Database document modification timestamp](http://the3dwebcoder.typepad.com/blog/2016/04/fireratingcloud-document-modification-timestamp.html) – in order to retrieve modified database records only, a timestamp is required.
- [Retrieving updated docs](http://the3dwebcoder.typepad.com/blog/2016/04/fireratingcloud-query-retrieving-updated-docs.html) – implementation and testing of the timestamp marker and retrieval.
Now I am ready to address the main goal, almost the holy grail:
- Implement automatic round-trip real-time BIM update from the cloud
This entails a couple of subtasks:
- Implement an external event to poll the cloud database for updated records, making use of the document modification timestamp and retrieval of updated docs already implemented.
- Implement an external application to manage the external event.
- Implement a custom ribbon tab and buttons for the user interface to handle the real-time update subscription toggle button.
- Migrate to Revit 2017. I already migrated
the [RoomEditorApp](https://github.com/jeremytammik/RoomEditorApp) sample and am reusing its implementation for several of the preceding items.
Once we have completed the FireRatingCloud sample, here are a few more exciting upcoming cloud related tasks:
- Document and improve [FireRatingClient](), the stand-alone Windows client – we will need this to demonstrate the real-time BIM update from arbitrary sources more nicely than the simple setup I discuss below.
- Completely rewrite the existing [RoomEditorApp](https://github.com/jeremytammik/RoomEditorApp) sample to move it from CouchDB to node.js plus MongoDB.
- Implement the database portion of the [TrackChangesCloud](https://github.com/jeremytammik/TrackChangesCloud) sample.
#### Migration to Revit 2017
Migrating the [RoomEditorApp](https://github.com/jeremytammik/RoomEditorApp) to Revit 2017 yesterday was really easy.
Now let's do the same for FireRatingCloud before anything else.
As always, I need to reference the new Revit API assemblies.
For Revit 2017, I also need to switch from Visual Studio 2012 to 2015 and update the .NET framework version from 4.5 to 4.5.2.
Furthermore, FireRatingCloud references the [RestSharp library](http://restsharp.org), which provides separate assemblies for those two .NET framework versions, so we need to update those references as well.
That was it.
No code changes required, nor anything else further at all.
Here is the full solution in Visual Studio 2015 after implementing all the further enhancements described below:
![FireRatingCloud solution in Visual Studio 2015](img/firerating_2017_sln.png)
The initial [release 2017.0.0.0](https://github.com/jeremytammik/FireRatingCloud/releases/tag/2017.0.0.0) is live in
the [FireRatingCloud GitHub repository](https://github.com/jeremytammik/FireRatingCloud),
and you can examine the flat migration changes by looking at
the [diffs](https://github.com/jeremytammik/FireRatingCloud/compare/2016.0.0.25...2017.0.0.0).
There is much more coming, though.
#### Reusing the RoomEditorApp Infrastructure
The migration was trivial, and only a minute part of the task I am addressing.
Once that was done, I was able to integrate and reuse all the important RoomEditorApp infrastructure to implementing the external application, external event, ribbon UI, command buttons, subscription command toggle button and database polling loop in a separate thread.
That took about a day.
#### Redesign
I spent another day cleaning up the result to make it cleaner and easier to understand.
One important cleanup step, for instance, was to separate the misleadingly named DbUpdater class into two separate classes named BimUpdater and DbAccessor.
The former implements the actual shared parameter value update in the BIM using a method that is used both by the interactive import command and the external event triggered by the database polling when it detects modification to be applied. It also implements this external event.
The latter reads records from the database, also for both the external command and the external event. It also implements the database polling loop that runs in a separate thread and raises the external event when it finds pending modification to be applied.
Can you imagine how much easier it is to understand the architecture when these two components are cleanly separated, and how confusing it is when they are bunched together into one, like they were before?
It all has historical reasons, of course.
Anyway, this sample is now hopefully much easier to comprehend than the RoomEditorApp.
Hence my aim to completely rewrite RoomEditorApp, basing it on this sample next time.
You can possibly get an idea of the various enhancement steps I made by looking at the list of intermediate releases:
- [2017.0.0.1](https://github.com/jeremytammik/FireRatingCloud/releases/tag/2017.0.0.1) – implemented real-time bim update: external app, external event, ribbon ui, subscribe command and toggle button
- [2017.0.0.2](https://github.com/jeremytammik/FireRatingCloud/releases/tag/2017.0.0.2) – converted timestamp to unsigned
- [2017.0.0.3](https://github.com/jeremytammik/FireRatingCloud/releases/tag/2017.0.0.3) – split DbUpdater into separate DbAccessor and BimUpdater classes
- [2017.0.0.4](https://github.com/jeremytammik/FireRatingCloud/releases/tag/2017.0.0.4) – moved and renamed UpdateBimFromDb to BimUpdater.UpdateBim
- [2017.0.0.5](https://github.com/jeremytammik/FireRatingCloud/releases/tag/2017.0.0.5) – cleanup: all except DbAccessor and BimUpdater is now clear
- [2017.0.0.6](https://github.com/jeremytammik/FireRatingCloud/releases/tag/2017.0.0.6) – rewrote UpdateBim to take list of modified doors from DbAccessor or import command
- [2017.0.0.7](https://github.com/jeremytammik/FireRatingCloud/releases/tag/2017.0.0.7) – remove obsolete external commands from add-in manifest and clean up
- [2017.0.0.8](https://github.com/jeremytammik/FireRatingCloud/releases/tag/2017.0.0.8) – replace Debug.Print by Util.Log and clean up using namespace statements
- [2017.0.0.9](https://github.com/jeremytammik/FireRatingCloud/releases/tag/2017.0.0.9) – fixed and created recording
- [2017.0.0.10](https://github.com/jeremytammik/FireRatingCloud/releases/tag/2017.0.0.10) – added readme documents for the two subprojects
FireRatingCloud now consists of the following modules:
- App.cs
- BimUpdater.cs
- Cmd_0_About.cs
- Cmd_1_CreateAndBindSharedParameter.cs
- Cmd_2a_ExportSharedParameterValues.cs
- Cmd_2b_ExportSharedParameterValuesBatch.cs
- Cmd_3_ImportSharedParameterValues.cs
- Cmd_4_Subscribe.cs
- DbAccessor.cs
- DoorData.cs
- Util.cs
The most interesting parts are the new additions, of course:
- [App.cs](#5.1) implements the external application, ribbon UI, command buttons, subscription command toggle button, and manages the external event.
- [BimUpdater.cs](#5.2) implements the external event and the method updating the Revit shared parameters.
- [Cmd_4_Subscribe.cs](#5.3) implements the new subscription external command.
- [DbAccessor.cs](#5.4) implements the database polling loop in a separate thread and raises the external event when external modifications are detected.
Let's look at them one by one.
I checked all the code comments.
They are just about as extensive as they ought to be, no more, no less, and all up to date.
So please read them as well :-)
#### Typepad Blocking Source Code in the Blog Post
Typepad blocked me from posting the four following source code sections.
Every time I tried, it triggered a message saying I have been blocked for security reasons:
> Sorry, you have been blocked. You are unable to access typepad.com. Why have I been blocked? This website is using a security service to protect itself from online attacks. The action you just performed triggered the security solution. There are several actions that could trigger this block including submitting a certain word or phrase, a SQL command or malformed data. CloudFlare Ray ID: 29a278666db82690 – Your IP: XXXXX – Performance & security by CloudFlare...
I submitted a ticket...
I experienced this once already, last month, and it took hours to resolve, wasted for both me and them. More on my side, of course. Painful.
Meanwhile, you can read the [full post](http://jeremytammik.github.io/tbc/a/1431_firerating_2017.html) in the [tbc GitHub repository](https://github.com/jeremytammik/tbc) without the Typepad support.
24 hours later and after several email exchanges and ticket submissions, they say the problem is resolved.
Let's see... yes, it works!
#### App.cs
Implements the external application, ribbon UI, command buttons, subscription command toggle button, and manages the external event:
```csharp
class App : IExternalApplication
{
///
/// Caption
/// summary>
public const string Caption = "FireRatingCloud";
///
/// Switch between subscribe
/// and unsubscribe commands.
/// summary>
const string _subscribe = "Subscribe";
const string _unsubscribe = "Unsubscribe";
///
/// Subscription debugging benchmark timer.
/// summary>
//static JtTimer _timer = null;
///
/// Store the external event.
/// summary>
static ExternalEvent _event = null;
///
/// Executing assembly namespace
/// summary>
static string _namespace = typeof( App ).Namespace;
///
/// Command name prefix
/// summary>
const string _cmd_prefix = "Cmd_";
///
/// Currently executing assembly path
/// summary>
static string _path = typeof( App )
.Assembly.Location;
///
/// Keep track of our ribbon
/// buttons to toggle their text.
/// summary>
static RibbonItem[] _buttons;
///
/// Kepp track of subscription command
/// button whose text is toggled.
/// summary>
static int _subscribeButtonIndex = 4;
#region Icon resource, bitmap image and ribbon panel stuff
///
/// Return path to embedded resource icon
/// summary>
static string IconResourcePath(
string name,
string size )
{
return _namespace
+ "." + "Icon" // folder name
+ "." + name + size // icon name
+ ".png"; // filename extension
}
///
/// Load a new icon bitmap from embedded resources.
/// For the BitmapImage, make sure you reference
/// WindowsBase and PresentationCore, and import
/// the System.Windows.Media.Imaging namespace.
/// summary>
static BitmapImage GetBitmapImage(
Assembly a,
string path )
{
// to read from an external file:
//return new BitmapImage( new Uri(
// Path.Combine( _imageFolder, imageName ) ) );
string[] names = a.GetManifestResourceNames();
Stream s = a.GetManifestResourceStream( path );
Debug.Assert( null != s,
"expected valid icon resource" );
BitmapImage img = new BitmapImage();
img.BeginInit();
img.StreamSource = s;
img.EndInit();
return img;
}
///
/// Create our custom ribbon panel and populate
/// it with our commands, saving the resulting
/// ribbon items for later access.
/// summary>
static void AddRibbonPanel(
UIControlledApplication a )
{
string[] tooltip = new string[] {
"Create and bind shared parameter definition.",
"Export shared parameter values one by one creating new and updating existing documents.",
"Export shared parameter values in batch after deleting all existing project documents.",
"Import shared parameter values.",
"Subscribe to or unsubscribe from updates.",
"About " + Caption + ": ..."
};
string[] text = new string[] {
"Bind Shared Parameter",
"Export one by one",
"Export batch",
"Import",
"Subscribe",
"About..."
};
string[] classNameStem = new string[] {
"1_CreateAndBindSharedParameter",
"2a_ExportSharedParameterValues",
"2b_ExportSharedParameterValuesBatch",
"3_ImportSharedParameterValues",
"4_Subscribe",
"0_About"
};
string[] iconName = new string[] {
"Knot",
"1Up",
"2Up",
"1Down",
"ZigZagRed",
"Question"
};
int n = classNameStem.Length;
Debug.Assert( text.Length == n
&& tooltip.Length == n
&& iconName.Length == n,
"expected equal number of text and class name entries" );
Debug.Assert(
text[_subscribeButtonIndex].Equals( _subscribe ),
"Did you set the correct _subscribeButtonIndex?" );
_buttons = new RibbonItem[n];
RibbonPanel panel
= a.CreateRibbonPanel( Caption );
SplitButtonData splitBtnData
= new SplitButtonData( Caption, Caption );
SplitButton splitBtn = panel.AddItem(
splitBtnData ) as SplitButton;
Assembly asm = typeof( App ).Assembly;
for( int i = 0; i < n; ++i )
{
PushButtonData d = new PushButtonData(
classNameStem[i], text[i], _path,
_namespace + "." + _cmd_prefix
+ classNameStem[i] );
d.ToolTip = tooltip[i];
d.Image = GetBitmapImage( asm,
IconResourcePath( iconName[i], "16" ) );
d.LargeImage = GetBitmapImage( asm,
IconResourcePath( iconName[i], "32" ) );
d.ToolTipImage = GetBitmapImage( asm,
IconResourcePath( iconName[i], "" ) );
_buttons[i] = splitBtn.AddPushButton( d );
}
}
#endregion // Icon resource, bitmap image and ribbon panel stuff
#region External event subscription and handling
///
/// Are we currently subscribed
/// to automatic cloud updates?
/// summary>
public static bool Subscribed
{
get
{
bool rc = _buttons[_subscribeButtonIndex]
.ItemText.Equals( _unsubscribe );
Debug.Assert( ( _event != null ) == rc,
"expected synchronised handler and button text" );
return rc;
}
}
///
/// Toggle on and off subscription to automatic
/// cloud updates. Return true when subscribed.
/// summary>
public static bool ToggleSubscription2(
IExternalEventHandler handler )
{
if( Subscribed )
{
Util.Log( "Unsubscribing..." );
_event.Dispose();
_event = null;
_buttons[_subscribeButtonIndex].ItemText
= _subscribe;
//_timer.Stop();
//_timer.Report( "Subscription timing" );
//_timer = null;
Util.Log( "Unsubscribed." );
}
else
{
Util.Log( "Subscribing..." );
_event = ExternalEvent.Create( handler );
_buttons[_subscribeButtonIndex].ItemText
= _unsubscribe;
//_timer = new JtTimer( "Subscription" );
Util.Log( "Subscribed." );
}
return null != _event;
}
///
/// Provide public read-only access to external event.
/// summary>
public static ExternalEvent Event
{
get { return _event; }
}
#endregion // External event subscription and handling
public Result OnStartup(
UIControlledApplication a )
{
AddRibbonPanel( a );
return Result.Succeeded;
}
public Result OnShutdown(
UIControlledApplication a )
{
if( Subscribed )
{
_event.Dispose();
_event = null;
}
return Result.Succeeded;
}
}
```
#### BimUpdater.cs
Implements the external event and the method updating the Revit shared parameters:
```csharp
///
/// BIM updater, driven both via external
/// command and external event handler.
/// summary>
class BimUpdater : IExternalEventHandler
{
///
/// Update the BIM with the given database records.
/// summary>
public static bool UpdateBim(
Document doc,
List> doors,
ref string error_message )
{
Guid paramGuid;
if ( !Util.GetSharedParamGuid( doc.Application,
out paramGuid ) )
{
error_message = "Shared parameter GUID not found.";
return false;
}
Stopwatch stopwatch = new Stopwatch();
stopwatch.Start();
// Loop through the doors and update
// their firerating parameter values.
if ( null != doors && 0 < doors.Count )
{
using ( Transaction t = new Transaction( doc ) )
{
t.Start( "Import Fire Rating Values" );
// Retrieve element unique id and
// FireRating parameter values.
foreach ( FireRating.DoorData d in doors )
{
string uid = d._id;
Element e = doc.GetElement( uid );
if ( null == e )
{
error_message = string.Format(
"Error retrieving element for "
+ "unique id {0}.", uid );
return false;
}
Parameter p = e.get_Parameter( paramGuid );
if ( null == p )
{
error_message = string.Format(
"Error retrieving shared parameter on "
+ " element with unique id {0}.", uid );
return false;
}
object fire_rating = d.firerating;
p.Set( (double) fire_rating );
p = e.get_Parameter( DoorData.BipMark );
if ( null == p )
{
error_message = string.Format(
"Error retrieving ALL_MODEL_MARK "
+ "built-in parameter on element with "
+ "unique id {0}.", uid );
return false;
}
p.Set( (string) d.tag );
}
t.Commit();
}
}
stopwatch.Stop();
Util.Log( string.Format(
"{0} milliseconds to import {1} element{2}.",
stopwatch.ElapsedMilliseconds, doors.Count,
Util.PluralSuffix( doors.Count) ) );
return true;
}
///
/// Execute method invoked by Revit via the
/// external event as a reaction to a call
/// to its Raise method.
/// summary>
public void Execute( UIApplication a )
{
uint timestamp_before_bim_update
= Util.UnixTimestamp();
Document doc = a.ActiveUIDocument.Document;
Debug.Assert( Util.GetProjectIdentifier( doc )
.Equals( DbAccessor.ProjectId ),
"expected same project" );
string error_message = null;
bool rc = UpdateBim( doc,
DbAccessor.ModifiedDoors,
ref error_message );
if( rc )
{
DbAccessor.Timestamp = timestamp_before_bim_update;
}
else
{
throw new SystemException( error_message );
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
}
```
#### Cmd_4_Subscribe.cs
Implements the new subscription external command:
```csharp
[Transaction( TransactionMode.ReadOnly )]
class Cmd_4_Subscribe : IExternalCommand
{
public Result Execute(
ExternalCommandData commandData,
ref string message,
ElementSet elements )
{
UIApplication uiapp = commandData.Application;
Document doc = uiapp.ActiveUIDocument.Document;
// Determine custom project identifier.
string project_id = Util.GetProjectIdentifier( doc );
if ( !App.Subscribed && 0 == DbAccessor.Timestamp )
{
DbAccessor.Init( project_id );
}
DbAccessor.ToggleSubscription( uiapp );
return Result.Succeeded;
}
}
```
#### DbAccessor.cs
Implements the database polling loop in a separate thread and raises the external event when external modifications are detected:
```csharp
///
/// Read records from the database, optionally filtering
/// according to the modified timestamp, and manage the
/// separate thread running the database polling loop
/// for the subscription command.
/// summary>
class DbAccessor
{
///
/// Current document project id.
/// Todo: update this when switching Revit documents.
/// summary>
static string _project_id = null;
///
/// Return the current Revit project id.
/// summary>
public static string ProjectId
{
get
{
return _project_id;
}
}
///
/// For subscription to automatic BIM updates,
/// retrieve database records modified after
/// this timestamp.
/// summary>
static public uint Timestamp
{
get;
set;
}
///
/// Initialise project id and set the timestamp
/// to start polling for database updates.
/// summary>
static public uint Init( string project_id )
{
_project_id = project_id;
Timestamp = Util.UnixTimestamp();
Util.InfoMsg( string.Format(
"Timestamp set to {0}."
+ "\nChanges from now on will be retrieved.",
Timestamp ) );
return Timestamp;
}
///
/// Store the modified door records
/// retrieved from the database.
/// summary>
static List>
_modified_door_records = null;
///
/// Return the current modified door records
/// retrieved from the cloud database.
/// summary>
public static List> ModifiedDoors
{
get
{
return _modified_door_records;
}
}
///
/// Retrieve all door documents for the specified
/// Revit project identifier, optionally filtering
/// for documents modified after the specified timestamp.
/// summary>
public static List> GetDoorRecords(
string project_id,
uint timestamp = 0 )
{
// Get all doors referencing this project.
string query = "doors/project/" + project_id;
if ( 0 < timestamp )
{
// Add timestamp to query.
Util.Log( string.Format(
"Retrieving door documents modified after {0}",
timestamp ) );
query += "/newer/" + timestamp.ToString();
}
return Util.Get( query );
}
///
/// Count total number of checks for
/// database updates made so far.
/// summary>
static int _nLoopCount = 0;
///
/// Count total number of checks for
/// database updates made so far.
/// summary>
static int _nCheckCount = 0;
///
/// Count total number of database
/// updates requested so far.
/// summary>
static int _nUpdatesRequested = 0;
///
/// Number of milliseconds to wait and relinquish
/// CPU control before next check for pending
/// database updates.
/// summary>
static int _timeout = 500;
#region Windows API DLL Imports
// DLL imports from user32.dll to set focus to
// Revit to force it to forward the external event
// Raise to actually call the external event
// Execute.
///
/// The GetForegroundWindow function returns a
/// handle to the foreground window.
/// summary>
[DllImport( "user32.dll" )]
static extern IntPtr GetForegroundWindow();
///
/// Move the window associated with the passed
/// handle to the front.
/// summary>
[DllImport( "user32.dll" )]
static extern bool SetForegroundWindow(
IntPtr hWnd );
#endregion // Windows API DLL Imports
///
/// This method runs in a separate thread and
/// continuously polls the database for modified
/// records. If any are detected, raise an
/// external event to update the BIM.
/// Relinquish control and wait for the specified
/// timeout period between each attempt.
/// summary>
static void CheckForPendingDatabaseChanges()
{
while ( null != App.Event )
{
++_nLoopCount;
if ( App.Event.IsPending )
{
Util.Log( string.Format(
"CheckForPendingDatabaseChanges loop {0} - "
+ "database update event is pending",
_nLoopCount ) );
}
else
{
//using( JtTimer pt = new JtTimer(
// "CheckForPendingDatabaseChanges" ) )
{
++_nCheckCount;
Util.Log( string.Format(
"CheckForPendingDatabaseChanges loop {0} - "
+ "check for changes {1}",
_nLoopCount, _nCheckCount ) );
_modified_door_records = GetDoorRecords(
_project_id, Timestamp );
if ( null != _modified_door_records
&& 0 < _modified_door_records.Count )
{
App.Event.Raise();
++_nUpdatesRequested;
Util.Log( string.Format(
"database update pending event raised {0} times",
_nUpdatesRequested ) );
// Set focus to Revit for a moment.
// Otherwise, it may take a while before
// Revit reacts to the raised event and
// actually calls the event handler Execute
// method.
IntPtr hBefore = GetForegroundWindow();
SetForegroundWindow(
ComponentManager.ApplicationWindow );
SetForegroundWindow( hBefore );
}
}
}
// Wait and relinquish control before
// next check for pending database updates.
Thread.Sleep( _timeout );
}
}
///
/// Separate thread running the loop
/// polling for pending database changes.
/// summary>
static Thread _thread = null;
///
/// Toggle subscription to automatic database
/// updates. Forward the call to the external
/// application that creates the external event,
/// store it and launch a separate thread checking
/// for database updates. When changes are pending,
/// invoke the external event Raise method.
/// summary>
public static void ToggleSubscription(
UIApplication uiapp )
{
// Todo: stop thread first!
if ( App.ToggleSubscription2( new BimUpdater() ) )
{
// Start a new thread to regularly check the
// database status and raise the external event
// when updates are pending.
_thread = new Thread(
CheckForPendingDatabaseChanges );
_thread.Start();
}
else
{
_thread.Abort();
_thread = null;
}
}
}
```
#### FireRatingCloud Video Recording
Here is an eight and a half minute recording demonstrating the final result, [FireRatingCloud up and running in Revit 2017](https://youtu.be/8VsQJkikXbA):

#### Download
The version presented above
is [release 2017.0.0.12](https://github.com/jeremytammik/FireRatingCloud/releases/tag/2017.0.0.12),
available from the [FireRatingCloud GitHub repository](https://github.com/jeremytammik/FireRatingCloud).
#### To Do
As already mentioned above, I have several more exciting tasks lined up, all related to connecting BIM and the cloud:
- Document and improve [FireRatingClient](https://github.com/jeremytammik/FireRatingCloud/tree/master/FireRatingClient),
the stand-alone Windows client – we will need this to demonstrate the real-time BIM update from arbitrary sources more elegantly than I did above using the minimalistic and rudimentary mongolab web site.
- Completely rewrite the existing [RoomEditorApp](https://github.com/jeremytammik/RoomEditorApp) sample to move it from CouchDB to node.js plus MongoDB.
- Implement the database portion of the [TrackChangesCloud](https://github.com/jeremytammik/TrackChangesCloud) sample.
Stay tuned and wish me luck.
And lots of extra time – I could use a lot more than 24 hours per day, man.
Well, who couldn't?