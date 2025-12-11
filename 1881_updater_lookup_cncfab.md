---
ai_optimized: true
author: Jeremy Tammik
complexity_score: 4.2
content_type: qa
optimization_date: '2025-12-11T11:44:16.953826'
original_url: https://thebuildingcoder.typepad.com/blog/1881_updater_lookup_cncfab.html
post_number: '1881'
reading_time_minutes: 7
series: general
slug: updater_lookup_cncfab
source_file: 1881_updater_lookup_cncfab.md
tags:
- elements
- family
- filtering
- levels
- parameters
- references
- revit-api
- sheets
- transactions
- views
- walls
title: Updater Lookup Cncfab
word_count: 1401
---

### Simple IUpdater and other TBC Updates
A nice new minimal DMU example, updates and enhancements to other important sample applications:
- [Simple dynamic model updater example](#2)
- [ExportCncFab `SortMark` update](#3)
- [RevitLookup exception on view `GetTemplateParameterIds`](#4)
#### Simple Dynamic Model Updater Example
Alan Clarke shared a nice new simple Dynamic Model Updater or DMU sample in
the [Revit API discussion forum](http://forums.autodesk.com/t5/revit-api-forum/bd-p/160) thread
on [`IUpdater`, simple example needed](https://forums.autodesk.com/t5/revit-api-forum/iupdater-simple-example-needed/m-p/9893248),
saying:
> Hi Jeremy, I just want to share my solution, with help from a friend.
I hope this is simple enough of an example and helpful for others.
This is an `IExternalCommand` example.
I like it and integrated it
into [The Building Coder samples](https://github.com/jeremytammik/the_building_coder_samples)
[release 2021.0.150.10](https://github.com/jeremytammik/the_building_coder_samples/releases/tag/2021.0.150.10).
Here is the [diff to the previous release](https://github.com/jeremytammik/the_building_coder_samples/compare/2021.0.150.9...2021.0.150.10).
In doing so, I noticed they already did include another nice DMU sample in the module `CmdElevationWatcher` that reacts to elevation view creation.
It compares the use of DMU versus the `DocumentChanged` event to track that happening, discussed in the article
on [`DocumentChanged` versus Dynamic Model Updater](https://thebuildingcoder.typepad.com/blog/2012/06/documentchanged-versus-dynamic-model-updater.html).
Here is my slightly modified version of your code:
```csharp
[Transaction( TransactionMode.Manual )]
class CmdSimpleUpdaterExample : IExternalCommand
{
class SimpleUpdater : IUpdater
{
const string Id = "d42d28af-d2cd-4f07-8873-e7cfb61903d8";
UpdaterId _updater_id { get; set; }
public SimpleUpdater(
Document doc,
AddInId addInId )
{
_updater_id = new UpdaterId(
addInId, new Guid( Id ) );
RegisterUpdater( doc );
RegisterTriggers();
}
public void RegisterUpdater( Document doc )
{
if( UpdaterRegistry.IsUpdaterRegistered(
_updater_id, doc ) )
{
UpdaterRegistry.RemoveAllTriggers( _updater_id );
UpdaterRegistry.UnregisterUpdater( _updater_id, doc );
}
UpdaterRegistry.RegisterUpdater( this, doc );
}
public void RegisterTriggers()
{
if( null != _updater_id
&& UpdaterRegistry.IsUpdaterRegistered(
_updater_id ) )
{
UpdaterRegistry.RemoveAllTriggers( _updater_id );
UpdaterRegistry.AddTrigger( _updater_id,
new ElementCategoryFilter( BuiltInCategory.OST_Walls ),
Element.GetChangeTypeAny() );
}
}
public void Execute( UpdaterData data )
{
Document doc = data.GetDocument();
ICollection ids
= data.GetModifiedElementIds();
IEnumerable names
= ids.Select(
id => doc.GetElement( id ).Name );
TaskDialog.Show( "Simple Updater",
string.Join( ",", names ) );
}
public string GetAdditionalInformation()
{
return "NA";
}
public ChangePriority GetChangePriority()
{
return ChangePriority.MEPFixtures;
}
public UpdaterId GetUpdaterId()
{
return _updater_id;
}
public string GetUpdaterName()
{
return "SimpleUpdater";
}
}
public Result Execute(
ExternalCommandData commandData,
ref string message,
ElementSet elements )
{
UIApplication uiapp = commandData.Application;
Document doc = uiapp.ActiveUIDocument.Document;
AddInId addInId = uiapp.ActiveAddInId;
SimpleUpdater su = new SimpleUpdater( doc, addInId );
return Result.Succeeded;
}
}
```
Many thanks to Alan for putting together and sharing this.
#### ExportCncFab SortMark Update
Another nice sample was also enhanced, prompted
by [SteelcoreSystems](https://github.com/SteelcoreSystems)'
[issue #1 – Add Instance Shared Parameter Value in Exported Filename](https://github.com/jeremytammik/ExportCncFab/issues/1) on
the ExportCncFab add-in to export Revit wall parts to DXF or SAT for CNC fabrication:
> I would like to know how to add an additional shared parameter instance value to the exported SAT/DXF filenames. The current SAT/DXF filename method only exports the "host level name_parentId_partId"
> Only exporting the Revit element id makes it difficult and time consuming to determine what part was exported after exporting many elements. I have added an instance shared parameter to my Revit models and assigned it to parts. The goal is to have this instance shared parameter value included within each exported filename.
> When exporting, I get the following default file name:
```csharp
[Level Name]_136667_136683.sat
```
> I would like for the file name to be exported with the "SortMark" instance value:
```csharp
[Level Name]_[SortMark]_136667_136683.sat
Level 0_3SB-10_13667_136683.sat
```
> Here is an image of the modified code:
![ExportCncFab SortMark](img/cncfab_sortmark.png "ExportCncFab SortMark")
Before buckling down to this, I migrated ExportCncFab from Revit 2019 to Revit 2020 and Revit 2021.
That was trivial.
Then I implemented the required enhancement that you can see in
the [diff for adding the sort mark](https://github.com/jeremytammik/ExportCncFab/compare/2021.0.0.0...2021.0.0.1).
\*\*Response:\*\* Great Work! I made the adjustments and on the first try everything exported perfectly.
I greatly appreciate you making it easy to assign any parameter name for value exporting.
Under the ExportCncFab/ExportParameters.cs file at line 32, I was able to change
const string `_sort_mark` = `"CncFabSortMark"` to my desired parameter.
Also, I have been able to customize the code for exporting floor parts, wall parts, and structural steel framing elements.
So far, it has worked perfectly with every `OST` family I have tested.
It is a great app!
Many thanks to SteelcoreSystems for raising this issue and your nice appreciation.
#### RevitLookup Exception on View GetTemplateParameterIds
We also updated RevitLookup to handle
the [issue #65 – Unhandled exception on View.GetTemplateParameterIds](https://github.com/jeremytammik/RevitLookup/issues/65) raised
by [RevitArkitek](https://github.com/RevitArkitek).
I suggested submitting a pull request for this functionality, which involved talking them through the basic steps required for this fundamental GitHub collaboration operation: fork, clone, modify, commit, pull.
\*\*Question:\*\* In a view (whether it's a view template or not), I click on `GetTemplateParameterIds`, a `NullReferenceException` is thrown.
\*\*Answer:\*\* Thank you for pointing it out.
Are you a programmer?
Do you have Visual Studio or something similar installed?
Can you run RevitLookup and Revit.exe in the debugger and see where this happens?
In that case, can you please wrap the offending line of code in a try/catch handler to handle the exception?
And submit a pull request with the fixed code?
That would be great!
Thank you!
Later: I searched the RevitLookup source code globally to try to understand how this can occur.
I searched, found and analysed all occurrences of SnoopableObjectWrapper and GetUnderlyingType, and they all make perfect sense and look perfectly all right to me.
GetTemplateParameterIds is a member on the View class.
It is called dynamically, and I cannot tell from your report how to trigger such a call.
Therefore, it would be helpful if you could do so.
The problem is probably trivial to fix and faster done than writing this long message.
Looking forward to your pull request!
Thank you!
\*\*Response:\*\* When I debugged, Object in public Type GetUnderlyingType() => Object.GetType(); was null.
I changed to public Type GetUnderlyingType() => Object?.GetType();
Then, in the Objects.AddObjectsToTree I had to handle the null being returned.
After that, it would open the next dialog.
\*\*Answer:\*\* Thank you! That helps narrow it down tremendously.
Could you share the handling code in `AddObjectsToTree`, or submit a pull request for the changes?
Have you made any other modifications and improvements? Cheers!
\*\*Response:\*\* I'm going to do a little testing, and then try to do a Pull request.
I've never done that before. Thanks!
Later: I figured out that the main issue was further up the code and had to add a couple of handlers for these objects.
Unfortunately, it won't let me push my commit.
GitHub desktop doesn't recognize my username/password.
Not sure how to proceed.
\*\*Answer:\*\* Wow, that sounds very good!
Looking forward very much to see your approach.
Yes, that is completely normal.
You cannot commit straight to my repository, since you might wreck the project for me.
That is obvious and expected.
The solution is easy: (i) fork my repository to your own copy of it. (ii) clone your repository, make changes, and commit as you please. (iii) in your forked directory, submit a pull request. that enables me to see the changes you made and merge them back into my main original source repo.
It all makes total sense and is utterly powerful and convenient.
You have learned something useful.
Search for something like [github fork commit pull merge](https://duckduckgo.com/?q=github+fork+commit+pull+merge).
That gives many hits.
One is [How to Fork Github Repository, Create Pull Request and Merge?](https://crunchify.com/how-to-fork-github-repository-create-pull-request-and-merge)
That interaction promptly resulted
in [pull request #66 – Adds handlers for GetTemplateParameterIds and GetNonControlledTemplateParameterIds](https://github.com/jeremytammik/RevitLookup/pull/66) that
I integrated into [RevitLookup release 2021.0.0.8](https://github.com/jeremytammik/RevitLookup/releases/tag/2021.0.0.8).
Since then, I created another [release 2021.0.0.9](https://github.com/jeremytammik/RevitLookup/releases/tag/2021.0.0.9) to
locally disable warning CS0618, `DisplayUnitType` is obsolete, for one specific use case.
Very many thanks to RevitArkitek for their valuable contribution!