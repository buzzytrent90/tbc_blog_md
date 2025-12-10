---
post_number: "0937"
title: "Compiling the Revit 2014 SDK"
slug: "compile_2014_sdk"
author: "Jeremy Tammik"
tags: ['elements', 'references', 'revit-api', 'selection', 'sheets', 'vbnet', 'views']
source_file: "0937_compile_2014_sdk.htm"
original_url: "https://thebuildingcoder.typepad.com/blog/0937_compile_2014_sdk.html"
---

### Compiling the Revit 2014 SDK

Here are some notes on the small issues I encountered compiling the Revit 2014 SDK:

- [Compile and install RevitLookup](#2)
- [Set up the Revit API assembly paths](#3)
- [First compilation run causes expected errors](#4)
- [Set up the RevitAddInUtility assembly path](#5)
- [PointCurveCreation Office reference](#6)
- [CancelSave RevitAddInUtility Reference](#7)
- [Set up the RvtSamples application](#8)
- [Fix errors in RvtSamples.txt](#9)
- [Download RvtSamples and RevitLookup](#10)

I already described this same process during the
[Revit 2013](http://thebuildingcoder.typepad.com/blog/2012/04/developer-center-and-sdk-update.html#2) timeframe.
Let's see if anything changed, or, better still, improved.

#### Compile and Install RevitLookup

My first step is always to compile, install and test RevitLookup.

This went completely smoothly, so I have nothing to report on that.

Please see [below](#9) for a link to the version I compiled.

#### Set up the Revit API Assembly Paths

Before the introduction of Revit Onebox, I used to make copies of the
[Revit API assemblies for the different flavours](http://thebuildingcoder.typepad.com/blog/2012/04/developer-center-and-sdk-update.html#12).

With the advent of Onebox, I opted for the simpler solution of using the Revit API DLLs path updater RevitAPIDllsPathUpdater.exe.
Simply launch it, enter the sample location and DLL folder, and let it do its job.
In my case, I entered the following paths:

- C:\a\lib\revit\2014\SDK\Samples\RevitAPIDllsPathUpdater.exe

- Sample location: C:\a\lib\revit\2014\SDK\Samples
- DLL folder: C:\Program Files\Autodesk\Revit Architecture 2014

It completes and reports that it "Replaced 169 files, skipped 0 files."

#### First Compilation Run Causes Expected Errors

With the correct assembly paths in place, it is time to open the Visual Studio solution SDKSamples2014.sln and compile the samples.

The first run reports
[Rebuild All: 166 succeeded, 3 failed, 0 skipped](zip/sdk_samples_2014_01.txt).

This no surprise, because there are some expected errors.

#### Set up the RevitAddInUtility Assembly Path

The first error is caused by the RevitAddInUtilitySample and says: "The type or namespace name 'Autodesk' could not be found (are you missing a using directive or an assembly reference?)"

The RevitAddInUtilitySample references the RevitAddInUtility assembly.
It is also located in the Revit API assembly path, but RevitAPIDllsPathUpdater.exe does not take it into account, so you have to open that project and set the assembly path manually instead:

![RevitAddInUtilitySample API reference](img/sdk_samples_2014_01.png)

In my case, the correct reference path to it is C:\Program Files\Autodesk\Revit Architecture 2014\RevitAddInUtility.dll.

#### PointCurveCreation Office Reference

The Massing PointCurveCreation sample references Microsoft.Office.Interop.Excel in order to interact with spreadsheets.

I have not set up Office on my virtual machine, so that caused an error saying "The type or namespace name 'Office' does not exist in the namespace 'Microsoft' (are you missing an assembly reference?)"

For a quick ad hoc solution to this, I simply unloaded this one project for the time being.

#### CancelSave RevitAddInUtility Reference

The next problem occurs in the Events CancelSave sample: "The type or namespace name 'RevitAddIns' does not exist in the namespace 'Autodesk' (are you missing an assembly reference?)"

Same resolution as [above](#5), set the RevitAddInUtility assembly path.

Wow!

That was it!

No more errors, just 345 warnings saying 'There was a mismatch between the processor architecture of the project being built "MSIL" and the processor architecture of the reference "RevitAPI, Version=2014.0.0.0, Culture=neutral, processorArchitecture=AMD64", "AMD64". This mismatch may cause runtime failures. Please consider changing the targeted processor architecture of your project through the Configuration Manager so as to align the processor architectures between your project and references, or take a dependency on references with a processor architecture that matches the targeted processor architecture of your project'.

I assume you can ignore those.

This was a smoother compilation than ever before.

#### Set up the RvtSamples Application

I always install RvtSamples to load all the other SDK samples if I ever want to test something in one of them.

It saves you from installing them one by one, which might prove a lengthy process, there being well over a hundred of them.

To do so, I first add two files to the RvtSamples project, its add-in manifest and its text file listing all the samples to load:

![Add manifest and sample list to RvtSamples project](img/sdk_samples_2014_02.png)

In the text file, I replace the samples path by my real installation folder, globally replacing "Z:\SDK2013\Samples\" by "C:\a\lib\revit\2014\SDK\Samples\".

At the end, I add placeholders for my two include files, for the Autodesk Developer Network ADN and The Building Coder sample collections:

```
##include C:\a\lib\revit\2014\adn\src\AdnSamples.txt
#include C:\a\lib\revit\2014\bc\BcSamples.txt
```

The ADN samples are commented out, because we have not completed their migration yet.

I already
[migrated The Building Coder samples to Revit 2014](http://thebuildingcoder.typepad.com/blog/2013/04/migrating-the-building-coder-samples-to-revit-2014.html),
though, so that include file is already active.

I need to add the RvtSamples assembly path to its add-in manifest and install that in the Revit Add-Ins folder, and we are set to go.

#### Fix errors in RvtSamples.txt

As usual, the list of samples to load specified by RvtSamples.txt is not perfectly set up.
Here are some of the add-ins causing errors on my system:

- RotateFramingObjects
- ProjectUnit (missing)
- GenericModelCreation
- ElementViewer
- PointCurveCreation (my fault)
- TraverseSystem
- CreateShared
- BarDescriptions (missing)
- StructuralLayerFunction

ProjectUnit is missing, presumably because the unit API changed in Revit 2014 and the sample has been removed.
It is still listed in RvtSamples.txt, and should be removed there as well.

Most of the others are caused by VB.NET samples listed in their 'bin' subfolder, whereas their assembly DLL really lives in 'bin/Debug', at least on my system, and vice versa, in the case of CreateShared.

After my first clean-up pass, the following still cause problems:

- GenericModelCreation
- PointCurveCreation (my fault)
- TraverseSystem
- CreateShared
- BarDescriptions (missing)
- StructuralLayerFunction

Again, these are almost all VB.NET samples.
Something strange going on with those.

There is a testing switch you can set in the RvtSamples source code, actually:

```
  bool testClassName = true;
```

Setting it to true turns up more errors:

- DeleteObject
- HelloRevit
- RotateFramingObjects
- MaterialProperties
- SlabProperties
- CreateBeamsColumnsBraces
- StructuralLayerFunction

I fixed some of these, but not all.

Anyway, I'll stop fixing this for today, though, because I have other things to do as well.

RvtSamples loads now and most of the samples are available:

![RvtSamples in Revit 2014](img/2014_rvtsamples.png)

#### Download RvtSamples and RevitLookup

Here is my current version of
[RvtSamples](zip/2014_RvtSamples.zip),
and also of
[RevitLookup](zip/2014_RevitLookup.zip),
which was missing from some intermediate versions of the SDK samples.

The RvtSamples package includes both the original and my modified add-in manifest and RvtSamples.txt sample list.
You can compere them to see the changes I applied, and add analogous changes of your own for any other samples that you wish to activate.

I hope this is of use to you.

This article was picked for TenLinks Daily by [www.caddigest.com](http://www.caddigest.com/current.htm).

![CADdigest](img/CADdigest_selection.jpeg)

Actually, taking a closer look at the CAD digest listing, I currently count 53 of The Building Coder articles listed there.
I was not aware of that before.
Surprise, surprise :-)