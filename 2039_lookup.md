---
post_number: "2039"
title: "Lookup"
slug: "lookup"
author: "Jeremy Tammik"
tags: ['elements', 'family', 'filtering', 'geometry', 'parameters', 'references', 'revit-api', 'sheets', 'views']
source_file: "2039_lookup.md"
original_url: "https://thebuildingcoder.typepad.com/blog/2039_lookup.html"
---

### RevitLookup Updates, Bounding Boxes and Podcast
A quick heads-up on a podcast interview, new releases of RevitLookup vastly expanding coverage to include numerous new classes and properties, and other notes of interest:
- [BIMrras podcast interview](#2)
- [RevitLookup 2025.0.3](#3)
- [RevitLookup 2025.0.4](#4)
- [`Outline` versus `BoundingBox`](#5)
- [Linking Revit files in BIM360 Docs](#6)
#### BIMrras Podcast Interview
Evelio Sánchez y Rogelio Carballo invited me to participate in
their [BIMrras Podcast](https://www.bimrras.com/):
> El Primer Podcast BIM Colaborativo

¡El podcast sobre BIM que Chuck Norris no se atreve a escuchar!
I joined them last week for a very pleasant chat in
episode [157 Building with code, with Jeremy Tammik](https://www.bimrras.com/episodio/157-building-with-code-with-jeremy-tammik/).
#### RevitLookup 2025.0.3
Roman [@Nice3point](https://t.me/nice3point) Karpovich, aka Роман Карпович,
published [RevitLookup release 2025.0.3](https://github.com/jeremytammik/RevitLookup/releases/tag/2025.0.3),
integrating extensive work
by [Sergey Nefyodov](https://github.com/SergeyNefyodov) to expand coverage to numerous new classes, properties and contexts.
Sergey packaged his enhancements in pull requests
[227 (`ConnectorManager`)](https://github.com/jeremytammik/RevitLookup/pull/227),
[228 (`Wire`)](https://github.com/jeremytammik/RevitLookup/pull/228),
[229 (`IndependentTag`)](https://github.com/jeremytammik/RevitLookup/pull/229),
[230 (`CurveElement`)](https://github.com/jeremytammik/RevitLookup/pull/230),
[231 (`TableView`)](https://github.com/jeremytammik/RevitLookup/pull/231),
[232 (`DatumPlane`)](https://github.com/jeremytammik/RevitLookup/pull/232)
and [233 (extensions)](https://github.com/jeremytammik/RevitLookup/pull/233).
Specific improvement include:
Memory diagnoser:
![Memory diagnoser](img/revitlookup_memory.png "Memory diagnoser")
- The `Memory` column shows the size of allocated \*\*managed memory\*\*
- Native ETW and allocations in C++ code are not included to avoid severe performance degradation
Different method overload variations now displayed in a `Variants` collection:
- Previously: `GeometryElement`
- Now: `Variants`
![Overload variations](img/revitlookup_variants.png "Overload variations")
More:
- ConnectorManager class support
– Added `ConnectorManager.Lookup`
- Wire class support
– Added `Wire.GetVertex`
- IndependentTag class support
– Added `IndependentTag.CanLeaderEndConditionBeAssigned`, `IndependentTag.GetLeaderElbow`, `IndependentTag.GetLeaderEnd`, `IndependentTag.HasLeaderElbow`, `IndependentTag.IsLeaderVisible`
- CurveElement class support
– Added `CurveElement.GetAdjoinedCurveElements`, `CurveElement.HasTangentLocks`, `CurveElement.GetTangentLock`, `CurveElement.HasTangentJoin`, `CurveElement.IsAdjoinedCurveElement`
- TableView class support
– Added `TableView.GetAvailableParameters`, `TableView.GetCalculatedValueName`, `TableView.GetCalculatedValueText`, `TableView.IsValidSectionType`, `TableView.GetCellText`
- DatumPlane class support
– Added `DatumPlane.CanBeVisibleInView`, `DatumPlane.GetPropagationViews`, `DatumPlane.CanBeVisibleInView`, `DatumPlane.GetPropagationViews`, `DatumPlane.GetDatumExtentTypeInView`, `DatumPlane.HasBubbleInView`, `DatumPlane.IsBubbleVisibleInView`, `DatumPlane.GetCurvesInView`, `DatumPlane.GetLeader`
- Extensions
– Added Family class extension `FamilySizeTableManager.GetFamilySizeTableManager`, FamilyInstance class extension `AdaptiveComponentInstanceUtils.GetInstancePlacementPointElementRefIds`, FamilyInstance class extension `AdaptiveComponentInstanceUtils.IsAdaptiveComponentInstance`, Solid class extension `SolidUtils.SplitVolumes`, Solid class extension `SolidUtils.IsValidForTessellation`
- [Full changelog 2025.0.2...2025.0.3](https://github.com/jeremytammik/RevitLookup/compare/2025.0.2...2025.0.3)
- [RevitLookup versioning](https://github.com/jeremytammik/RevitLookup/wiki/Versions)
#### RevitLookup 2025.0.4
As if that were not enough, Roman and Sergey immediately followed up
with [RevitLookup release 2025.0.4](https://github.com/jeremytammik/RevitLookup/releases/tag/2025.0.4),
integrating the further pull requests
[235](https://github.com/jeremytammik/RevitLookup/pull/235)
and [236](https://github.com/jeremytammik/RevitLookup/pull/236),
focused on improving core functionalities and robustness of the product.
- Introducing a preview feature for \*\*Family Size Table\*\*, making it easier to manage and visualize family sizes
![Family size table](img/revitlookup_family_size_table.png "Family size table")
To access it:
- Enable \*\*Show Extensions\*\* in the view menu
- Select any \*\*FamilyInstance\*\*
- Navigate to the \*\*Symbol\*\*
- Navigate to the \*\*Family\*\* (or just search for Family class objects in the \*\*Snoop database\*\* command)
- Navigate to the \*\*GetFamilySizeTableManager\*\* method
- Navigate to the \*\*GetSizeTable\*\* method
- Right-click on one of the tables and select the \*\*Show table\*\* command
- Note: Family size table is currently in read-only mode
More:
- Added new context menu item for selecting elements without showing
- Added new fresh, intuitive icons to the context menu for a more user-friendly interface.
- Refined labels, class names, and exception messages
- Resolved an issue where the delete action was not displayed in the context menu for ElementType classes
- Fixed the context menu display issue for Element classes, broken in previous release
- Fixed the order of descriptors to prevent missing extensions and context menu items in some classes, broken in previous release
- [Full changelog](https://github.com/jeremytammik/RevitLookup/compare/2025.0.3...2025.0.4)
- [RevitLookup versioning](https://github.com/jeremytammik/RevitLookup/wiki/Versions)
Ever so many thanks to Roman and Sergey for their impressive and untiring implementation and maintenance work!
#### Outline Versus BoundingBox
Interesting aspects of different kinds of bounding boxes and their uses in intersection filters are discussed in the thread
on [`Outline` vs `BoundingBoxXYZ` in Revit API](https://forums.autodesk.com/t5/revit-api-forum/outline-vs-boundingboxxyz-in-revit-api/m-p/12670522).
#### Linking Revit Files in BIM360 Docs
Several users asked whether it is possible to link Revit projects directly in ACC and BIM360 Docs.
Luckily, Eason Kang has covered that topic extensively in his article
on [BIM360 Docs: Setting up external references between files (Upload Linked Files)](https://aps.autodesk.com/blog/bim360-docs-setting-external-references-between-files-upload-linked-files).