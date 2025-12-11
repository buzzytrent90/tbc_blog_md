---
ai_optimized: true
author: Jeremy Tammik
complexity_score: 1.9
content_type: qa
optimization_date: '2025-12-11T11:44:14.961755'
original_url: https://thebuildingcoder.typepad.com/blog/0944_addin_wizard_2014.html
post_number: 0944
reading_time_minutes: 4
series: general
slug: addin_wizard_2014
source_file: 0944_addin_wizard_2014.htm
tags:
- csharp
- references
- revit-api
- schedules
- vbnet
- views
- windows
title: Add-In Wizards for Revit 2014
word_count: 878
---

### Add-In Wizards for Revit 2014

I updated my Visual Studio Revit add-in wizards and am taking the time to publish them today, which is
[Ascension Day](http://en.wikipedia.org/wiki/Ascension_Day) and
a holiday in Neuchâtel.

That also gives me some extra time to prepare for the June 3-4 Tech Summit presentation of my
[cloud-based round-trip 2D Revit model editing project](http://thebuildingcoder.typepad.com/blog/2013/03/cloud-mobile-extensible-storage-data-use-in-schedules.html#3).

The presentation must be pre-submitted by May 20, and a full recording by May 27, so pressure is rising.
I completed the implementation and still want to catch up on documenting it here on the blog as well.
You will be happy to hear that the auto-updating functionality using the Idling event now works fine.

#### Old and New Wizardry

I am starting to create numerous Revit 2014 add-ins now for quick testing purposes.
As soon as that stage is reached, I immediately update my Visual Studio Revit C# add-in wizard.

The 2014 version generates the same boiler-plate code as the
[Revit 2013](http://thebuildingcoder.typepad.com/blog/2012/04/add-in-wizard-for-revit-2013.html) one
did, which can be simply deleted if not needed.

Last time around, I egoistically did not implement the
[VB version](http://thebuildingcoder.typepad.com/blog/2012/06/update-api-assembly-references-and-wizards.html#4) until later.

This time though, better realising how much appreciated it is, I took care of that as well, right up front.

It is important to understand how easy it is to modify the wizards for your own needs, and make copies with variations to support different requirements.

Here is an overview of previous explanations of various aspects that also show how create your own flavours:

- [Original introduction, benefits, and usage example](http://thebuildingcoder.typepad.com/blog/2010/07/devtv-addin-templates.html) for C# and VB.- Personalised
    [minimal C# version](http://thebuildingcoder.typepad.com/blog/2010/10/revit-2011-devtv.html#2) for Revit 2011.- A short additional
      [usage note](http://thebuildingcoder.typepad.com/blog/2010/12/snow-and-woe-with-manifest-files.html).- [64-bit versions](http://thebuildingcoder.typepad.com/blog/2011/01/automate-designoption-and-64-bit-add-in-templates.html#2) for C# and VB.- Support for the
          [Revit 2012 API](http://thebuildingcoder.typepad.com/blog/2011/04/visual-studio-add-in-wizards-for-revit-2012.html) for
          C# and VB.- Updated C# and VB versions placing
            [assembly DLL alongside add-in manifest](http://thebuildingcoder.typepad.com/blog/2011/10/product-and-add-in-wizard-updates.html#3) and
            including other changes.- [Revit 2013 C#](http://thebuildingcoder.typepad.com/blog/2012/04/add-in-wizard-for-revit-2013.html) version including more skeleton code.
            - [Revit 2013 VB](http://thebuildingcoder.typepad.com/blog/2012/06/update-api-assembly-references-and-wizards.html#4) version.

#### Installing to non-x86 Program Files

I had a new issue to struggle with this time around, mentioned in the list above as well, dealing with 64-bit systems.

My Windows 7 virtual machine is a 64-bit system.

The $(ProgramFiles) used in the Visual Studio project template file automatically resolves that to "C:\Program Files (x86)", which is the wrong location.

Revit 2014 is installed in "C:\Program Files" with no x86 suffix.

Researching this, I found a resolution described and explained in this discussion on
[MSBuild and $(ProgramFiles) issue with 32/64 bits](http://stackoverflow.com/questions/7066910/msbuild-and-programfiles-issue-with-32-64-bits).

Unfortunately, that did not work at all.
MSBuildExtensionsPath64 is not recognised in the Visual Studio wizard, at least in my simple use of it.

In the end, I simply checked what system variables are actually set on my system and found ProgramW6432 instead.
That works fine.

Now both the C# and the VB versions work perfectly for me.

#### Revit Add-in Wizard Usage

I can simply install the wizard zip files in the appropriate locations, start up Visual Studio, create a new C# or VB Revit add-in project using the wizard default settings, and immediately hit F5 to start up Revit.exe in the debugger.

Revit is started up, my add-in is automatically loaded, and I am able to click on my new external command in the External Tools menu to test it immediately without having entered even one single byte of code myself.

The new command even executes in zero document state, although the default external command skeleton implementation throws an exception trying to access a property on the current UI document, which is null.
It shows you that everything is working correctly right away, though.

#### Revit 2014 C# and VB Add-in Wizard Download

So, what are the appropriate locations, then?
And where are the zip files?

Right here!

To install, simply copy the zip file of your choice to the matching Visual Studio project template folder in your local file system:

- [Revit2014AddinWizardCs.zip](zip/Revit2014AddinWizardCs.zip) – copy to

  [My Documents]\Visual Studio 2010\Templates\ProjectTemplates\Visual C#- [Revit2014AddinWizardVb.zip](zip/Revit2014AddinWizardVb.zip) – copy to

    [My Documents]\Visual Studio 2010\Templates\ProjectTemplates\Visual Basic

I hope you find this useful and look forward to your suggestions for improvement.