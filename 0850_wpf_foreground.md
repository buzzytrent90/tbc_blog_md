---
post_number: "0850"
title: "Ensure WPF Add-in Remains in Foreground"
slug: "wpf_foreground"
author: "Jeremy Tammik"
tags: ['csharp', 'elements', 'family', 'filtering', 'levels', 'revit-api', 'views', 'walls', 'windows']
source_file: "0850_wpf_foreground.htm"
original_url: "https://thebuildingcoder.typepad.com/blog/0850_wpf_foreground.html"
---

﻿

### Ensure WPF Add-in Remains in Foreground

Autumn arrived and passed in a flash.
The woods are beautifully coloured, and, unusually, the first snow is already lying on the surrounding hills.

Here are some items of interest from cases we looked at last week:

1. [Ensure WPF add-in remains in foreground](#1)- [Retrieve all model lines](#2)- [Set linked file visibility](#3)- [Access linked file elements and data](#4)- [Visual Studio 2012 Model Editor Supports OBJ and FBX](#5)- [Sie Mögen Sich](#6)

#### Ensure WPF Add-in Remains in Foreground

Here is an issue concerning an interaction between a WPF add-in and the Revit TaskDialog, raised and solved by Simon Hooper of
[Bestech Systems Ltd](http://www.bestech.co.uk):

**Question:** I have a modal WPF Revit add-in application.

If I show a TaskDialog, e.g. call `TaskDialog.Show("Global", "Hello World")`, my WPF add-in is sent to 'the back' behind Revit, in spite of being modal!

If I use `MessageBox.Show("")`, the problem does not occur.

How can this be resolved, please?

**Answer:** I ensured that the WPF form properly parented, i.e. is it a child form of the Revit main application window.

Googling for "JtWindowHandle wrapper class" provides lots of examples of how to parent a normal .NET modeless form.

The procedure is applicable to WPF forms as well.

It seems like the problem is solved by using the following lines:
```csharp
  exportForm = new ExportForm();

  System.Windows.Interop.IWin32Window revit\_window
    = new JtWindowHandle( Autodesk.Windows
      .ComponentManager.ApplicationWindow );

  System.Windows.Interop.WindowInteropHelper helper
    = new System.Windows.Interop.WindowInteropHelper(
      exportForm );

  helper.Owner = revit\_window.Handle;
```

The helper line was the key.

#### Retrieve all Model Lines

**Question:** I have a very simple question.

I want to scan my Revit model for all Model Line instances.

RevitLookup tells me I should use the built-in category OST\_Lines in my filter, but it does not find the lines.

Here is my logic:
```csharp
  FilteredElementCollector collector
    = new FilteredElementCollector( doc );

  ElementClassFilter familyInstanceFilter
    = new ElementClassFilter(
      typeof( FamilyInstance ) );

  ElementCategoryFilter oCategoryFilter
    = new ElementCategoryFilter(
      BuiltInCategory.OST\_Lines );

  LogicalAndFilter andFilter
    = new LogicalAndFilter(
      familyInstanceFilter,
      oCategoryFilter );

  ICollection<Element> oElements
    = collector.WherePasses( andFilter )
      .ToElements();
```

However, this returns nothing at all!

What is wrong?

**Answer:**
Yes, this is indeed a rather simple question.

Model lines are model curves.
Model curves are not family instances.

Your collector includes a family instance filter.
The family instance filter will eliminate all model curves, preventing any model lines from being found.

Simply remove the family instance class filter, or replace it by a model curve class filter.

By the way, the most common quick filters are all available using shortcut methods on the collector class itself.

Using the shortcut methods saves you from having to instantiate a separate filter instance, and ensures that all the filters you apply are quick filers.

All the following three variants should work, and presumably produce identical results:
```csharp
  FilteredElementCollector collector
    = new FilteredElementCollector( doc )
      .OfClass( typeof( ModelCurve ) )
      .OfCategory( BuiltInCategory.OST\_Lines );

  FilteredElementCollector collector
    = new FilteredElementCollector( doc )
      .OfCategory( BuiltInCategory.OST\_Lines );

  FilteredElementCollector collector
    = new FilteredElementCollector( doc )
      .OfClass( typeof( ModelCurve ) );
```

#### Set Linked File Visibility

Saikat Bhattacharya demonstrates the use of the View class HideElements and UnhideElements methods to
[control the visibility of linked files](http://adndevblog.typepad.com/aec/2012/10/setting-visibility-of-linked-files-using-revit-api.html),
and points out that the SetVisibility method cannot be used for this purpose.

#### Access Linked File Elements and Data

On a still more exciting and ever recurring linked file topic, Saikat shows how easy it is to access the elements and data contained in a linked file, e.g. to
[read the level information of all walls in a linked file](http://adndevblog.typepad.com/aec/2012/10/accessing-data-from-linked-file-using-revit-api.html).

#### Visual Studio 2012 Model Editor Supports OBJ and FBX

Visual Studio 2012 includes a
[Model Editor](http://msdn.microsoft.com/en-us/library/hh315734.aspx) that
enables you to inspect the 3-D model formats OBJ, Autodesk FBX and COLLADA.
You can also use built-in 3-D primitive generation and materials to create placeholder art for 3-D games and applications.

That sounds handy, doesn't it?

I recently had a look at the OBJ format to implement
[Revit model export to OBJ](http://thebuildingcoder.typepad.com/blog/2012/07/obj-model-exporter-with-transparency-support.html),
and obviously Revit already provides built-in support for
[FBX export](http://wikihelp.autodesk.com/Revit/enu/2013/Help/00001-Revit_He0/1468-Document1468/2171-Print_Ex2171/2172-Export2172/2226-Exportin2226/2232-Exportin2232)
([LT](http://wikihelp.autodesk.com/Revit_LT/enu/2013/Help/0075-Revit_LT75/0888-Document888/1536-Print_Ex1536/1537-Export1537/1575-Exportin1575/1581-Exportin1581),
[Vasari](http://wikihelp.autodesk.com/Vasari/enu/B1/Help/0296-Visualiz296/0499-Print_Ex499/0500-Export500/0544-Exportin544)).

If you wanted to work in depth with FBX yourself, you would presumably use the
[FBX SDK](http://usa.autodesk.com/adsk/servlet/pc/item?siteID=123112&id=10775847),
currently at version 2013.3, supporting C++ development in Visual Studio 2005, 2008, and 2010.

#### Sie Mögen Sich

For something completely unrelated to computers and technology, here is a seven minute video, [Sie mögen sich](http://www.youtube.com/watch?v=apCal7ihvy0), a quite deep philosophical relationship analysis by Shaban & Käptn Peng that my son Christopher pointed out and that I enjoyed a lot:

It is in German, however, and probably makes no sense at all unless you understand the language.