---
post_number: "1350"
title: "Revit Add-In Wizard GitHub Installer"
slug: "wizard_github_install"
author: "Jeremy Tammik"
tags: ['csharp', 'revit-api', 'vbnet', 'views', 'windows']
source_file: "1350_wizard_github_install.htm"
original_url: "https://thebuildingcoder.typepad.com/blog/1350_wizard_github_install.html"
---

### Revit Add-In Wizard GitHub Installer

I recently mentioned the
[unrestricted VendorId](http://thebuildingcoder.typepad.com/blog/2015/08/batch-processing-dwfx-links-and-future-proofing.html#2) in
Revit 2016, and pointed out that I would like to update the
[Visual Studio Revit add-in wizards](http://thebuildingcoder.typepad.com/blog/about-the-author.html#5.20) accordingly.

I now went ahead and did so, adding a couple of other enhancements as well along the way – oh, and I did some work on putting my personal calendar online, as well:

- [Sharing a calendar via GCal, Google API, and GitHub Pages](#1).
- [Visual Studio Revit add-in wizards on GitHub](#2).
- [Updated VendorId tag value](#3).
- [Wizard installer batch file](#4).

#### Sharing a Calendar via GCal, Google API, and GitHub Pages

I fiddled around a bit in the past two days to share my calendar online via the cloud, for personal convenience and to provide access to friends and colleagues.

Here is an overview of the steps I took:

- [My Text Based Calendar](http://the3dwebcoder.typepad.com/blog/2015/08/populate-google-calendar-using-google-calendar-api.html#2)
- [The pcal PostScript calendar program](http://the3dwebcoder.typepad.com/blog/2015/08/populate-google-calendar-using-google-calendar-api.html#3)
- [The Google Calendar API](http://the3dwebcoder.typepad.com/blog/2015/08/populate-google-calendar-using-google-calendar-api.html#4)
- [Adding an event to the Google calendar](http://the3dwebcoder.typepad.com/blog/2015/08/populate-google-calendar-using-google-calendar-api.html#5)

- [Google OAuth credential handling](http://the3dwebcoder.typepad.com/blog/2015/08/adding-gcal-entries-using-google-calendar-api.html#2)
- [Date and time parsing and formatting](http://the3dwebcoder.typepad.com/blog/2015/08/adding-gcal-entries-using-google-calendar-api.html#3)
- [Adding a calendar event](http://the3dwebcoder.typepad.com/blog/2015/08/adding-gcal-entries-using-google-calendar-api.html#4)
- [Mainline](http://the3dwebcoder.typepad.com/blog/2015/08/adding-gcal-entries-using-google-calendar-api.html#5)
- [GitHub pages](http://the3dwebcoder.typepad.com/blog/2015/08/adding-gcal-entries-using-google-calendar-api.html#6)
- [GitHub repository](http://the3dwebcoder.typepad.com/blog/2015/08/adding-gcal-entries-using-google-calendar-api.html#7)

Back to the Revit API...

#### Visual Studio Revit Add-in Wizards on GitHub

I created a new
[VisualStudioRevitAddinWizard GitHub repository](https://github.com/jeremytammik/VisualStudioRevitAddinWizard) to
host the wizards and simplify the tracking of changes I make along the way.

It includes
[releases](https://github.com/jeremytammik/VisualStudioRevitAddinWizard/releases) supporting all versions of Revit from Revit 2012 onwards.

#### Updated VendorId Tag Value

I updated the default VendorId tag from `TBC_`, The Building Coder Autodesk registered developer symbol or RDS, to
`com.typepad.thebuildingcoder`, the reversed version of its domain name, as recommended by the Revit developer guide, in the
[Revit 2016 online help](http://help.autodesk.com/view/RVT/2016/ENU) >
Developers > Revit API Developers Guide > Introduction > Add-In Integration >
[Add-in Registration](http://help.autodesk.com/view/RVT/2016/ENU/?guid=GUID-4FFDB03E-6936-417C-9772-8FC258A261F7).

The wizard now generates an add-in manifest file looking like this:

```
<?xml version="1.0" encoding="utf-8"?>
<RevitAddIns>
  <AddIn Type="Command">
    <Text>Command RevitAddin1</Text>
    <Description>Some description for RevitAddin1</Description>
    <Assembly>RevitAddin1.dll</Assembly>
    <FullClassName>RevitAddin1.Command</FullClassName>
    <ClientId>9d12d875-a6f8-44dc-b346-ec29e4153527</ClientId>
    <VendorId>com.typepad.thebuildingcoder</VendorId>
    <VendorDescription>The Building Coder, http://thebuildingcoder.typepad.com</VendorDescription>
  </AddIn>
  <AddIn Type="Application">
    <Name>Application RevitAddin1</Name>
    <Assembly>RevitAddin1.dll</Assembly>
    <FullClassName>RevitAddin1.App</FullClassName>
    <ClientId>c38b3dfc-2b1c-463f-a871-0943bb947f08</ClientId>
    <VendorId>com.typepad.thebuildingcoder</VendorId>
    <VendorDescription>The Building Coder, http://thebuildingcoder.typepad.com</VendorDescription>
  </AddIn>
</RevitAddIns>
```

#### Wizard Installer Batch File

Once I was at it anyway, I thought about how to further simplify installation of the wizard.

The wizard template files need to be zipped up into an archive file which is then placed in the corresponding Visual Studio project template file subdirectory, as described repeatedly, e.g. for the last
[update of the Visual Studio add-in wizards](http://thebuildingcoder.typepad.com/blog/2015/05/autodesk-university-q1-adn-labs-and-wizard-update.html#5).

That can be simplified and automated by setting up a suitable batch file, so I did.

Here is the first version of `install.bat`, also included in the GitHub repo:

```
@echo off
set "D=C:\Users\%USERNAME%\Documents\Visual Studio 2012\Templates\ProjectTemplates"
set "F=%TEMP%\Revit2016AddinWizardCs2.zip"
echo Creating C# wizard archive %F%...
cd cs
zip -r "%F%" *
cd ..
echo Copying C# wizard archive to %D%\Visual C#...
copy "%F%" "%D%\Visual C#"
set "F=%TEMP%\Revit2016AddinWizardVb2.zip"
echo Creating VB wizard archive %F%...
cd vb
zip -r "%F%" *
cd ..
echo Copying VB wizard archive to %D%\Visual Basic...
copy "%F%" "%D%\Visual Basic"
```

With that in place, and the command line
[zip utility for Windows](http://gnuwin32.sourceforge.net/packages/zip.htm) installed,
download and installation consists of the following simple two-step process:

- Clone the GitHub repository:

```
C:> git clone https://github.com/jeremytammik/VisualStudioRevitAddinWizard
```

- Run `install.bat`:

```
C:> install.bat
Z:\a\doc\revit\tbc\src\VisualStudioRevitAddinWizard>install.bat
Creating C# wizard archive C:\Users\tammikj\AppData\Local\Temp\Revit2016AddinWizardCs2.zip...
  adding: App.cs (deflated 54%)
  adding: Command.cs (deflated 59%)
  adding: Properties/ (stored 0%)
  adding: Properties/AssemblyInfo.cs (deflated 56%)
  adding: RegisterAddin.addin (deflated 66%)
  adding: TemplateIcon.ico (deflated 76%)
  adding: TemplateRevitCs.csproj (deflated 70%)
  adding: TemplateRevitCs.vstemplate (deflated 62%)
Copying C# wizard archive to C:\Users\tammikj\Documents\Visual Studio 2012\Templates\ProjectTemplates\Visual C#...
        1 file(s) copied.
Creating VB wizard archive C:\Users\tammikj\AppData\Local\Temp\Revit2016AddinWizardVb2.zip...
  adding: AdskApplication.vb (deflated 68%)
  adding: AdskCommand.vb (deflated 58%)
  adding: My Project/ (stored 0%)
  adding: My Project/AssemblyInfo.vb (deflated 54%)
  adding: RegisterAddin.addin (deflated 67%)
  adding: TemplateIcon.ico (deflated 76%)
  adding: TemplateRevitVb.vbproj (deflated 72%)
  adding: TemplateRevitVb.vstemplate (deflated 62%)
Copying VB wizard archive to C:\Users\tammikj\Documents\Visual Studio 2012\Templates\ProjectTemplates\Visual Basic...
        1 file(s) copied.
```

I hope this works out for you as well.

Please let me know if you hit any snags.

Better still, fork the GitHub repo, fix them, and request a pull. Thank you!

Happy weekend!