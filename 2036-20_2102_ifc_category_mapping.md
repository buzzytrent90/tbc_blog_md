---
author: Jeremy Tammik
optimization_date: '2025-12-11T11:24:41.343015'
original_url: https://thebuildingcoder.typepad.com/blog/2036_whats_new_2025.html
parent_post: 2036_whats_new_2025.md
part_number: '20'
part_total: '30'
post_number: '2036'
series: geometry
slug: whats_new_2025_2.10.2._ifc_category_mapping
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
title: 1. API Changes - 2.10.2. IFC Category Mapping
word_count: 372
---

### 2.10.2. IFC Category Mapping

Added functionality that allows user to control the way how Revit categories are mapped to IFC Classes during IFC Export. User can now create, retrieve and modify IFC category mapping templates.
The new classes:
- Autodesk.Revit.DB.IFCCategoryTemplate – Represents an element that contains IFC category mapping template stored in a Revit document.
- Autodesk.Revit.DB.ExportIFCCategoryKey – Represents a Revit category item stored in a template.
- Autodesk.Revit.DB.ExportIFCCategoryInfo – Represents the mapped IFC information stored in the template.
The new enum:
- CustomSubCategoryId – Represents pseudo sub-categories that can appear in a mapping template
has the following values:
- None – Represents the default value for most Revit categories and subcategories.
- InteriorWall – Represents the custom id for interior walls.
- ExteriorWall – Represents the custom id for exterior walls.
- FoundationWall – Represents the custom id for foundation walls.
- RetainingWall – Represents the custom id for retaining walls.
- Coreshaft – Represents the custom id for cores/shafts.
- Soffit – Represents the custom id for soffits.

## 2.11. Import Export API additions

The new functionality provides support for importing, linking and exporting files of STEP format.
The new class:
- STEPImportOptions
represents the options for STEP formats.
The new methods, uses the new class STEPImportOptions:
- Document.Import(String, STEPImportOptions, View)
- Document.Link(String, STEPImportOptions, View)
- Document.Export(String, String, STEPExportOptions)

## 2.12. PDF Export API additions

Document export for PDF now allows using a separate Revit Worker to create the PDF in the background, leaving the main Revit process free for other work. FileExporting and FileExported events are triggered at the start and end of the export job respectively. There are new API calls for PDFExportOptions, FileExportedEventArgs, and FileExportingEventArgs. Third party Addins may interfere with the PDF generation using this feature if they change the appearance of Elements in a way that is not serialized.
The new method:
- PDFExportOptions.SetExportInBackground() – When set to true, Document.Export launches a new process to export the PDF that leaves Revit unblocked. Changes to the document made after the export has started are not accounted for in the PDF.
The classes FileExportingEventArgs and FileExportedEventArgsBoth have a new member:
- BackgroundOperation – The value is true if a background process was requested for the background operation that raised the event.

## 2.13. Reinforcement API additions
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
- [1. API Changes - 2.13.1. Rebar](./2036-21_2131_rebar.md)
- [1. API Changes - 2.13.2. Rebar splice type options and rules](./2036-22_2132_rebar_splice_type_options_and_rules.md)
- [1. API Changes - 2.14.1. UI Application](./2036-23_2141_ui_application.md)
- [1. API Changes - 2.15.1. Wall APIs](./2036-24_2151_wall_apis.md)
- [1. API Changes - 2.16.1. Analytical Elements](./2036-25_2161_analytical_elements.md)
- [1. API Changes - 2.16.2. Analytical Surface](./2036-26_2162_analytical_surface.md)
- [1. API Changes - 2.16.3. Bending Details on Drawings](./2036-27_2163_bending_details_on_drawings.md)
- [1. API Changes - 2.16.4. Radial Array](./2036-28_2164_radial_array.md)
- [1. API Changes - 2.19.1. Context Menu](./2036-29_2191_context_menu.md)
- [1. API Changes - 2.20.1. SheetCollection](./2036-30_2201_sheetcollection.md)
