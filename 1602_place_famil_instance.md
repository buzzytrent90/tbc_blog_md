---
ai_optimized: true
author: Jeremy Tammik
complexity_score: 6.5
content_type: qa
optimization_date: '2025-12-11T11:44:16.373018'
original_url: https://thebuildingcoder.typepad.com/blog/1602_place_famil_instance.html
post_number: '1602'
reading_time_minutes: 4
series: elements
slug: place_famil_instance
source_file: 1602_place_famil_instance.md
tags:
- elements
- family
- references
- revit-api
- sheets
title: Place Famil Instance
word_count: 842
---

### Migrating PlaceInstances to Revit 2018
Migrating a Revit add-in to a new release of the Revit API is generally very easy.
The API features slight changes from version to version.
Modifications are announced a year or two in advance, and signalled during compilation by deprecated API usage warnings.
If you clean up your code every year or two and remove all API usage that causes warning messages, you will normally have very little to do to migrate it later on.
If you run into any problems, just search the documentation on \*What's New in the Revit API\*; most modifications are listed there, including instructions on how to update the code to handle them:
- [What's New in the Revit 2010 API](http://thebuildingcoder.typepad.com/blog/2013/02/whats-new-in-the-revit-2010-api.html)
- [What's New in the Revit 2011 API](http://thebuildingcoder.typepad.com/blog/2013/02/whats-new-in-the-revit-2011-api.html)
- [What's New in the Revit 2012 API](http://thebuildingcoder.typepad.com/blog/2013/02/whats-new-in-the-revit-2012-api.html)
- [What's New in the Revit 2013 API](http://thebuildingcoder.typepad.com/blog/2013/03/whats-new-in-the-revit-2013-api.html)
- [What's New in the Revit 2014 API](http://thebuildingcoder.typepad.com/blog/2013/04/whats-new-in-the-revit-2014-api.html)
- [What's New in the Revit 2015 API](http://thebuildingcoder.typepad.com/blog/2014/04/whats-new-in-the-revit-2015-api.html)
- [What's New in the Revit 2016 API](http://thebuildingcoder.typepad.com/blog/2015/04/whats-new-in-the-revit-2016-api.html)
- [What's New in the Revit 2017 API](http://thebuildingcoder.typepad.com/blog/2016/04/whats-new-in-the-revit-2017-api.html)
- [What's New in the Revit 2017.1 API](http://thebuildingcoder.typepad.com/blog/2016/11/whats-new-in-the-revit-20171-api.html)
- [What's New in the Revit 2018 API](http://thebuildingcoder.typepad.com/blog/2017/04/whats-new-in-the-revit-2018-api.html)
- Revit 2018.1 API:
- [Revit 2018.1 and the Visual Materials API](http://thebuildingcoder.typepad.com/blog/2017/08/revit-20181-and-the-visual-materials-api.html)
- [Revit 2018.1.1 API documentation](http://thebuildingcoder.typepad.com/blog/2017/09/revit-201811-fixes-cropbox-setting.html)
As an example, let's look at the migration of
the [PlaceInstances add-in implementing text file driven automatic placement of family instances](http://thebuildingcoder.typepad.com/blog/2013/10/text-file-driven-automatic-placement-of-family-instances.html) from
Revit 2014 to Revit 2018, prompted
by [two comments](http://thebuildingcoder.typepad.com/blog/2013/10/text-file-driven-automatic-placement-of-family-instances.html#comment-3619372844) by
Campbell and Renzo:
\*\*Question:\*\*
> I'm trying to get this to work in Revit 2016; however, `FamilySymbolSet` was removed in 2016.
Is there a workaround to get the plug in on this page to work without it?
I can get the form to populate with families, but not show the types when I select one...
> Could you find the solution?
\*\*Answer:\*\* Yes, this is easy.
The class `FamilySymbolSet` no longer exists; use a generic collection of element ids instead, in this case, `ISet`.
The property `Family.Symbols` no longer exists; use `GetFamilySymbolIds` instead.
I first performed a flat migration from the Revit 2014 API to Revit 2018, sinply updating the license year, .NET build target version and Revit API DLL references.
That causes the following errors and warnings:

```
------ Rebuild All started: Project: PlaceInstances, Configuration: Debug Any CPU ------

error CS0246: The type or namespace name 'FamilySymbolSet' could not be found (are you missing a using directive or an assembly reference?)

error CS1061: 'Family' does not contain a definition for 'Symbols' and no extension method 'Symbols' accepting a first argument of type 'Family' could be found (are you missing a using directive or an assembly reference?)

warning CS0162: Unreachable code detected
========== Rebuild All: 0 succeeded, 1 failed, 0 skipped ==========
```

The warning message is intentional.
The Revit 2014 code causing the two errors looks like this:
```csharp
FamilySymbolSet symbols = f.Symbols;
// I have to convert the FamilySymbolSet to a
// List, or the DataSource assignment will throw
// an exception saying "Complex DataBinding
// accepts as a data source either an IList or
// an IListSource.
List symbols2
= new List(
symbols.Cast() );
```
I converted it to compile for Revit 2018 like this:
```csharp
ISet ids = f.GetFamilySymbolIds();
Document doc = f.Document;
List symbols2
= new List(
ids.Select( id
=> doc.GetElement( id ) as FamilySymbol ) );
```
The update is provided in
the [PlaceInstances GitHub repository](https://github.com/jeremytammik/PlaceInstances)
in [release 2018.0.0.0](https://github.com/jeremytammik/PlaceInstances/releases/tag/2018.0.0.0).
You can check out the changes I made in
the [diff to the preceding version](https://github.com/jeremytammik/PlaceInstances/compare/2014.0.0.7...2018.0.0.0).
For the sake of completeness, here are
the [error lists before and after the fix](zip/2018_placeinstances_01.txt).
I hope this helps.
![Migration](img/migration.png)