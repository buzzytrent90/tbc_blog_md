---
ai_optimized: true
author: Jeremy Tammik
complexity_score: 3.1
content_type: qa
optimization_date: '2025-12-11T11:44:14.530931'
original_url: https://thebuildingcoder.typepad.com/blog/0757_devcenter_sdk_update.html
post_number: '0757'
reading_time_minutes: 7
series: general
slug: devcenter_sdk_update
source_file: 0757_devcenter_sdk_update.htm
tags:
- elements
- filtering
- levels
- parameters
- references
- revit-api
- schedules
- sheets
- views
- walls
title: Developer Center and SDK Update
word_count: 1479
---

### Developer Center and SDK Update

I enjoyed the first really hot weekend this spring.
It was warming up seriously last week in Munich, and now it feels as if we were suddenly catapulted from a hesitant spring straight into summertime.
I also took some time to explore the developer centre and Revit 2013 SDK update, though.

The
[Revit Developer Center](http://www.autodesk.com/developrevit) has
been updated to a new layout.

The API and the available training material have grown so fast and voluminous that the old layout was getting hard to navigate.
I hope the updated site makes it easier still for you to quickly find exactly what you need.
Take a look and let us know what you think.

At the same time, the Revit SDK has been updated.
If you are using the SDK, I recommend that you immediately switch to the
[updated version](http://images.autodesk.com/adsk/files/revit2013sdk0.exe).

#### Revit 2013 Developer Guide

Some developers asked about the Revit 2013 developer guide, pointing out that it is not included in the SDK in PDF format as it was in previous releases.

The fact is that the one and only official version of the Revit API Developer Guide is the one provided online at
[wikihelp.autodesk.com/Revit/enu/2013](http://wikihelp.autodesk.com/Revit/enu/2013) >
[API Developer's Guide](http://wikihelp.autodesk.com/Revit/enu/2013/Help/00006-API_Developer%27s_Guide).

The developer guide has been
[available online](http://thebuildingcoder.typepad.com/blog/2011/05/wiki-api-help-view-event-and-structural-material-type.html#1) for
a year and is now 100% on wikihelp.
The PDF provided in previous releases was never as up-to-date as the wikihelp location.

Work on the 2013 version is still underway, **and** it is already in a completely usable state.

#### More New Revit 2013 SDK Samples

I recently presented an overview of the enhancements, new functionality and six
[new SDK samples](http://thebuildingcoder.typepad.com/blog/2012/03/new-revit-2013-sdk-samples.html) provided
by the
[Revit 2013 API](http://thebuildingcoder.typepad.com/blog/2012/03/revit-2013-and-its-api.html):

- [ModelessForm\_ExternalEvent and ModelessForm\_IdlingEvent (ModelessDialog)](http://thebuildingcoder.typepad.com/blog/2012/03/new-revit-2013-sdk-samples.html#1)- [ProgressNotifier (Events)](http://thebuildingcoder.typepad.com/blog/2012/03/new-revit-2013-sdk-samples.html#2)- [RoutingPreferenceTools](http://thebuildingcoder.typepad.com/blog/2012/03/new-revit-2013-sdk-samples.html#3)- [UIAPI](http://thebuildingcoder.typepad.com/blog/2012/03/new-revit-2013-sdk-samples.html#4)- [WorkThread (MultiThreading)](http://thebuildingcoder.typepad.com/blog/2012/03/new-revit-2013-sdk-samples.html#5)

The updated version includes three more ones in addition to those:

- [DisableCommand](#21)- [ScheduleCreation](#22)- [StairsAutomation](#23)

These three all have to do with the
[enhanced add-in integration](http://thebuildingcoder.typepad.com/blog/2012/03/revit-2013-and-its-api.html#2) supported
by Revit 2013, and just like several of the previous samples, they were demonstrated at the DevDays 2011 conferences by the Revit UI API demo sample application, which is available for ADN members together with all other DevDays presentations on the ADN Extranet at
[adn.autodesk.com](http://adn.autodesk.com) under "Events".

The only noteworthy other changes in the updated version are the inclusion of the three new samples plus the UIAPI one to the SDKSamples2013.sln Visual Studio solution file, and the addition of these four plus CompoundStructure to the SamplesReadMe.htm documentation file.

Well, OK, the What's New section of the Revit API help file RevitAPI.chm has been revamped and looks nicer, as well.

#### Compiling and Installing the Revit SDK

To compile the new version of the SDK, I had to go through the same steps as always:

1. [Compile and install RevitLookup](#11)- [Compile the SDK samples](#12)- [Install RvtSamples](#13)

**1.** Compile and install RevitLookup.
Nothing special here; just open the project file, compile, edit the add-in manifest, and copy it to the add-ins folder.

**2.** Compile all the SDK samples using the SDKSamples2013.sln Visual Studio solution.
This requires updating the Revit API assembly DLL paths in the project files to whatever version of Revit I have installed.
In my case, I still only have the Revit Quasar RP version installed, whereas the project files refer to Revit Architechture 2013, Revit MEP 2013 and Revit Structure 2013.

One way to do this is to use the RevitAPIDllsPathUpdater.exe provided in the SDK Samples folder.

I prefer not to modify the HintPath tags in the project files at all, but rather to simply create dummy directory structures for the three products under the Program Files > Autodesk root folder and copy the required DLLs into a Program subfolder in each of them, ending up with the following seven copied assemblies:

- C:\Program Files\Autodesk\
  - Revit Architecture 2013\Program\
    - RevitAddInUtility.dll- RevitAPI.dll- RevitAPIUI.dll- Revit MEP 2013\Program\
      - RevitAPI.dll- RevitAPIUI.dll- Revit Structure 2013\Program\
        - RevitAPI.dll- RevitAPIUI.dll

The copy of RevitAddInUtility.dll in the architectural dummy folder is only required to compile the ExternalCommand
[RevitAddInUtilitySample](http://thebuildingcoder.typepad.com/blog/2010/04/revitaddinutility.html) SDK
sample.
By the way, the location of this reference is not updated by RevitAPIDllsPathUpdater.exe, like the other two main Revit API assembly DLLs.
It would help if the source code for this utility were available.

I also had to change the ProgressNotifier sample Revit API assembly reference paths, which were hardwired to 64 bits.

Once that was done, the SDKSamples2013.sln solution compiled all SDK samples with no problems.

**3.** Set up RvtSamples to load all SDK samples into Revit.

To achieve this, I had to set up and install RvtSamples and make a few fixes to the text file it reads to specify the locations of all the samples to load;
here is my slightly updated version of
[RvtSamples.txt](zip/RvtSamples_2013_rp_update.txt).

All in all this is a very clean update with only minor changes required to set it up on my system.

Now let's take a look at the three newly introduced SDK samples.

#### DisableCommand

This sample is an external application, not a command, so you cannot test it by simply invoking it via RvtSamples.
You have to copy its add-in manifest to the add-ins folder.

Once installed, it disables a command by replacing its implementation with a simple popup message.
Specifically, it overrides the Design Options command, thus prevent users from accessing it.
It performs the following steps:

- A RevitCommandId is found to match the journal name of the command.- An AddInCommandBinding is created for this command id.- An alternate implementation is provided for the command binding.

To run it, once installed, simply start up Revit, open or create a project, and choose the Design Options button on the Manage tab.
A popup explains that the command is disabled:

![DisableCommand message](img/2013_DisableCommand_message.png)

#### ScheduleCreation

This sample demonstrates how to create a wall category view schedule and display its data on a sheet.
It performs the following steps:

- Create an empty wall schedule.- Add some schedule fields by schedulable field.- Create new schedule filter.- Create schedule sorting/grouping field.- Display a sheet containing the wall schedule.

Run it as follows:

1. Open Revit and create a new project based on the default Revit template.- Manually create a couple of walls with different types.- Run this command, for instance via RvtSamples > Views > ScheduleAPI.- A view schedule of wall category and a sheet to show its data are created.

Here is a trivial sample model to test it in:

![ScheduleCreation sample walls](img/2013_ScheduleCreation_walls.png)

This is the resulting schedule:

![ScheduleCreation generated schedule](img/2013_ScheduleCreation_schedule.png)

#### StairsAutomation

This is a slightly more complex sample that creates a series of stairs, stairs runs and stairs landings configurations based upon predefined hard-wired rules and parameters.
It provides an example of how to create and populate stairs elements, e.g.

- Use of StairsEditMode- Creation of standard stairs runs- Creation of sketched stairs runs- Creation of sketched landings

It also implements an extensible structure demonstrating how runs and landings creation can be combined into different stairs configurations.

To test this, start up Revit and open the 'Stairs automation.rvt' sample model.
Execute the command, e.g. using RvtSamples > Elements > Stairs automation.
Each time the command is run, a different predefined stair configuration is created:

1. Single straight stair run

![StairsAutomation straight run](img/2013_StairsAutomation_1.png)

2. Stairs/landing combination up to level 2

![StairsAutomation landing combination](img/2013_StairsAutomation_2.png)

3. Multi-span stairs/landing combination up to level 3

![StairsAutomation landing combination](img/2013_StairsAutomation_3.png)

4. Single curved stairs run

![StairsAutomation curved run](img/2013_StairsAutomation_4.png)

5. Curve stairs run > 360 degrees

![StairsAutomation 360 degrees curved run](img/2013_StairsAutomation_5.png)

As said, a clean and interesting update, and I highly recommend you grab and install it.