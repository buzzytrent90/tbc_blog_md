---
ai_optimized: true
author: Jeremy Tammik
complexity_score: 2.7
content_type: qa
optimization_date: '2025-12-11T11:44:13.399347'
original_url: https://thebuildingcoder.typepad.com/blog/0121_install_2010_sdk.html
post_number: '0121'
reading_time_minutes: 3
series: general
slug: install_2010_sdk
source_file: 0121_install_2010_sdk.htm
tags:
- family
- revit-api
title: Installing the Revit 2010 SDK
word_count: 599
---

### Installing the Revit 2010 SDK

#### Compiling the SDK samples

As discussed previously, you can
[use SDKSamples2010.sln to compile](http://thebuildingcoder.typepad.com/blog/2008/09/the-sdk-samples.html)
all the samples in one fell swoop.
The various sample projects in this solution expect you to have all three flavours of Revit installed in the default locations, otherwise compilation will fail for some of the samples.

So far, I installed only Revit Architecture 2010, not MEP or Structure.
In order to easily compile all the samples anyway, I simply copied the same RevitAPI.dll to the other two locations, so I now have three copies of it on my hard disk:

- C:\Program Files\Autodesk Revit Architecture 2010\Program\RevitAPI.dll
- C:\Program Files\Autodesk Revit MEP 2010\Program\RevitAPI.dll
- C:\Program Files\Autodesk Revit Structure 2010\Program\RevitAPI.dll

Once this was done, I was immediately able to compile all the samples in one go using SDKSamples2010.sln.

I noticed one little omission in the solution file, which you will notice if you try to run the Ribbon sample: the Ribbon sample commands are implemented in a separate assembly using the project file AddInCommands.csproj located in a subdirectory of the Ribbon one, and I had to add that to SDKSamples2010.sln myself.

#### Loading the SDK samples

I use the Revit SDK sample external application RvtSamples to
[load all Revit SDK external command samples](http://thebuildingcoder.typepad.com/blog/2008/09/loading-sdk-sam.html).

To set this up for Revit 2010, I performed the following steps:

- I edited RvtSamples.txt in the SDK/Samples/RvtSamples folder and replaced Z:\SDK2010\Samples by my local SDK samples path.
- I compiled the RvtMgdDbg for 2010.
- I added an ExternalApplications section to Revit.ini in the Revit Program folder, with entries for the external applications RvtMgdDbg and RvtSamples:

```
[ExternalApplications]
EACount=2

EAAssembly1=C:\Program Files\Autodesk Revit Architecture 2010\Program\RvtMgdDbg.dll
EAClassName1=RvtMgdDbg.App

EAAssembly2=C:\a\lib\revit\2010\SDK\Samples\RvtSamples\CS\RvtSamples.dll
EAClassName2=RvtSamples.Application
```

That's it, I am done and can now start up Revit.
The Add-Ins menu item is available and the corresponding panel displayed even in zero document state:

![RvtMgdDbg and RvtSamples in Add-Ins panel](img/2010_RvtSamples.gif)

RvtMgdDbg is up and running in its own panel:

![RvtMgdDbg Add-Ins panel](img/RvtMgdDbg_panel.png)

RvtSamples now displays the samples sorted by category only, there are no longer any multiple menu hierarchies by various different classifications.
Even though it is displayed in the zero document state, the menu entries are not active until a document has been opened:

![RvtSamples pulldown buttons](img/2010_RvtSamples_2.gif)

#### Updating the ADN training material

The next thing I am interested in is updating the Revit API introduction labs and all the rest of our training material to 2010.
One looming deadline is the upcoming
[webcast on the Revit API](http://usa.autodesk.com/adsk/servlet/item?siteID=123112&id=6364883)
on April 29th, one of our ADN
[training classes](http://www.adskconsulting.com/adn/cs/api_course_sched.php),
which will require this material.

By the way, our webcast plans for this year are more elaborate than in previous years.
We are thinking of holding this first webcast to cover the basics and discuss new API areas, and following it up with several dedicated sessions focusing in more depth on various areas affected by the new API functionality, such as:

- Family editor user interface features
- Family API
- Conceptual design API and form creation
- Revit MEP API

I'll keep you posted as these plans mature.