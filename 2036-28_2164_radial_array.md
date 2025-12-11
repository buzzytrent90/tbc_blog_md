---
author: Jeremy Tammik
optimization_date: '2025-12-11T11:24:41.350319'
original_url: https://thebuildingcoder.typepad.com/blog/2036_whats_new_2025.html
parent_post: 2036_whats_new_2025.md
part_number: '28'
part_total: '30'
post_number: '2036'
series: geometry
slug: whats_new_2025_2.16.4._radial_array
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
title: 1. API Changes - 2.16.4. Radial Array
word_count: 417
---

### 2.16.4. Radial Array

The new methods:
- RadialArray.GetMinimumSize() – Allows users to get the minimum size of a radial array based on if the document is a family document.
- RadialArray.GetNumberOfMembersIncludingPlaceholders() – Allows users to get the number of members in a radial array, including placeholders that are still there in families with small array counts.
- RadialArray.IsValidNumberOfMembers() – Indicates whether the input count is a valid size for an array based on the document.

## 2.17. Tag/Keynotes API additions

We now provide functionality to align multiple text, tags and keynotes with new contextual alignment tools in the ribbon.
The new class:
- Autodesk.Revit.DB.AnnotationMultipleAlignmentUtils
allows users to align annotation elements to one another. Currently supports TextNotes, Tags and Keynotes.
The new methods:
- AnnotationMultipleAlignmentUtils.ElementSupportsMultiAlign() – allows users to check whether the element type can be aligned using the multiple allignment commands.
- AnnotationMultipleAlignmentUtils.GetAnnotationOutlineWithoutLeaders() – allows users to get the four corners of the annotations bounding box, not including leaders. Outline calculations include leader/border offsets wherever applicable. (Eg. in the case of TextNotes).
- AnnotationMultipleAlignmentUtils.MoveWithAnchoredLeaders() – allows users to move the given element to the position specified by the input moveVec, while keeping the leader end points anchored.

## 2.18. Toposolid API additions

Added functionality for Toposolid smooth shading.
The new methods:
- Toposolid.ExcavateBy() – Allows users to excavate toposolid by a given element.
- Toposolid.RemoveExcavationBy() – Allows users to remove the excavation between the given element and the toposolid.
- Toposolid.CanBeExcavatedBy() – Allows users to check if the given element can be used to excavate the toposolid.
- Toposolid.SetSmoothedSurface() – Allows users to set the smoothed surface setting of toposolid category in the given document.
- Toposolid.IsSmoothedSurfaceEnabled() – Allows users to check if smoothed surface setting of toposolid category is enabled in the given document.
- Toposolid.GetIntersectingElementData()
- ToposolidType.SetContourSetting() – Allows users to set the contour setting for the current toposolid type by copying from an existing contour setting object.
- ContourSetting.IsItemEnabled()
- ContourSettingItem.GetContourSettingItemType()
- FaceToposolid.Create()
- FaceToposolid.UpdateToFace() – Allows users to reset the face toposolid to its defining face.
- FaceToposolid.GetReferencedFaces()
- FaceToposolid.SetReferencedFaces()
The new classes:
- Autodesk.Revit.DB.IntersectingElementData – Stores information of an element that intersects with another element.
- Autodesk.Revit.DB.FaceToposolid – represents a face-based Toposolid within the Autodesk Revit project.
The new enum:
- Autodesk.Revit.DB.IntersectionType – It has the following values.
- Cut
- Excavate
- Autodesk.Revit.DB.ContourSettingItemType – Represents the type of contour setting item. It has the following values:
- Single
- UnboudedRange
- BoundedRange.
The new properties:
- IntersectingElementData.IntersectionType
- IntersectingElementData.IntersectingElementId
- IntersectingElementData.IntersectedElementId
- IntersectingElementData.IntersectionVolume

## 2.19. UI API additions
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
- [1. API Changes - 2.19.1. Context Menu](./2036-29_2191_context_menu.md)
- [1. API Changes - 2.20.1. SheetCollection](./2036-30_2201_sheetcollection.md)
