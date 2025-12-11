---
ai_optimized: true
author: Jeremy Tammik
complexity_score: 2.2
content_type: code_example
optimization_date: '2025-12-11T11:44:15.773082'
original_url: https://thebuildingcoder.typepad.com/blog/1324_revitlookup_python.html
post_number: '1324'
reading_time_minutes: 2
series: general
slug: revitlookup_python
source_file: 1324_revitlookup_python.htm
tags:
- elements
- python
- references
- revit-api
- walls
- windows
title: RevitLookup in Python Shell and Multiple Release Solution
word_count: 499
---

### RevitLookup in Python Shell and Multiple Release Solution

Here is some exciting news from Daren Thomas on
[RevitLookup](https://github.com/jeremytammik/RevitLookup) and the
[Revit Python Shell](https://github.com/architecture-building-systems/revitpythonshell).

The Python and Ruby shells came up a couple of times recently, and I also mentioned Daren's initial thoughts on making the RevitLookup snoop functionality easily accessible from within the interactive Python IDE:

- [Curved wall elevation profile implementation in Python](http://thebuildingcoder.typepad.com/blog/2015/04/curved-wall-elevation-profile-and-creator-class-update.html#6)
- [Live development](http://thebuildingcoder.typepad.com/blog/2015/05/live-development-and-a-share-bar.html#3)
- [Revit 2016 Python shell and RevitLookup incorporation](http://thebuildingcoder.typepad.com/blog/2015/05/copyelements-revit-2016-scalability-python-and-ruby-shells.html#4)
- [Revit 2016 Ruby shell](http://thebuildingcoder.typepad.com/blog/2015/05/copyelements-revit-2016-scalability-python-and-ruby-shells.html#5)

This idea has now come to fruition, and more easily than one might expect.

In Daren's own words:

The current version of RPS now includes a function 'lookup' in the startup script. Passing in an Element or an ElementId object will open up the "Snoop Objects" window if RevitLookup is installed. Otherwise, a message will be displayed directing the user to the RevitLookup GitHub repository.

I describe it in more detail in this discussion on [RevitLookup and RevitPythonShell](http://darenatwork.blogspot.ch/2015/05/revitlookup-and-revitpythonshell.html).

Also, here is a technique I found useful in the RPS project: I edit the .csproj file itself and change the way the Revit API assembly DLLs RevitAPI.dll and RevitAPIUI.dll are referenced:

```
  <ItemGroup Condition="'$(Configuration)' == 'Debug 2014'">
    <Reference Include="RevitAPI">
      <HintPath>..\RequiredLibraries\Revit2014\RevitAPI.dll</HintPath>
    </Reference>
    <Reference Include="RevitAPIUI">
      <HintPath>..\RequiredLibraries\Revit2014\RevitAPIUI.dll</HintPath>
    </Reference>
  </ItemGroup>
  <ItemGroup Condition="'$(Configuration)' == 'Debug 2015'">
    <Reference Include="RevitAPI">
      <HintPath>..\RequiredLibraries\Revit2015\RevitAPI.dll</HintPath>
    </Reference>
    <Reference Include="RevitAPIUI">
      <HintPath>..\RequiredLibraries\Revit2015\RevitAPIUI.dll</HintPath>
    </Reference>
  </ItemGroup>
  <ItemGroup Condition="'$(Configuration)' == 'Debug 2016'">
    <Reference Include="RevitAPI">
      <HintPath>..\RequiredLibraries\Revit2016\RevitAPI.dll</HintPath>
    </Reference>
    <Reference Include="RevitAPIUI">
      <HintPath>..\RequiredLibraries\Revit2016\RevitAPIUI.dll</HintPath>
    </Reference>
  </ItemGroup>
```

I then included the API files for each supported version in a RequiredLibraries folder.
When you change the configuration in Visual Studio (e.g., from "Debug 2014" to "Debug 2016"), the referenced assemblies also change (I think – you might need to reload Visual Studio) and compilation works just fine!

This technique can be extended to add conditional compilation (but I think that is already handled by the VS UI).

Anyway. I'd say this is a first stab at the
[RPS/RevitLookup collaboration](http://thebuildingcoder.typepad.com/blog/2015/05/copyelements-revit-2016-scalability-python-and-ruby-shells.html#4) we discussed a week or two ago and is quite useful already.

Many thanks to Daren for the good news!

Congratulations on getting it up and running with such minimal fuss!

The multi-version support you implemented looks very nice and useful too.