---
ai_optimized: true
author: Jeremy Tammik
complexity_score: 3.5
content_type: qa
optimization_date: '2025-12-11T11:44:16.528455'
original_url: https://thebuildingcoder.typepad.com/blog/1690_convert_addin_da4r.html
post_number: '1690'
reading_time_minutes: 4
series: general
slug: convert_addin_da4r
source_file: 1690_convert_addin_da4r.md
tags:
- csharp
- parameters
- references
- revit-api
- sheets
- transactions
- views
- walls
- windows
title: Convert Addin Da4r
word_count: 799
---

### Auto-Run an Add-In for Design Automation
Still at the Forge Accelerator in Rome and looking further into
the [Forge](https://autodesk-forge.github.io)
[Design Automation API](https://forge.autodesk.com/en/docs/design-automation/v2/overview) for Revit.
As mentioned yesterday, it is not yet available or documented, except to a closely restricted private beta.
For more information on its current status, please refer to
[Mikako Harada's discussion of Design Automation for Revit](https://fieldofviewblog.wordpress.com/revit).
However, you can stiil start preparing your add-in for the day when it comes:
- [Aspects to consider](#2)
- [Implementing DB application and accessing the Revit `Application` object](#3)
- [DB application add-in manifest](#4)
- [Next steps](#5)
- [Download](#6)
#### Aspects to Consider
Here are some aspects to consider:
- No user interface
- Ensure that no warnings are displayed
- No references to RevitAPIUI, Windows Forms, or other user interface related assemblies
- Driven automatically with input received via JSON files and model path of RVT document to process
- The app is responsible for opening the model itself
Yesterday, as a first example step,
we [modified the StairsAutomations sample to avoid displaying any warnings](http://thebuildingcoder.typepad.com/blog/2018/09/swallowing-stairsautomation-warnings.html),
so the second item listed above is handled.
Today, we'll address most of the remaining ones:
We'll move the execution away from an external command and trigger it from the `ApplicationInitialized` event instead.
In fact, we'll entirely remove all references to `RevitAPIUI.dll`.
We'll also open the model file ourselves.
In Forge, a different system will be used, so you cannot later use the `ApplicationInitialized` event there.
Design Automation for Revit continues doing setup past the point at which `ApplicationInitialized` is raised.
For the time being, though, we can use it to just mimic the 'run automatically' behaviour.
#### Implementing DB Application and Accessing the Revit Application Object
The trickiest step for me was finding out how to access the Revit `Application` object using only the `IExternalDBApplication` interface, because that is apparently not documented anywhere at all.
I finally found the solution in a previous blog post
on [automatically opening a project on start-up](http://thebuildingcoder.typepad.com/blog/2015/03/automatically-open-a-project-on-startup.html) –
the `sender` argument passed in to the `ApplicationInitialized` can be cast to `Application`.
That enables me to implement the entirely UI-independent DB application to drive the stairs creation utility class like this:
```csharp
using System;
using Autodesk.Revit.DB;
using Autodesk.Revit.DB.Events;
using Autodesk.Revit.ApplicationServices;
namespace Revit.SDK.Samples.StairsAutomation.CS
{
///
/// Implement the Revit add-in IExternalDBApplication interface
/// summary>
public class DbApp : IExternalDBApplication
{
string _model_path = "C:/a/vs/StairsAutomation/CS/Stairs_automation_2019_1.rvt";
///
/// The implementation of the automatic stairs creation.
/// summary>
public void Execute( Document document )
{
// Create an automation utility with a hardcoded
// stairs configuration number
StairsAutomationUtility utility
= StairsAutomationUtility.Create(
document, stairsConfigs[stairsIndex] );
// Generate the stairs
utility.GenerateStairs();
++stairsIndex;
if( stairsIndex > 4 )
stairsIndex = 0;
}
void OnApplicationInitialized(
object sender,
ApplicationInitializedEventArgs e )
{
// Sender is an Application instance:
Application app = sender as Application;
Document doc = app.OpenDocumentFile( _model_path );
if( doc == null )
{
throw new InvalidOperationException(
"Could not open document." );
}
Execute( doc );
}
public ExternalDBApplicationResult OnStartup(
ControlledApplication a )
{
// ApplicationInitialized cannot be used in Forge!
a.ApplicationInitialized += OnApplicationInitialized;
return ExternalDBApplicationResult.Succeeded;
}
public ExternalDBApplicationResult OnShutdown(
ControlledApplication a )
{
return ExternalDBApplicationResult.Succeeded;
}
private static int stairsIndex = 0;
private static int[] stairsConfigs = { 0, 3, 4, 1, 2 };
}
}
```
Note the absence of all references to the `Autodesk.Revit.UI` namespace.
#### DB Application Add-In Manifest
Now that we implement no external command, Revit complains that no external command is found:
![External command not found](img/external_command_not_found.png)
That makes perfect sense, of course.
We need to adapt the add-in manifest and inform Revit that we are loading a DB application instead.
As an external application, it requires a `Name` node:
![External application requires a Name node](img/external_application_requires_name_node.png)
We end up with the following add-in manifest file:
```csharp
xml version="1.0" encoding="utf-8"?

StairsAutomation.dllAssembly>
4ce08562-a2e1-4cbf-816d-4923e1363a21ClientId>
Revit.SDK.Samples.StairsAutomation.CS.DbAppFullClassName>
StairsAutomationName>
A utility sample that creates a series of stairs, stairs runs and stairs landings configurations
based upon predefined rules and parameters.Description>
ADSKVendorId>
AddIn>
RevitAddIns>
```
#### Next Steps
There is not much more left to do now, really.
These are all that come to mind off-hand:
- Save the resulting model
- Do we need to shut down Revit when we are done?
- Read the required stair configuration from a JSON input file
- Test in the real Forge environment
The first three we can address right away...
#### Download
Oops, I almost forgot:
You can download the modified SDK sample and examine every step I took in modifying it so far from and in
the [StairsAutomation GitHub repository](https://github.com/jeremytammik/StairsAutomation).