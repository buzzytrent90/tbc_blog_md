---
post_number: "0569"
title: "Visual Studio Add-In Wizards for Revit 2012"
slug: "addin_wizard"
author: "Jeremy Tammik"
tags: ['csharp', 'revit-api', 'transactions', 'vbnet']
source_file: "0569_addin_wizard.htm"
original_url: "https://thebuildingcoder.typepad.com/blog/0569_addin_wizard.html"
---

### Visual Studio Add-In Wizards for Revit 2012

As I mentioned yesterday, this weekend was spent traveling to Verona to give a Revit API training here.
Happily, I enjoyed some mountain air the weekend before, on Helgenhorn 2837m,
[Basodino](http://en.wikipedia.org/wiki/Bas%C3%B2dino) 3273m and
Cima di Lago south of All'Acqua in the
[Bedretto Valley](http://en.wikipedia.org/wiki/Bedretto)
([more photos by Jogi](https://picasaweb.google.com/104823077300843743475/BasodinoSTTour)):
![Jeremy on Basodino](file:////j/photo/jeremy/2011/2011-04-03_bedretto_basodino/DSCN1122.jpg)

Anyway, that was last weekend; back to here and now in Verona, where I was busy yesterday, preparing for the training, starting today.

I already updated the DevTV Visual Studio wizards for creating skeleton C# and VB Revit 2011 add-ins several times in the past:

- [Original introduction, benefits, and usage example](http://thebuildingcoder.typepad.com/blog/2010/07/devtv-addin-templates.html) for C# and VB.- Personalised [minimal C# version](http://thebuildingcoder.typepad.com/blog/2010/10/revit-2011-devtv.html#2) for Revit 2011.- A short additional [usage note](http://thebuildingcoder.typepad.com/blog/2010/12/snow-and-woe-with-manifest-files.html).- [64 bit versions](http://thebuildingcoder.typepad.com/blog/2011/01/automate-designoption-and-64-bit-add-in-templates.html#2) for C# and VB.- [Updated C# version](http://thebuildingcoder.typepad.com/blog/2011/02/external-application-attributes.html) without the extraneous transaction attribute on the external application.

One step in my preparations was to migrate the wizard templates to Visual Studio 2010 and the Revit 2012 API.
Here is the first result:

- [TemplateRevitArch2012AddinCsJt\_1.zip](zip/TemplateRevitArch2012AddinCsJt_1.zip) – copy to

  [My Documents]\Visual Studio 2010\Templates\ProjectTemplates\Visual C#- [TemplateRevitArch2012AddinVbJt\_1.zip](zip/TemplateRevitArch2012AddinVbJt_1.zip) – copy to

    [My Documents]\Visual Studio 2010\Templates\ProjectTemplates\Visual Basic

The two most important changes that I made for 2012 are:

- Added required vendor id and optional vendor description to the add-in manifest file.- Removed the obsolete regeneration option attribute.

This is what the auto-generated add-in manifest now looks like:
```csharp
<?xml version="1.0" encoding="utf-8"?>
<RevitAddIns>
  <AddIn Type="Command">
    <Text>Command RevitAddin1</Text>
    <Description>Some description for RevitAddin1</Description>
    <Assembly>c:\tmp\revit\RevitAddin1\RevitAddin1\bin\debug\RevitAddin1.dll</Assembly>
    <FullClassName>RevitAddin1.Command</FullClassName>
    <ClientId>4e980ab7-e6f4-456f-b8cc-646ef733128c</ClientId>
    <VendorId>TBC\_</VendorId>
    <VendorDescription>The Building Coder, http://thebuildingcoder.typepad.com</VendorDescription>
  </AddIn>
  <AddIn Type="Application">
    <Name>Application RevitAddin1</Name>
    <Assembly>c:\tmp\revit\RevitAddin1\RevitAddin1\bin\debug\RevitAddin1.dll</Assembly>
    <FullClassName>RevitAddin1.App</FullClassName>
    <ClientId>dddfad28-c7a0-45fe-836e-0eb03e919e81</ClientId>
    <VendorId>TBC\_</VendorId>
    <VendorDescription>The Building Coder, http://thebuildingcoder.typepad.com</VendorDescription>
  </AddIn>
</RevitAddIns>
```

I used The Building Coder
[Autodesk Registered Developer Symbol RDS](http://www.autodesk.com/symbreg) "TBC\_"
for the vendor id.
You should obviously replace that with your own id before using the template.

Whatever you would like to change, it is extremely easy to adapt the templates for your own personal use:

Simply unpack the zip file somewhere, make your changes, update the zip file with the modified files, and copy the new zip file to the appropriate location.