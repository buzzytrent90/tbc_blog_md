---
post_number: "1151"
title: "Add-In Wizards for Revit 2015"
slug: "addin_wizard_2015"
author: "Jeremy Tammik"
tags: ['csharp', 'references', 'revit-api', 'vbnet', 'views']
source_file: "1151_addin_wizard_2015.htm"
original_url: "https://thebuildingcoder.typepad.com/blog/1151_addin_wizard_2015.html"
---

### Add-In Wizards for Revit 2015

I updated my Visual Studio Revit add-in wizards for Revit 2015.

The 2015 version generates the same boilerplate code as the reliable old
[Revit 2014](http://thebuildingcoder.typepad.com/blog/2013/05/add-in-wizards-for-revit-2014-1.html) one and its
[update](http://thebuildingcoder.typepad.com/blog/2013/06/sun-direction-shadow-calculation-and-wizard-update.html#2) did,
which can be simply deleted if not needed.

I selflessly also implemented and tested the Visual Basic version right away.

#### Revit Add-in Wizard Customisation

As I keep pointing out, it is important to understand how easy it is to modify the wizards for your own needs, and make copies with variations to support different requirements.

Here is an overview of previous explanations of various aspects that also show how to create your own flavours:

- [Original introduction, benefits, and usage example](http://thebuildingcoder.typepad.com/blog/2010/07/devtv-addin-templates.html) for C# and VB.- Personalised
    [minimal C# version](http://thebuildingcoder.typepad.com/blog/2010/10/revit-2011-devtv.html#2) for Revit 2011.- A short additional
      [usage note](http://thebuildingcoder.typepad.com/blog/2010/12/snow-and-woe-with-manifest-files.html).- [64-bit versions](http://thebuildingcoder.typepad.com/blog/2011/01/automate-designoption-and-64-bit-add-in-templates.html#2) for C# and VB.- Support for the
          [Revit 2012 API](http://thebuildingcoder.typepad.com/blog/2011/04/visual-studio-add-in-wizards-for-revit-2012.html) for
          C# and VB.- Updated C# and VB versions placing
            [assembly DLL alongside add-in manifest](http://thebuildingcoder.typepad.com/blog/2011/10/product-and-add-in-wizard-updates.html#3).- [Revit 2013 C#](http://thebuildingcoder.typepad.com/blog/2012/04/add-in-wizard-for-revit-2013.html) version including more skeleton code.
            - [Revit 2013 VB](http://thebuildingcoder.typepad.com/blog/2012/06/update-api-assembly-references-and-wizards.html#4) version.
            - C# and VB
              [add-in wizards for Revit 2014](http://thebuildingcoder.typepad.com/blog/2013/05/add-in-wizards-for-revit-2014-1.html).
            - [Wizard update](http://thebuildingcoder.typepad.com/blog/2013/06/sun-direction-shadow-calculation-and-wizard-update.html#2)
              suppressing the processor architecture mismatch warning and referring to Revit Onebox instead of the architectural flavour.
            - Wizard generating a
              [multi-version add-in](http://thebuildingcoder.typepad.com/blog/2013/11/multi-version-visual-studio-revit-add-in-wizard.html) supporting
              both Revit 2013 and 2014.

Just like for Revit 2014, the Revit 2015 add-in wizard uses the
[$(ProgramW6432)](http://thebuildingcoder.typepad.com/blog/2013/05/add-in-wizards-for-revit-2014-1.html#2) Visual Studio project template variable to determine where to locate the Revit API assembly files for referencing and the Revit.exe executable for debugging.

#### Revit Add-in Wizard Usage

Simply install the wizard zip files you need in the appropriate locations, start up Visual Studio, create a new C# or VB Revit add-in project using the wizard default settings, and immediately hit F5 to start up Revit.exe in the debugger.

The wizards perform the following functions for C# and VB, respectively:

- Generate the skeleton source code, Visual Studio solution and add-in manifest.
- On successful build, copy the add-in assembly DLL and add-in manifest to the Revit add-in folder, thus automatically installing it for Revit to pick up and load.
- Define the add-in debugging settings to start up the Revit.exe program.

Therefore, immediately after the initial add-in creation, you can immediately launch the debugger.
The add-in is compiled, Revit is started up, the add-in is loaded, you can select the new external command in the External Tools menu, launch and test it immediately without entering one single further keystroke yourself.

The new command even executes in zero document state, although the default external command skeleton implementation throws an exception trying to access a property on the current UI document, which is null.
It shows you that everything is working correctly right away, though.

#### Wizard Download and Installation

The appropriate locations to install the wizards for Visual Studio to pick them up are language dependent.

Copy the zip file of your choice to the matching Visual Studio project template folder in your local file system:

- C# – copy [Revit2015AddinWizardCs1.zip](zip/Revit2015AddinWizardCs1.zip) to

  [My Documents]\Visual Studio 2012\Templates\ProjectTemplates\Visual C#- Visual Basic – copy [Revit2015AddinWizardVb1.zip](zip/Revit2015AddinWizardVb1.zip) to

    [My Documents]\Visual Studio 2012\Templates\ProjectTemplates\Visual Basic

I hope you find this useful and look forward to hearing any suggestions for improvement you come up with.

Better still, implement them yourself and let us know where to pick them up :-)