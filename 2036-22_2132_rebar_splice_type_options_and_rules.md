---
author: Jeremy Tammik
optimization_date: '2025-12-11T11:24:41.345013'
original_url: https://thebuildingcoder.typepad.com/blog/2036_whats_new_2025.html
parent_post: 2036_whats_new_2025.md
part_number: '22'
part_total: '30'
post_number: '2036'
series: geometry
slug: whats_new_2025_2.13.2._rebar_splice_type_options_and_rules
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
title: 1. API Changes - 2.13.2. Rebar splice type options and rules
word_count: 956
---

### 2.13.2. Rebar splice type options and rules

Added functionality for splicing the rebar. Users can now splice a rebar, remove the splice (keeping the bars separated), unify into one bar, modify the data related to splice and modify the constraints of the spliced bars seeing as splice chain.
The new classes:
- Autodesk.Revit.DB.Structure.RebarSpliceTypeUtils – Utility class for dealing with rebar splice type operations.
- Autodesk.Revit.DB.Structure.RebarSpliceOptions
- Autodesk.Revit.DB.Structure.RebarSpliceGeometry – This class consists of a vector and a point which will be projected to the nearest rebar curve.
- Autodesk.Revit.DB.Structure.RebarSpliceRules
- Autodesk.Revit.DB.Structure.RebarSplice – A class that can used to access the data between two connected rebars.
- Autodesk.Revit.DB.Structure.RebarSpliceByRulesResult
- Autodesk.Revit.DB.Structure.RebarSpliceUtils
The new methods:
- RebarBarType.GetLapLength()
- RebarBarType.SetLapLength()
- RebarBarType.GetAutoCalculatedLapLength()
- RebarBarType.SetAutoCalculatedLapLength()
- RebarBarType.GetStaggerLength)
- RebarBarType.SetStaggerLength()
- RebarBarType.GetAutoCalculatedStaggerLength()
- RebarBarType.SetAutoCalculatedStaggerLength()
- Rebar.GetRebarSplice()
- Rebar.RemoveSplice()
- Rebar.GetLapLength()
- Rebar.GetSpliceStaggerLength()
- RebarConstrainedHandle.GetHandleSurface()
- RebarConstrainedHandle.Move()
- RebarConstrainedHandle.CanSetBehavior()
- RebarConstrainedHandle.GetPossibleHandleBehaviors()
- RebarHostData.GetRebarHostDirectNeighbors() – Returns the first level of neighbors for the provided host that can host reinforcement, i.e. elements that are joined directly to the provided host element and not the neighbors of the joined elements.
The new properties:
- RebarSpliceOptions.SpliceTypeId
- RebarSpliceOptions.SplicePosition
- Rebar.CanHaveVaryingLengthBars
- Rebar.HasVariableLengthBars
- RebarShapeDrivenAccessor.UseRebarConstraintsToProduceVaryingBars
- RebarConstrainedHandle.HandleBehavior
The new enums:
- RebarSplicePosition – Describes the position of the splice. It has the following values:
- End1 – Lap is towards the start of the splice chain.
- Middle – Lap goes into both directions.
- End2 – Lap is towards the end of the splice chain.
- RebarSpliceShiftOption – Describes the way bars are shifted in the splice relation. It has the following values:
- BarPlane – Represents the bar plane is shifted so that the spliced rebars are not clashing.
- None – The bars are not shifted at all.
- RebarSpliceByRulesRunOutPosition – Describes the run-out position. It has the following values:
- Start – Represents the rest will remain at the start of the bar.
- End – Represents the rest will remain at the end of the bar
- RebarSpliceError – Represents the states for splicing a Rebar. It has the following values:
- Success
- Unknown – Represents there is an unexpected error.
- InvalidRebar – Represents free form rebars or shape driven rebars that are multiplanar or having shape whose definition is RebarShapeDefinitionByArc or rebars part of a group that cannot be spliced.
- InvalidLineOrLinePlaneNormal – Represents the line length is zero or the line direction is parallel with the line plane normal.
- LineDoesNotIntersectRebarBoundingBox – Represents the line doesn't intersect the rebar bounding box.
- SpliceGeometryOnHookOrFillet – Represents if the splice geometry is on a hook or a fillet, the rebar can't be spliced with it.
- TooSmallSegments – Represents one of the resulting segments is too small to apply the lap.
- SpliceGeometryDoesNotIntersectAllTheBarsInTheSet – Represents a plane obtained from splice geometry doesn't intersect all the bars in the set.
- SpliceGeometryAlmostParallelToBarSegment – Represents the plane formed by splice geometry is almost parallel to bar segment plane.
- RebarSpliceByRulesError – Represents the states for splicing a Rebar by rules. It has the following values:
- Success
- Unknown – Represents there is an unexpected error.
- InvalidRebar – Represents free form rebars or shape driven rebars that are multiplanar or having shape whose definition is RebarShapeDefinitionByArc or rebars part of a group that cannot be spliced.
- TooBigHook – Represents the hook lengths exceed the maximum length.
- TooSmallRunOut – Represents the run-out is below minimum length, or the lap can't be applied to it.
- MaximumLengthBiggerThanBarLength – Represents the maximum length exceeds the bar length.
- TooBigArc – Represents the arc segment exceeds the maximum length.
- CantSpliceAllTheBarsInSet – Represents some bars in the varying set are not intersected by the resulting splice geometries.
- LapLengthBiggerThanMaximumBarLength – Represents the lap length is greater than the maximum length.
- InvalidCombinationOfMaximumMinimumBarLengthAndLapLength – Represents the combination of the maximum bar length, minimum bar length and lap length is invalid.
- RebarHandleBehavior – Represents different behaviors that can be applied to a RebarConstrainedHandle. It has the following values:
- Default – Represents the default behavior of a RebarConstrainedHandle.
- SpliceMainEndOnEnd1Position
- SpliceMainEndOnMiddlePosition – Represents the behavior can be set to a StartOfBar or EndOfBar RebarConstrainedHandle of a bar that is part of splice. On the connected bar there is a ToOtherRebar constraint whose target is the current rebar. The RebarConstrainedHandle's plane in the same position as the splice plane for Middle.
- SpliceMainEndOnEnd2Position – Represents the behavior can be set to a StartOfBar or EndOfBar RebarConstrainedHandle of a bar that is part of splice. On the connected bar there is a ToOtherRebar constraint whose target is the current rebar. The RebarConstrainedHandle's plane in the same position as the splice plane for Middle
- SpliceConnectedEndOnEnd1Position – Represents the behavior can be set to a StartOfBar or EndOfBar RebarConstrainedHandle of a bar that is part of splice. On this RebarConstrainedHandle is a ToOtherRebar constraint whose target is the other bar involved in splice. The RebarConstrainedHandle's plane in the same position as the splice plane for End1.
- SpliceConnectedEndOnMiddlePosition – Represents the behavior can be set to a StartOfBar or EndOfBar RebarConstrainedHandle of a bar that is part of splice. On this RebarConstrainedHandle is a ToOtherRebar constraint whose target is the other bar involved in splice. The RebarConstrainedHandle's plane in the same position as the splice plane for Middle
- SpliceConnectedEndOnEnd2Position – Represents the behavior can be set to a StartOfBar or EndOfBar RebarConstrainedHandle of a bar that is part of splice. On this RebarConstrainedHandle is a ToOtherRebar constraint whose target is the other bar involved in splice. RebarConstrainedHandle's plane in the same position as the splice plane for End2
- SpliceRebarPlaneOnSpliceSetExtent – Represents the behavior can be set to a RebarPlane RebarConstrainedHandle of a bar that is part of splice
- SpliceOutOfPlaneExtentOnSpliceSetExtent – Represents the behavior can be set to a OutOfPlaneExtent RebarConstrainedHandle of a bar that is part of splice.
- SpliceEdge – Represents the behavior can be set to an edge segment that is connected to the other rebar of splice.

## 2.14. Selection API additions
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
- [1. API Changes - 2.14.1. UI Application](./2036-23_2141_ui_application.md)
- [1. API Changes - 2.15.1. Wall APIs](./2036-24_2151_wall_apis.md)
- [1. API Changes - 2.16.1. Analytical Elements](./2036-25_2161_analytical_elements.md)
- [1. API Changes - 2.16.2. Analytical Surface](./2036-26_2162_analytical_surface.md)
- [1. API Changes - 2.16.3. Bending Details on Drawings](./2036-27_2163_bending_details_on_drawings.md)
- [1. API Changes - 2.16.4. Radial Array](./2036-28_2164_radial_array.md)
- [1. API Changes - 2.19.1. Context Menu](./2036-29_2191_context_menu.md)
- [1. API Changes - 2.20.1. SheetCollection](./2036-30_2201_sheetcollection.md)
