---
post_number: "1790"
title: "Macro Updater"
slug: "macro_updater"
author: "Jeremy Tammik"
tags: ['csharp', 'elements', 'family', 'filtering', 'revit-api', 'selection', 'sheets', 'transactions', 'walls']
source_file: "1790_macro_updater.md"
original_url: "https://thebuildingcoder.typepad.com/blog/1790_macro_updater.html"
---

### Dynamic Model Updater Macro
Let's take a quick look at implementing a dynamic model updater in a macro:
- [Task](#2)
- [Solution](#3)
- [Drill up the filter](#4)
#### Task
Dave raised and solved an interesting issue concerning macros, an area that we have not discussed much here yet, in
his [Revit API discussion forum](http://forums.autodesk.com/t5/revit-api-forum/bd-p/160) thread
on [`IUpdater` in a project macro on startup](https://forums.autodesk.com/t5/revit-api-forum/iupdater-in-a-project-macro-on-startup/m-p/9087481):
I'm trying to use `IUpdater` in a macro that automatically starts when a project opens.
Is this possible?
So far, I used Macro Manager / Create to set up some boilerplate.
Then, I pasted in Autodesk's `WallUpdater` example code from the knowledge article
on [Implementing IUpdater](https://knowledge.autodesk.com/search-result/caas/CloudHelp/cloudhelp/2015/ENU/Revit-API/files/GUID-6D434229-0A2E-41FE-B29D-1BB2E6471F50-htm.html)
into my `public partial class ThisDocument`.
I figured out how to run the code on project startup by calling it from the boilerplate's `private void Module_Startup`.
But I haven't had any luck calling `WallUpdater`.
Here is my code:
```csharp
using System;
using Autodesk.Revit;
using Autodesk.Revit.DB;
using Autodesk.Revit.UI.Selection;
using Autodesk.Revit.UI;
using Autodesk.Revit.Attributes;
using System.Collections.Generic;
using System.Linq;
namespace test
{
[Transaction( TransactionMode.Manual )]
[Autodesk.Revit.DB.Macros.AddInId( "redacted" )]
public partial class ThisDocument
{
private void Module_Startup( object sender, EventArgs e )
{
TaskDialog.Show( "hello",
"this pops up when you open the project" );
}
private void Module_Shutdown( object sender, EventArgs e )
{
}
#region Revit Macros generated code
private void InternalStartup()
{
this.Startup += new System.EventHandler( Module_Startup );
this.Shutdown += new System.EventHandler( Module_Shutdown );
}
#endregion
public class WallUpdaterApplication : IExternalApplication
{
public Result OnStartup( UIControlledApplication a )
{
// Register wall updater with Revit
WallUpdater updater = new WallUpdater(
a.ActiveAddInId );
UpdaterRegistry.RegisterUpdater( updater );
// Change Scope = any Wall element
ElementClassFilter wallFilter
= new ElementClassFilter(
typeof( Wall ) );
// Change type = element addition
UpdaterRegistry.AddTrigger(
updater.GetUpdaterId(), wallFilter,
Element.GetChangeTypeElementAddition() );
return Result.Succeeded;
}
public Result OnShutdown( UIControlledApplication a )
{
WallUpdater updater = new WallUpdater( a.ActiveAddInId );
UpdaterRegistry.UnregisterUpdater( updater.GetUpdaterId() );
return Result.Succeeded;
}
}
public class WallUpdater : IUpdater
{
static AddInId m_appId;
static UpdaterId m_updaterId;
WallType m_wallType = null;
// constructor takes the AddInId for the add-in
// associated with this updater
public WallUpdater( AddInId id )
{
m_appId = id;
m_updaterId = new UpdaterId( m_appId, new Guid(
"FBFBF6B2-4C06-42d4-97C1-D1B4EB593EFF" ) );
}
public void Execute( UpdaterData data )
{
Document doc = data.GetDocument();
// Cache the wall type
if( m_wallType == null )
{
TaskDialog.Show( "hello", "world" );
}
if( m_wallType != null )
{
TaskDialog.Show( "hello", "world" );
}
}
public string GetAdditionalInformation()
{
return "Wall type updater example: updates all "
+ "newly created walls to a special wall";
}
public ChangePriority GetChangePriority()
{
return ChangePriority.FloorsRoofsStructuralWalls;
}
public UpdaterId GetUpdaterId()
{
return m_updaterId;
}
public string GetUpdaterName()
{
return "Wall Type Updater";
}
}
}
}
```
All the samples I've found so far appear to be written as add-ins.
I found one single sample showing an `IUpdater` in a macro,
for [wrangling revisions with Ruby](https://thebuildingcoder.typepad.com/blog/2014/02/wrangling-revisions-with-ruby.html).
Unfortunately, that's a bit too complex for me to follow at this point, especially in Ruby.
I was hoping to find an RVT with a vanilla C# `IUpdater` macro embedded, similar to the SDK \*Revit_Macro_Samples.rvt\*.
#### Solution
The Boost your BIM article
on [automatically running API code when your model changes](https://boostyourbim.wordpress.com/2012/12/17/automatically-run-api-code-when-your-model-changes) looks
promising...
Yes, I got it working!
Here are the steps I followed:
- Use MacroManager to create a new C# module.
- Edit the new module in SharpDevelop.
- Add `RegisterUpdater` to `Module_Startup` as follows:
```csharp
private void Module_Startup( object sender, EventArgs e )
{
RegisterUpdater();
}
```
- Add `UnregisterUpdater` to `Module_Shutdown` as above.
- Just below the boilerplate `#endregion`, add the boostyourbim code:
```csharp
public class FamilyInstanceUpdater : IUpdater
{ ... }
public void RegisterUpdater()
{ ... }
public void UnregisterUpdater()
{ ... }
```
Make sure `FamilyInstanceUpdater`, `RegisterUpdater`, and `UnregisterUpdater` are inside the scope of `ThisDocument`.
For testing, I found it handy to replace boostyourbim's `Execute` code with this:
```csharp
public void Execute( UpdaterData data )
{
Document doc = data.GetDocument();
TaskDialog.Show( "Revit", "hello" );
}
```
- Save and hit F9 to build the macro.
Check the MacroManager to see if the build was successful.
For some reason, the build occasionally fails for me.
I've found that having AssemblyInfo.cs open in SharpDevelop helps.
No idea why.
At this point, the macro should be working.
Any time you draw an element, you'll get a little popup that says 'hello'.
If you close your project with a successfully built macro, the macro will run automatically the next time you open the project.
I'm sure lots of improvements can be made.
For instance, Jeremy recommended using DocumentOpened.
And my version currently works for columns and beams, but not for walls.
Any input would be appreciated.
Meanwhile, thanks very much to Jeremy for taking a look and to boostyourbim for their code.
Hope this helps someone.
Many thanks to Dave for his research and sharing the solution!
#### Drill Up the Filter
The initial version works for columns and beams, but not for walls.
This is due to the `FamilyInstanceUpdater`.
It defines a `familyInstanceFilter` variable as `new ElementClassFilter( typeof( FamilyInstance ) )`.
Beams and columns are family instances, and walls are not.
To expand the filter to include `Wall` objects as well as `FamilyInstance` objects, you can change its name and definition to something like this:
```csharp
ElementFilter f = new LogicalOrFilter(
new ElementClassFilter( typeof( FamilyInstance ) ),
new ElementClassFilter( typeof( Wall ) );
```

![Macro photography](img/macro_photography.png)