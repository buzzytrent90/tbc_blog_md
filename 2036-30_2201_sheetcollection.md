---
author: Jeremy Tammik
optimization_date: '2025-12-11T11:24:41.352898'
original_url: https://thebuildingcoder.typepad.com/blog/2036_whats_new_2025.html
parent_post: 2036_whats_new_2025.md
part_number: '30'
part_total: '30'
post_number: '2036'
series: geometry
slug: whats_new_2025_2.20.1._sheetcollection
source_file: 2036_whats_new_2025.md
tags:
- csharp
- elements
- family
- filtering
- geometry
- levels
- parameters
- python
- references
- revit-api
- schedules
- selection
- sheets
- views
- walls
title: 1. API Changes - 2.20.1. SheetCollection
word_count: 366
---

### 2.20.1. SheetCollection

The new class:
- Autodesk.Revit.DB.SheetCollection – Represents a sheet collection in Autodesk Revit.
The new methods:
- Autodesk.Revit.DB.SheetCollection.Create(document, name) – Allows users to create a new instance of sheet collection with a specified name and adds it to the document. It returns the newly created sheet collection element.
- Autodesk.Revit.DB.SheetCollection.Create(document) – Allows users to create a new instance of sheet collection with an auto-generated name and adds it to the document. It returns the newly created sheet collection element.
The new property:
- Autodesk.Revit.DB.ViewSheet.SheetCollectionId – Represents the Id of the sheet collection this sheet is associated with.

## 2.21. Link Visibility/Graphic Override API additions

The new functionality allows Revit Link Visibility/Graphic Overrides, for the 'Custom' option.
The new methods in class Autodesk.Revit.DB.RevitLinkGraphicsSettings:
- RevitLinkGraphicsSettings.IsViewRangeSupported(View) – Allows users to check if the input view supports view range settings for RevitLinkGraphicsSettings graphic overrides.
- RevitLinkGraphicsSettings.GetPhaseId()
- RevitLinkGraphicsSettings.GetPhaseType()
- RevitLinkGraphicsSettings.SetPhase(LinkVisibility, ElementId) – Allows users to configure phase and phase type of RevitLinkGraphicsSettings. Accepts LinkVisibility and ElementId of the phase from the linked document or ElementId.InvalidElementId.
- RevitLinkGraphicsSettings.GetPhaseFilterId()
- RevitLinkGraphicsSettings.GetPhaseFilterType()
- RevitLinkGraphicsSettings.SetPhaseFilter(LinkVisibility, ElementId) – Allows users to configure phase filter and phase filter type of RevitLinkGraphicsSettings. Accepts LinkVisibility and ElementId of the phase filter from the linked document or ElementId.InvalidElementId.
- RevitLinkGraphicsSettings.GetViewDetailLevel()
- RevitLinkGraphicsSettings.GetViewDetailLevelType()
- RevitLinkGraphicsSettings.SetViewDetailLevel(LinkVisibility, ViewDetailLevel) – Allows users to configure detail level and detail level type of RevitLinkGraphicsSettings. Accepts LinkVisibility and ViewDetailLevel types.
- RevitLinkGraphicsSettings.GetDiscipline()
- RevitLinkGraphicsSettings.GetDisciplineType()
- RevitLinkGraphicsSettings.SetDiscipline(LinkVisibility, ViewDiscipline) – Allows users to configure discipline and discipline type of RevitLinkGraphicsSettings. Accepts LinkVisibility and ViewDiscipline types.
The new properties in Autodesk.Revit.DB.RevitLinkGraphicsSettings:
- RevitLinkGraphicsSettings.ViewFilterType
- RevitLinkGraphicsSettings.ViewRange
- RevitLinkGraphicsSettings.ColorFill
- RevitLinkGraphicsSettings.ObjectStyles
- RevitLinkGraphicsSettings.NestedLinks

## 2.22. RevitServer Enterprise / Revit Cloud Worksharing API additions

The following events have been supported for file-based worksharing since 2021. We now support them in RevitServer Enterprise/Revit Cloud Worksharing.
- DocumentReloadingLatest – Subscribe to the DocumentReloadingLatestEventArgs event to be notified when Revit is just about to reload latest changes from a central model.
- DocumentReloadedLatest – Subscribe to the DocumentReloadedLatestEventArgs event to be notified immediately after Revit has finished reloading a document with central model.
---

## Related Sections
- [1. API Changes - What's New in the Revit 2025 API](./2036-01_whats_new_in_the_revit_2025_api.md)
- [1. API Changes - 1.3.1. MacroManager API](./2036-02_131_macromanager_api.md)
- [1. API Changes - 1.8.1. Distribution system](./2036-03_181_distribution_system.md)
- [1. API Changes - 1.8.2. Parameter Naming](./2036-04_182_parameter_naming.md)
- [1. API Changes - 1.10.1. Duct Settings](./2036-05_1101_duct_settings.md)
- [1. API Changes - 1.11.1. Rebar](./2036-06_1111_rebar.md)
- [1. API Changes - 1.13.1. Bending details in view](./2036-07_1131_bending_details_in_view.md)
- [1. API Changes - 1.13.2. Analytical Surface](./2036-08_1132_analytical_surface.md)
- [1. API Changes - 1.16.1. Classes](./2036-09_1161_classes.md)
- [1. API Changes - 1.16.2. Properties](./2036-10_1162_properties.md)
- [1. API Changes - 1.16.3. Methods](./2036-11_1163_methods.md)
- [1. API Changes - 1.16.4. Enums](./2036-12_1164_enums.md)
- [1. API Changes - 2.1.1. MacroManager API](./2036-13_211_macromanager_api.md)
- [1. API Changes - 2.2.1. MEP Analytical networks](./2036-14_221_mep_analytical_networks.md)
- [1. API Changes - 2.2.2. MEP Duct/Pipe Pressure Loss calculation](./2036-15_222_mep_ductpipe_pressure_loss_calculation.md)
- [1. API Changes - 2.2.3. MEP Space Engineering Parameters](./2036-16_223_mep_space_engineering_parameters.md)
- [1. API Changes - 2.4.1. Linear Array](./2036-17_241_linear_array.md)
- [1. API Changes - 2.9.1. gbXML export options](./2036-18_291_gbxml_export_options.md)
- [1. API Changes - 2.10.1. IFC Hybrid Import](./2036-19_2101_ifc_hybrid_import.md)
- [1. API Changes - 2.10.2. IFC Category Mapping](./2036-20_2102_ifc_category_mapping.md)
- [1. API Changes - 2.13.1. Rebar](./2036-21_2131_rebar.md)
- [1. API Changes - 2.13.2. Rebar splice type options and rules](./2036-22_2132_rebar_splice_type_options_and_rules.md)
- [1. API Changes - 2.14.1. UI Application](./2036-23_2141_ui_application.md)
- [1. API Changes - 2.15.1. Wall APIs](./2036-24_2151_wall_apis.md)
- [1. API Changes - 2.16.1. Analytical Elements](./2036-25_2161_analytical_elements.md)
- [1. API Changes - 2.16.2. Analytical Surface](./2036-26_2162_analytical_surface.md)
- [1. API Changes - 2.16.3. Bending Details on Drawings](./2036-27_2163_bending_details_on_drawings.md)
- [1. API Changes - 2.16.4. Radial Array](./2036-28_2164_radial_array.md)
- [1. API Changes - 2.19.1. Context Menu](./2036-29_2191_context_menu.md)
