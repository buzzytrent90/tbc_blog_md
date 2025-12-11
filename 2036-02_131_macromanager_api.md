---
author: Jeremy Tammik
optimization_date: '2025-12-11T11:24:41.322715'
original_url: https://thebuildingcoder.typepad.com/blog/2036_whats_new_2025.html
parent_post: 2036_whats_new_2025.md
part_number: '02'
part_total: '30'
post_number: '2036'
series: geometry
slug: whats_new_2025_1.3.1._macromanager_api
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
title: 1. API Changes - 1.3.1. MacroManager API
word_count: 334
---

### 1.3.1. MacroManager API

Updated Revit macro tool with modernized UI and code editor that also supports modern .NET API. Following methods were changed.
- Deprecated member(s) → Replacement members(s)
- MacroManager.AddModule(ModuleSettings moduleSettings) → MacroManager.AddModule(ModuleSettings moduleSettings, IModuleMaker\* maker)
- UIMacroManager.AddModule(ModuleSettings moduleSettings, MacroEnvironment environment) → UIMacroManager.AddModule(ModuleSettings moduleSettings, MacroEnvironment environment, IModuleMaker\* maker)
The following properties were deprecated:
- ModuleSettings.Description
- MacroModule.MacroLevel
- MacroModule.Description
- Macro.Description
- MacroManage.MacroLevel
The following methods were deprecated:
- MacroModule.AddMacro()
- MacroModule.RemoveMacro()
- MacroManager.GetDocumentMacroSecurityOptions()
- MacroManager.SetDocumentMacroSecurityOptions()
- MacroManager.GetMacroManager()
- MacroManager.IsDocLvlMacro()
- UIMacroManager.EditModule()
- UIMacroManager.EditMacro()
- UIMacroManager.StepInto()
- UIMacroManager.GetMacroManager()
- UIMacroManager.GetUIDocumentMacroSecurityOptions()
- UIMacroManager.SetUIDocumentMacroSecurityOptions()
The following enums were deprecated:
- Deprecated values of enum Autodesk.Revit.DB.Macro.MacroLanguageType, only C# is supported.
- MacroLanguageType.VBNet
- MacroLanguageType.Python
- Autodesk.Revit.DB.Macros.MacroLevel
- Autodesk.Revit.DB.Macros.DocumentMacroOptions
- UIDocumentMacroOptions

## 1.4. Array API changes

The valid arrays size differs in project and family documents. The replacement method takes the document as an argument so we get an accurate answer.
- Deprecated member(s) → Replacement members(s)
- LinearArray.IsValidArraySize() → LinearArray.IsValidNumberOfMembers()
- RadialArray.IsValidArraySize() → RadialArray.IsValidNumberOfMembers()

## 1.5. BRepBuilder API changes

BRepBuilder now accepts HermiteSurface as a permitted support surface type for faces.
Changed behavior:
- BRepBuilder.IsPermittedSurfaceType() will now return true for HermiteSurface.
- BRepBuilderSurfaceGeometry.Create() can now accept a HermiteSurface and use it to create the corresponding BRepBuilderSurfaceGeometry.

## 1.6. Dimension API changes

After the introduction of LinearDimension class, when filtering for Dimensions, linear dimensions will return as type LinearDimension (child class) instead of Dimension (parent class).
Methods affected:
- typeof(), etc. : If using typeof() or .getType().Equals(), to check if a LinearDimension is Dimension, it will not work due to the nature of the derived classes.

## 1.7. Extensible Storage(Schema) API changes

Fixed two API(s) for the Extensible Storage feature:
- Document.EraseSchemaAndAllEntities() – Fixed the API to ensure it removes entities from all parts of a Revit model.
- ExtensibleStorageFilter() – Fixed the API to ensure it filters all elements associated with extensible storage data based on specific Schema IDs.

## 1.8. Electrical API changes
---

## Related Sections
- [1. API Changes - What's New in the Revit 2025 API](./2036-01_whats_new_in_the_revit_2025_api.md)
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
- [1. API Changes - 2.20.1. SheetCollection](./2036-30_2201_sheetcollection.md)
