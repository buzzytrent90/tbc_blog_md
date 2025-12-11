---
ai_optimized: true
author: Jeremy Tammik
complexity_score: 2.9
content_type: tutorial
optimization_date: '2025-12-11T11:44:15.013047'
original_url: https://thebuildingcoder.typepad.com/blog/0968_architecture_mismatch.html
post_number: 0968
reading_time_minutes: 4
series: general
slug: architecture_mismatch
source_file: 0968_architecture_mismatch.htm
tags:
- levels
- references
- revit-api
- windows
title: Processor Architecture Mismatch Warning and Key Hook
word_count: 745
---

### Processor Architecture Mismatch Warning and Key Hook

In the past ten days, I migrated
[The Building Coder samples](http://thebuildingcoder.typepad.com/blog/2013/06/removing-deprecated-api-compilation-warnings.html),
[ADN training labs](http://thebuildingcoder.typepad.com/blog/2013/06/migrating-the-adn-training-labs-to-revit-2014.html),
[MEP](http://thebuildingcoder.typepad.com/blog/2013/06/the-adn-sample-adnrme-for-revit-mep-2014.html) and
[Structural](http://thebuildingcoder.typepad.com/blog/2013/06/adn-training-material-for-revit-structure-2014.html) sample material from Revit 2013 to 2014.

In all of these migrations, I lamented the processor architecture mismatch warning MSB3270 saying
*There was a mismatch between the processor architecture of the project being built "MSIL" and the processor architecture of the reference "RevitAPI, Version=2014.0.0.0, Culture=neutral, processorArchitecture=x86", "AMD64". This mismatch may cause runtime failures. Please consider changing the targeted processor architecture of your project through the Configuration Manager so as to align the processor architectures between your project and references, or take a dependency on references with a processor architecture that matches the targeted processor architecture of your project.*

This behaviour is in new in .NET 4.5 and meant to be "helpful",
[according to Microsoft](http://connect.microsoft.com/VisualStudio/feedback/details/759796/mismatch-between-the-processor-architecture-of-the-project-being-built-msil-and-the-processor-architecture-of-the-reference).
Unfortunately, it is not all that helpful for us Revit add-in developers.

Of course, there is some level of "danger" if you reference an architecture-specific DLL with an "AnyCPU" one.
But that's exactly what Revit provides in offering a .NET API to its native codebase.

Here are a couple of possibilities how the owner of an add-in project can deal with this warning:

1. Change it to be 32-bit or 64-bit specific. This is not an option for samples where they should build against either architecture, or for add-ins which are intended to work anywhere in Revit.
2. Use the workaround described in the link,
   [setting ResolveAssemblyWarnOrErrorOnTargetArchitectureMismatch=false](http://connect.microsoft.com/VisualStudio/feedback/details/759796/mismatch-between-the-processor-architecture-of-the-project-being-built-msil-and-the-processor-architecture-of-the-reference).
   This can be done either by adding a property into your project file, or by setting an environment variable, or by using /p if you're running msbuild.exe.
3. Ignore it.

Here is
[Andrew Potts' take on the issue](http://andypottsblog.blogspot.ch/2012/08/installing-net-45-or-vsnet-2012-causes.html).

For The Building Coder samples, I am aiming for option 2, setting the following property in the project file to turn the warning off:

```
<PropertyGroup>
  <ResolveAssemblyWarnOrErrorOnTargetArchitectureMismatch>
    None
  </ResolveAssemblyWarnOrErrorOnTargetArchitectureMismatch>
</PropertyGroup>
```

I did not see any hint on how to add this to the project file through the Visual Studio IDE, so I simply edited BuildingCoder.csproj as a text file using Notepad and added the lines above as a new property group just above all existing ones.

It works fine, the offending warning is no longer shown, and I am down to just the
[two expected warnings](zip/bc_migr_2014_i.txt) about
use of the obsolete TitleBlocks property and FindReferencesWithContextByDirection method.

Here is the updated
[version 2014.0.100.4](zip/bc_14_100_4.zip) of
The Building Coder samples, in case you would like examine the resulting project file yourself directly.

#### Using a Windows Keyboard Hook to Handle Keys in a Revit Add-in

My fellow ADN DevTech engineer
[Augusto Goncalves](http://adndevblog.typepad.com/aec/augusto-goncalves.html)
just published a nice sample showing how to
[capture the Escape key](http://adndevblog.typepad.com/aec/2013/06/capture-escape-key-on-revit.html),
or any other keyboard interaction, for that matter, by setting a Windows hook using the Win32 SetWindowsHookEx API method in the add-in OnStartup and OnShutdown calls.

It is actually a follow-up of a previous post by
[Xiaodong Liang](http://adndevblog.typepad.com/manufacturing/xiaodong-liang.html)
for handling the Tab, Enter and Escape keys in a modeless form displayed by an
[Inventor add-in](http://adndevblog.typepad.com/manufacturing/2012/08/tab-key-not-working-in-modeless-form-displayed-by-add-in.html).

This kind of interaction can be useful e.g. to control or interrupt a long Revit process, either using input from a modeless form or just a keyboard hit with no form at all being displayed.

Thank you Augusto for this useful hint!