---
ai_optimized: true
author: Jeremy Tammik
complexity_score: 2.0
content_type: qa
optimization_date: '2025-12-11T11:44:13.776720'
original_url: https://thebuildingcoder.typepad.com/blog/0342_visual_studio_2010.html
post_number: '0342'
reading_time_minutes: 2
series: general
slug: visual_studio_2010
source_file: 0342_visual_studio_2010.htm
tags:
- revit-api
title: Debugging with Visual Studio 2010 and RvtSamples.txt Update
word_count: 412
---

### Debugging with Visual Studio 2010 and RvtSamples.txt Update

Rod Howarth just published a description of a problem
[debugging a Revit 2011 add-in with Visual Studio 2010](http://blog.rodhowarth.com/2010/04/revit-api-visual-studio-2010-and-revit.html).
He found a related thread on the Microsoft Connect Visual Studio discussion forum describing the
[same issue with Revit 2010](http://connect.microsoft.com/VisualStudio/feedback/details/487949/debugging-external-application) and
presenting a solution for it:

The issue that you are encountering is that Visual Studio 2010 is launching Revit under the default debugger (v4.0) but since you are building a project against the 2.0 framework (3.5 runs on the CLR 2.0) in order to correctly debug your Revit plugin, Visual Studio must launch Revit under the 2.0 debugger, not the 4.0.

You can either launch Revit and manually attach to the Revit process with your Revit plugin open (the debugger will automatically attach with the correct engine once the process has already started), or you can add a <supportedRuntime> attribute to the <startup> section of the Revit.exe.config file that will allow Visual Studio to determine which debugger should be attached:

As Rod says, the workaround is to edit your Revit.exe.config file which is located in the same directory as Revit.exe and add the following just before the </configuration> part of it:

```
<startup>
<supportedRuntime version="v2.0.50727" />
</startup>
```

For more details, please refer to
[Rod's post](http://blog.rodhowarth.com/2010/04/revit-api-visual-studio-2010-and-revit.html).

#### RvtSamples.txt Update

In a separate thread, Rod also mentioned a problem loading the version of RvtSamples.txt included with the Revit 2011 SDK.
This is the text file read by the
[RvtSamples](http://thebuildingcoder.typepad.com/blog/2008/09/loading-sdk-sam.html)
SDK sample application
(for [2010](http://thebuildingcoder.typepad.com/blog/2009/05/porting-the-building-coder-samples.html))
to load all the other Revit SDK sample commands as well as potentially process
[include files](http://thebuildingcoder.typepad.com/blog/2008/11/loading-the-building-coder-samples.html) to
load an additional set of own personal commands.
I ran into the same issues Rod describes, so I thought it worthwhile to publish
[my current version of RvtSamples.txt](http://thebuildingcoder.typepad.com/files/rvtsamples.txt)
for Revit 2011 to save others the bother of having to explore what fixes need to be made.