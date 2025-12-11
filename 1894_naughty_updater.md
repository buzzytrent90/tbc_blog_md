---
ai_optimized: true
author: Jeremy Tammik
complexity_score: 4.5
content_type: code_example
optimization_date: '2025-12-11T11:44:16.983996'
original_url: https://thebuildingcoder.typepad.com/blog/1894_naughty_updater.html
post_number: '1894'
reading_time_minutes: 10
series: general
slug: naughty_updater
source_file: 1894_naughty_updater.md
tags:
- csharp
- elements
- family
- levels
- parameters
- python
- revit-api
- schedules
- sheets
- views
- walls
- windows
title: Naughty Updater
word_count: 1900
---

### Naughty Updaters, DIY Add-In Manifest, GD, AI, etc.
Lots of exciting discussion going on in the [Revit API discussion forum](http://forums.autodesk.com/t5/revit-api-forum/bd-p/160) and elsewhere:
- [No redemption for naughty updaters](#2)
- [DIY Add-in manifest](#3)
- [Generative design in C#](#4)
- [AI identifies and classifies BIM elements in 2D sketch](#5)
#### No Redemption for Naughty Updaters
An interesting aspect of the DMU dynamic model updater framework was raised in
the [Revit API discussion forum](http://forums.autodesk.com/t5/revit-api-forum/bd-p/160) thread
on [notifying when `IUpdater` is disabled by Revit error and re-enabling](https://forums.autodesk.com/t5/revit-api-forum/notify-when-iupdater-is-disabled-by-revit-error-amp-re-enable/m-p/10114949),
and clarified for us by Scott Conover:
\*\*Question:\*\* Is there any way to be notified when an IUpdater is disabled by Revit error and re-enable it?
I have a several IUpdaters in my add-in of which a user can disable or enable by clicking an associated button. For example, there is a pushbutton A for IUpdaterA, which the Push Button image shows the status of the IUpdater.
On PushButton Click:
```csharp
if (UpdaterRegistry.IsUpdaterRegistered(updater.GetUpdaterId()))
{
UpdaterRegistry.UnregisterUpdater(updater.GetUpdaterId());
pushButtonA.LargeImage = Off;
}
else
{
UpdaterRegistry.RegisterUpdater(updater);
UpdaterRegistry.EnableUpdater(updater.GetUpdaterId());
// ... Add Triggers, etc. (omitted here for conciseness)
pushButtonA.LargeImage = On;
}
```
This works fine when the user manually clicks and turns on or off the IUpdater.
However, when something during the session or project environment causes the IUpdater execution to fail throwing an error, the button does not respond to re-enabling the IUpdater after it has been disabled:
![Updater experienced a problem](img/updater_experienced_a_problem.jpg "Updater experienced a problem")
So, two questions:
- When this error is thrown and disable Updater is clicked, is there a way to tie this back to change the pushButtonA img to OFF?
- As seen in the code, `UpdaterRegistry.EnableUpdater` is used but it doesn't seem to enable the `IUpdater` back up after it has been disabled through the error dialog. How can one re-enable it?
This is not to say one should always want to re-enable a disabled IUpdater as there was a reason it was disabled, but in some cases under discretion it may be due to something resolvable like loading in a missing family that was needed, etc.
In those situations, it would be ideal to resolve the missing item, and re-enable the IUpdater back up.
Thank you.
\*\*Answer:\*\* Thank you for the interesting question.
If all else fails, have you tried to unregister the updater and reinitialise it completely from scratch?
Or, even more extremely, maybe even change its GUID, so that every failed updater GUID is discarded, and a new one issued on every failure?
Anyway, I have asked the development team whether they have any constructive suggestions for you.
They respond: this is asking the question in the wrong way.
If it's critical to keep the Updater functional, it's on the implementer to ensure that exceptions are not passed back to Revit.
Of course, there are runtime things (System exceptions) that they may not want to catch and deal with, but if the exceptions are thrown from Revit API calls when the updater tries to do its work, I'd suggest the updater catch and deal with them.
It may be that once a call-back or interface class starts throwing exceptions, it goes put on the "naughty list".
There may be no way to recover from that in the current session.
Richard [RPThomas108](https://forums.autodesk.com/t5/user/viewprofilepage/user-id/1035859) Thomas added an explanation of how the current 'naughty list' approach disabling the updater may lead to (the most knowledgeable) people not using DMU at all:
The only possible approach perhaps, since that is a failure, would be to deal with the failure:
- Autodesk.Revit.DB.BuiltInFailures.DocumentFailures.DUMisbehavingUpdater
If you are able to cancel, rather than user doing it (not sure), you are then able to the disable your updater yourself and know about it.
I've long found this approach to 'suspending' rather than disabling DMU's very problematic.
For example, people have asked me 'why can't you make it so pile coordinates are updated automatically when they are moved?'
In theory, I know I could use a DMU for this, but what happens if it gets disabled halfway through an alteration and the user assumes something is being updated when it is not?
I then potentially have a pile schedule with incorrect coordinates being sent out and the distinct possibility of very expensive work being done in the wrong place (I hope some sanity check would prevent that but you never know).
These are the users that say, "I'm just the guy pressing the button (your button)!"
You need some fail-safe approach, really.
So, I prefer to press a button at a certain point to do a task, then check the results against previous before issue.
Even for the most simplistic code DMU's have the potential to be disabled due to complex interactions, i.e., you can account for your own code, but not that of other DMU's by others and the timings of such (the impacts for the states of elements these have between your interactions).
Thanks to Scott and Richard for their input on this.
#### DIY Add-In Manifest
Joshua Lumley added
a [comment](https://thebuildingcoder.typepad.com/blog/2021/02/addin-file-learning-python-and-ifcjs.html#comment-5276653852) on
the discussion on
the [add-in manifest using a relative path](https://thebuildingcoder.typepad.com/blog/2021/02/addin-file-learning-python-and-ifcjs.html#2.1) showing
how to generate your own add-in manifest XML `addin` file on the fly:
> This is the code I use to make the manifest from the CustomMethods of the Deployment Project.
```csharp
void GenerateAddInManifest(
string dll_folder,
string dll_name )
{
string sDir = Environment.GetFolderPath(
Environment.SpecialFolder.CommonApplicationData )
+ "\Autodesk\Revit\Addins";
bool exists = Directory.Exists( sDir );
if( !exists )
Directory.CreateDirectory( sDir );
XElement XElementAddIn = new XElement( "AddIn",
new XAttribute( "Type", "Application" ) );
XElementAddIn.Add( new XElement( "Name", dll_name ) );
XElementAddIn.Add( new XElement( "Assembly", dll_folder + dll_name + ".dll" ) );
XElementAddIn.Add( new XElement( "AddInId", Guid.NewGuid().ToString() ) );
XElementAddIn.Add( new XElement( "FullClassName", dll_name + ".SettingUpRibbon" ) );
XElementAddIn.Add( new XElement( "VendorId", "01" ) );
XElementAddIn.Add( new XElement( "VendorDescription", "Joshua Lumley Secrets, twitter @joshnewzealand" ) );
XElement XElementRevitAddIns = new XElement( "RevitAddIns" );
XElementRevitAddIns.Add( XElementAddIn );
foreach( string d in Directory.GetDirectories( sDir ) )
{
string myString_ManifestPath = d + "\" + dll_name + ".addin";
string[] directories = d.Split( Path.DirectorySeparatorChar );
if( int.TryParse( directories[ directories.Count() - 1 ],
out int myInt_FromTextBox ) )
{
// Install on version 2017 and above
if( myInt_FromTextBox >= 2017 )
{
new XDocument( XElementRevitAddIns ).Save(
myString_ManifestPath );
}
else
{
if( File.Exists( myString_ManifestPath ) )
{
File.Delete( myString_ManifestPath );
}
}
}
}
}
```
Many thanks to Joshua for sharing this DIY approach.
I added it to The Building Coder samples, cf.
the [diff to the previous version](https://github.com/jeremytammik/the_building_coder_samples/compare/2021.0.150.19...2021.0.150.20).
#### Generative Design in C#
Fernando Malard, CTO at [ofcdesk](http://ofcdesk.com), brought up an interesting question that an unnamed colleague of mine kindly clarified:
\*\*Question:\*\* Looking for a suggestion about what route to pursue...
We are creating a Revit plugin for a customer that requires a wall panel tiling system in Revit.
The tiling problem involves lots of optimization variables and it would be perfect to be addressed by a Generative algorithm.
Basically, the plugin would walk through the project walls (inside and outside), perform panel tiling, evaluate, do the genetic operations, repeat, etc.
Is it possible to use Revit GD tools via C# API or is it mandatory to use Dynamo?
I know we could pursue the creation of an SGAII or III algorithm in pure C# but Revit/Forge would give us those extremely helpful tools to visualize design options, parameter graphs, etc.
Any advice here?
\*\*Answer:\*\* It does sound like a good use of GD.
That said, the GD feature doesn’t have an automation API: you use Dynamo to define the parametric models that it uses. The Dynamo graph can use C# “zero-touch” nodes, if you want it to – and people more commonly integrate Python code, when they need to – but that’s just helping flesh out the logic of the graph, it’s not to automate the overall process.
In case it helps, I made a first pass (which is not at all optimal) at doing
a [floor tiling graph for use with Refinery](https://autode.sk/tiling-graph) ([^](zip/RefineryTiling.zip)).
\*\*Response:\*\* Interesting.
Is it possible to trigger the Dynamo graph from Revit and have it running in the background?
Maybe we could create the manager app in C# and call the graph as we need without exposing Dynamo UI to the end user.
I just want to avoid any complexity to the user.
Thanks!
\*\*Answer:\*\* The user doesn’t see Dynamo at all: the GD feature does exactly what you’ve described (actually that’s it’s whole point :-)).
\*\*Response:\*\* It seems your sample graph loads ok but it shows a missing PolyCurve custom node:
![Generative design in C#](img/fm_generative_design_1.jpg "Generative design in C#")
I’m running Revit 2021.1.2, Dynamo Core 2.6.1.8786 and Dynamo Revit 2.6.1.8850.
Any additional package I need to install?
\*\*Answer:\*\* Ampersand, I believe.
\*\*Response:\*\* Exactly, thanks!
![Generative design in C#](img/fm_generative_design_2.jpg "Generative design in C#")
#### AI Identifies and Classifies BIM Elements in 2D Sketch
Before closing, I'd like to pick up a couple of interesting miscellaneous items I happened to run into, starting with
a [LinkedIn post](https://www.linkedin.com/posts/mohamed-adel-a3b26160_autodesk-revit-modeling-activity-6769520499216158720-AFS7)
by Mohamed Adel, BIM Coordinator at SEPCO Electric Power Construction Corporation, Egypt:
> Using machine learning in modelling is quite an approach which definitely will save hours of work.
I developed an application that can automatically model from linked AutoCAD file in Revit.
Using machine learning concept which guide the Revit API to model the proper element.
In his video, a CAD contains only polylines with nothing to distinguish them from each other.
The user provides some initial information in the UI like which family to use in the loadable families, type of system families and working levels.
For any element it will automatically duplicate the family to a new type with the right dimension that fits the linked polyline.
For the depth of the floors, the user will choose which type.

It looks pretty neat!
Here are a few more:
- Why did OS/2 Lose to Windows 3?

[Why did IBM's OS/2 project lose to Microsoft, given that IBM had much more resources than Microsoft at that time?](https://www.quora.com/Why-did-IBMs-OS-2-project-lose-to-Microsoft-given-that-IBM-had-much-more-resources-than-Microsoft-at-that-time/answers/12576993)
- Google Sheets as a REST API and React App –

[How to turn Google sheets into a REST API and use it with a React application](https://www.freecodecamp.org/news/react-and-googlesheets)
- [JavaScript array methods tutorial – the most useful methods explained with examples](https://www.freecodecamp.org/news/complete-introduction-to-the-most-useful-javascript-array-methods)
- [Allserver](https://github.com/flash-oss/allserver), a minimalist multi-transport and multi-protocol simple RPC server and (optional) client