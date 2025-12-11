---
ai_optimized: true
author: Jeremy Tammik
complexity_score: 2.9
content_type: qa
optimization_date: '2025-12-11T11:44:15.215727'
original_url: https://thebuildingcoder.typepad.com/blog/1059_multi_version_wizard.html
post_number: '1059'
reading_time_minutes: 4
series: general
slug: multi_version_wizard
source_file: 1059_multi_version_wizard.htm
tags:
- csharp
- references
- revit-api
- views
- windows
title: Multi-Version Visual Studio Revit Add-In Wizard
word_count: 786
---

### Multi-Version Visual Studio Revit Add-In Wizard

Developers often ask how to support multiple versions of a Revit add-in from the same codebase.

Many have implemented solutions for this using various Visual Studio project settings.

Alexander Ignatovich of
[Investicionnaya Venchurnaya Companiya](http://www.iv-com.ru) went
one step further and implemented a version of the
[Visual Studio Revit add-in wizard](http://thebuildingcoder.typepad.com/blog/about-the-author.html#5.20) that
automatically generates an add-in skeleton supporting both Revit 2013 and 2014 in the same Visual Studio project.

He is very kindly sharing is with us here, saying:

> I have one another thing I want to share.
> I found very nice way to support both Revit 2013 and Revit 2014.
> I created a project wizard, based on the
> [Visual Studio Revit 2014 add-in wizards](http://thebuildingcoder.typepad.com/blog/2013/05/add-in-wizards-for-revit-2014-1.html).
>
> Look at the csproj file. There are four build configurations in it:
>
> - Debug
> - Release
> - Debug2014
> - Release2014
>
> First, I used choose tags to determine what reference should be used in the build configuration.
> Unfortunately, there are some problems with $(ProgramW6432), so I replaced it with C:\Program files.
>
> Secondly, I defined a VERSION2014 compilation constant and add it as an example in Command.cs.
>
> Thirdly, I add copying files to 2013 add-ins folder in post-build event.
> Unfortunately, I can't use choose tag there – it does not work – so the add-in files are copied to both 2013 and 2014 add-ins folders.
>
> Lastly, StartProgram tags contain the proper Revit executable path for each build configuration.

Many thanks to Alexander for analysing, setting up and sharing this!

Here is an overview of the affected files in the highly recommended
[kdiff3 file comparison tool](http://kdiff3.sourceforge.net):

![Modified files](img/wiz_ai_1.png)

The differences in Command.cs are purely for testing purposes, reporting what release we compiled for in the Visual Studio debug output window:

![Compiler pragmas](img/wiz_ai_2.png)

The differences in the main template file are pure syntactic sugar, affecting only the user interface display strings.

The only significant changes are the ones that Alexander describes above in the C# project file.

Here is Alexander's
[Revit2014AddinWizardCs3Ai.zip](zip/Revit2014AddinWizardCs3Ai.zip) for
you to explore in detail for yourself.

Please note that this version of the wizard lacks the ResolveAssemblyWarnOrErrorOnTargetArchitectureMismatch tag that I added to suppress the
[processor architecture mismatch warning](http://thebuildingcoder.typepad.com/blog/2013/06/processor-architecture-mismatch-warning.html) when
compiling for Revit 2014.

You could always post-process the resulting project files using my tool to
[recursively disable the architecture mismatch warning](http://thebuildingcoder.typepad.com/blog/2013/07/recursively-disable-architecture-mismatch-warning.html).

#### Support for Revit Flavours and Additional Assembly References

Alexander added some more details on the CSPROJ file editing:

I think it might be useful to briefly describe \*.csproj file, maybe only the "Reference Include" tags for Revit API assemblies.

I specify an absolute path, because my Visual Studio 2010 sometimes does not correctly change the project configuration.

One issue with the references is that everybody has installed a different flavour of Revit.
I use the Revit Building Design Suite, which is located in `C:\Program Files\Autodesk\Revit 2014` by default, other people may use Revit Architecture, Revit Structure or Revit MEP.
The default location of Revit Architecture is `C:\Program Files\Autodesk\Revit Architecture 2014`, and if there is no Building Design Suite on developers computer, the wizard template files need to be edited, replacing the reference paths in \*.csproj.

Furthermore, if you want to reference additional Revit API assemblies, you can do so by adding them to the csproj file.
For example, if I want to add the Revit UIFramework assembly, I can open \*.csproj in an external editor – my choice is
[Notepad++](http://notepad-plus-plus.org) –
and add the Revit 2014 version to the Choose - When - ItemGroup tag:

```
  <Reference Include="UIFramework">
    <HintPath>C:\Program Files\Autodesk\Revit 2014\UIFramework.dll</HintPath>
    <Private>False</Private>
  </Reference>
```

Similarly, add the Revit 2013 version to the Choose - Otherwise - ItemGroup tag:

```
  <Reference Include="UIFramework">
    <HintPath>C:\Program Files\Autodesk\Revit 2013\Program\UIFramework.dll</HintPath>
    <Private>False</Private>
  </Reference>
```

Here is what it looks like:

![Additional UIFramework assembly reference](img/wiz_ai_3.png)

#### Today is the Last Day

Before closing, let me point out that we are reaching the end of another week, and today is the
[deadline for submitting the handouts](http://auspeaker.wordpress.com/2013/11/14/upload-your-class-handouts-and-other-materials-by-november-15) for
my Autodesk University classes.
So I will do just that.

Let me also wish you a wonderful weekend.