---
ai_optimized: true
author: Jeremy Tammik
complexity_score: 1.8
content_type: code_example
optimization_date: '2025-12-11T11:44:14.515275'
original_url: https://thebuildingcoder.typepad.com/blog/0749_addin_wizard_2013.html
post_number: 0749
reading_time_minutes: 2
series: general
slug: addin_wizard_2013
source_file: 0749_addin_wizard_2013.htm
tags:
- csharp
- elements
- filtering
- python
- revit-api
- selection
- transactions
- walls
title: Add-In Wizard for Revit 2013
word_count: 326
---

### Add-In Wizard for Revit 2013

I have been using an updated version of my Visual Studio Revit C# add-in wizard for Revit 2013 for a while now and thought you might find it useful as well.

It now generates a bit more boiler-plate code up front which can be simply deleted if not needed:
```python
  UIApplication uiapp = commandData.Application;
  UIDocument uidoc = uiapp.ActiveUIDocument;
  Application app = uiapp.Application;
  Document doc = uidoc.Document;

  // Access current selection

  Selection sel = uidoc.Selection;

  // Retrieve elements from database

  FilteredElementCollector col
    = new FilteredElementCollector( doc )
      .WhereElementIsNotElementType()
      .OfCategory( BuiltInCategory.INVALID )
      .OfClass( typeof( Wall ) );

  // Filtered element collector is iterable

  foreach( Element e in col )
  {
    Debug.Print( e.Name );
  }

  // Modify document within a transaction

  Transaction tx = new Transaction( doc );

  tx.Start();
  tx.Commit();

  return Result.Succeeded;
```

For the full description of the wizards, please refer to these previous posts:

- [Original introduction, benefits, and usage example](http://thebuildingcoder.typepad.com/blog/2010/07/devtv-addin-templates.html) for C# and VB.- Personalised
    [minimal C# version](http://thebuildingcoder.typepad.com/blog/2010/10/revit-2011-devtv.html#2) for Revit 2011.- A short additional
      [usage note](http://thebuildingcoder.typepad.com/blog/2010/12/snow-and-woe-with-manifest-files.html).- [64 bit versions](http://thebuildingcoder.typepad.com/blog/2011/01/automate-designoption-and-64-bit-add-in-templates.html#2) for C# and VB.- Support for the
          [Revit 2012 API](http://thebuildingcoder.typepad.com/blog/2011/04/visual-studio-add-in-wizards-for-revit-2012.html) for
          C# and VB.- Updated C# and VB versions placing
            [assembly DLL alongside add-in manifest](http://thebuildingcoder.typepad.com/blog/2011/10/product-and-add-in-wizard-updates.html#3) and
            including other changes.

To install, simply copy the zip file to the Visual Studio C# project template folder in your local file system:

- [RevitAddinWizardQuasarRP.zip](zip/RevitAddinWizardQuasarRP.zip) – copy to

  [My Documents]\Visual Studio 2010\Templates\ProjectTemplates\Visual C#

Any volunteers to create and test the VB version?