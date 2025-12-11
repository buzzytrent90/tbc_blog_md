---
ai_optimized: true
author: Jeremy Tammik
complexity_score: 2.1
content_type: qa
optimization_date: '2025-12-11T11:44:16.471931'
original_url: https://thebuildingcoder.typepad.com/blog/1655_revit_2019_sdk_apr.html
post_number: '1655'
reading_time_minutes: 4
series: general
slug: revit_2019_sdk_apr
source_file: 1655_revit_2019_sdk_apr.md
tags:
- elements
- geometry
- parameters
- references
- revit-api
- sheets
- walls
- windows
title: Revit 2019 Sdk Apr
word_count: 885
---

### Installing the Revit 2019 SDK April Update
After the significant struggle I had
to [compile the initial release of the Revit 2019 SDK samples](http://thebuildingcoder.typepad.com/blog/2018/04/compiling-the-revit-2019-sdk-samples.html)
and [set up RvtSamples 2019](http://thebuildingcoder.typepad.com/blog/2018/04/rvtsamples-2019.html),
I am happy to report that installing and compiling the Revit 2019 SDK April 27 update is a lot easier:
- [Downloading the April 27 SDK update](#2)
- [Initial compilation – 41 warnings](#3)
- [Processor architecture mismatch suppressed – 5 warnings](#4)
- [Update reference to `RevitAPISteel.dll` – 3 warnings](#5)
- [Setting up `RvtSamples`](#6)
- [Updated `RvtSamples` download](#7)
#### Downloading the April 27 SDK Update
We are still waiting for the Revit 2019 SDK April 27 Update to appear in its proper location in
the [Revit Developer Centre](http://autodesk.com/developrevit)
at [autodesk.com/developrevit](http://autodesk.com/developrevit),
which nowadays points to a new location
at [www.autodesk.com/developer-network/platform-technologies/revit](https://www.autodesk.com/developer-network/platform-technologies/revit).
If you are in a hurry, you can grab it right now from
the [direct `REVIT_2019_SDK.msi` download link](http://download.autodesk.com/us/revit-sdk/REVIT_2019_SDK.msi),
which is [download.autodesk.com/us/revit-sdk/REVIT_2019_SDK.msi](http://download.autodesk.com/us/revit-sdk/REVIT_2019_SDK.msi).
The resulting Revit 2019 SDK update installer file dated April 27, 2018, is 375521280 bytes in size.
#### Initial compilation – 41 Warnings
I ran the installer and loaded the SDK solution `SDKSamples` into Visual Studio.
The first compilation worked pretty well right out of the box.
It reported [186 projects succeeded, 0 failed, 0 errors and 41 warnings](zip/revit_2019_sdk_samples_errors_warnings_9_1.txt).
Almost all the warnings are related to the processor architecture mismatch:
- Warning: There was a mismatch between the processor architecture of the project being built "MSIL" and the processor architecture of the reference "RevitAPI", "AMD64". This mismatch may cause runtime failures. Please consider changing the targeted processor architecture of your project through the Configuration Manager so as to align the processor architectures between your project and references, or take a dependency on references with a processor architecture that matches the targeted processor architecture of your project. RebarFreeForm
These warnings can easily be suppressed using
my [`DisableMismatchWarning.exe` command line utility](https://github.com/jeremytammik/DisableMismatchWarning)
to [recursively disable architecture mismatch warnings](http://thebuildingcoder.typepad.com/blog/2013/07/recursively-disable-architecture-mismatch-warning.html).
#### Eliminated Processor Architecture Mismatch Warnings – 5 Warnings
After running the DisableMismatchWarning utility,
the [number of warnings is reduced from 41 to 5](zip/revit_2019_sdk_samples_errors_warnings_9_2.txt).
- FrameBuilder: Could not find rule set file "Migrated rules for FrameBuilder.ruleset".
- GeometryCreation_BooleanOperation: Could not find rule set file "GeometryCreation_BooleanOperation.ruleset".
- ProximityDetection_WallJoinControl: Could not find rule set file "ProximityDetection_WallJoinControl.ruleset".
- SampleCommandsSteelElements: The referenced component 'RevitAPISteel' could not be found.
- SampleCommandsSteelElements: Could not resolve this reference. Could not locate the assembly "RevitAPISteel". Check to make sure the assembly exists on disk. If this reference is required by your code, you may get compilation errors.
I will ignore the first three for now.
The last two are more serious, of course.
#### Update Reference to RevitAPISteel.dll – 3 Warnings
The missing reference to the `RevitAPISteel.dll` .NET library assembly can be easily resolved by manually updating it to point to the existing DLL in the Revit executable folder:
- C:\Program Files\Autodesk\Revit 2019\RevitAPISteel.dll
That reduced
the [number of warning messages from 5 to 3](zip/revit_2019_sdk_samples_errors_warnings_9_3.txt),
which, as said, I will ignore:
- FrameBuilder: Could not find rule set file "Migrated rules for FrameBuilder.ruleset".
- GeometryCreation_BooleanOperation: Could not find rule set file "GeometryCreation_BooleanOperation.ruleset".
- ProximityDetection_WallJoinControl: Could not find rule set file "ProximityDetection_WallJoinControl.ruleset".
#### Setting up RvtSamples
With the SDK samples compiling successfully, and the number of warnings reduced to a tolerable number, I next set up RvtSamples to load all the external commands.
First of all, I added the add-in manifest and the RvtSamples input text file to the project for easier access and modification:
![RvtSamples project files](img/RvtSamples_project_files.png)
Next, I updated the paths in both of them to point to my SDK samples folder.
The original input text file still refers to \*C:/Revit Copernicus SDK/Samples/\*.
I replaced the backslashes to forward slashes for simpler regular expression editing and updated the paths to point to my installation in \*C:/a/lib/revit/2019/SDK/Samples/\*.
To test that all the external commands listed in the text file are found, I temporarily toggled the `testClassName` flag to true:
```csharp
bool testClassName = true; // jeremy
```
With that flag enabled, a number of warnings are issued:
- VB not loaded DeleteObject 63
- VB not loaded HelloRevit 84
- Tooltip missing PlacementOptions 147 fixed
- VB not loaded RotateFramingObjects 210
- VB not loaded MaterialProperties 805
- External command not found FabricationPartLayout 854
- Assembly missing CreateShared 917 removed
- Assembly missing CreateShared 924 removed
- VB not loaded SlabProperties 1008
- VB not loaded CreateBeamsColumnsBraces 1050
- VB not loaded StructuralLayerFunction 1267
- Assembly does not exist 1365
After some more twiddling, just the seven or eight VB problems remain, and I am satisfied:
![RvtSamples ribbon panel](img/RvtSamples_2019_april_27_update.png)
I mustn't forget to toggle off the debugging flag again...
#### Updated RvtSamples Download
For your convenience, here is my freshly
baked [RvtSamples_2019_april_update.zip archive file](/a/doc/revit/tbc/git/a/zip/RvtSamples_2019_april_update.zip).