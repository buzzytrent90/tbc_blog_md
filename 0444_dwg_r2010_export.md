---
ai_optimized: true
author: Jeremy Tammik
complexity_score: 1.9
content_type: qa
optimization_date: '2025-12-11T11:44:13.955340'
original_url: https://thebuildingcoder.typepad.com/blog/0444_dwg_r2010_export.html
post_number: '0444'
reading_time_minutes: 2
series: general
slug: dwg_r2010_export
source_file: 0444_dwg_r2010_export.htm
tags:
- csharp
- revit-api
- views
title: AutoCAD 2010 DWG Export
word_count: 347
---

### AutoCAD 2010 DWG Export

Today is a public holiday in Neuchtel, the
[jene fdral](http://en.wikipedia.org/wiki/Je%C3%BBne_genevois) or
[Federal Fast holiday](http://en.wikipedia.org/wiki/Store_Bededag).
I will use that for a hike in the Swiss mountains, that I so sorely neglected and missed this summer.

Meanwhile, here is a simple question on the AutoCAD file format version used for DWG export.

**Question:** How can I set the AutoCAD 2010 version as the destination file format for DWG export?

The ACADVersion enumeration only has options for 2000, 2004, and 2007.

In the user interface, it is possible to select 2010 as well.

**Answer:** Looking at the ACADVersion enumeration, I see the following entries:

- Default: The Autodesk Revit application's default export format.- R2000: AutoCAD 2000 file format.- R2004: AutoCAD 2004 file format.- R2007: AutoCAD 2007 file format.

The AutoCAD 2010 file format was recently added as another format in the DWG export options.
It is currently created by specifying the 'Default' value.

Because this is a simple enum wrapper, you can also cause Revit to export the AutoCAD 2010 file format by passing in '4' to the options settings.
Using Reflector, you can see that the numerical values of the existing entries are 0-3, which makes it very probable that 4 is the next one.
In any case it worked, for example, using the following code:
```csharp
  DWGExportOptions options = new DWGExportOptions();

  options.FileVersion = ( ACADVersion ) ( 4 );

  ViewSet views = new ViewSet();

  views.Insert( uiDoc.Document.ActiveView );

  uiDoc.Document.Export( @"d:\temp", @"junk.dwg",
    views, options );
```

This creates a resulting DWG file in AutoCAD 2010 file format with a header of ac1024, equal to the ObjectARX enum value kDHL\_1024 = 29, which stands for '2010 final'.

As said, ACADVersion.Default also results in an AutoCAD 2010 file format DWG, but that will obviously change in future releases.

Casting an integer to an enum value to pass into the settings is obviously not an officially supported way of using the Revit API, but it might help you get around this in the short term.