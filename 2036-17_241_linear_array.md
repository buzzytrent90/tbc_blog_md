---
author: Jeremy Tammik
optimization_date: '2025-12-11T11:24:41.339533'
original_url: https://thebuildingcoder.typepad.com/blog/2036_whats_new_2025.html
parent_post: 2036_whats_new_2025.md
part_number: '17'
part_total: '30'
post_number: '2036'
series: geometry
slug: whats_new_2025_2.4.1._linear_array
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
title: 1. API Changes - 2.4.1. Linear Array
word_count: 549
---

### 2.4.1. Linear Array

The new methods:
- LinearArray.GetMinimumSize() – Allows users to get the minimum size of a linear array based on if the document is a family document.
- LinearArray.GetNumberOfMembersIncludingPlaceholders() – Allows users to get the number of members in a linear array, including placeholders that are still there in families with small array counts.
- LinearArray.IsValidNumberOfMembers() – Indicates whether the input count is a valid size for an array based on the document.

## 2.5. MEP Fabrication API additions

The new methods in the FabricationConfiguration class now provides support to check for bad connections between fabrication parts prior to reloading the configuration.
- CheckConnectionsForAllFabricationParts() – Allows users to check the connections for all fabrication parts in the current project. It will create reviewable warnings for all bad connections found.
- ValidateConnectionsForAllFabricationParts() – Allows users to validate all fabrication part connections in the current project. Invalid connections found will be added to the connection validation information class. The validation checks for bad alignments or gaps, incompatible connection types, mismatches of size, mismatches of shapes.
- PostReviewableWarningsForBadConnections() – Reviewable warnings are created for all entries contained in the connection validation information.
- GetUpdatedStraightsFromValidateConnections() – Allows users to get the set of element identifiers of fabrication part straights that were previously updated. If no straights were updated, it will return an empty set of element identifiers.

## 2.6. DirectShape API additions

The new method:
- DirectShapeType.SetShape(IList, DirectShapeTargetViewType) – Allows users to set a custom plan view representation of a DirectShapeType.

## 2.7. Dimension API additions

Users can now create radial, linear and arc length dimensions via the API in a project document.
The new classes:
- RadialDimension
- ArchLengthDimension
- LinearDimension
The new methods:
- RadialDimension.Create(Document, View, Reference, XYZ origin, bool isDiameterDimension)
- ArcLengthDimension.Create(Document, View, Arc, Reference, Reference firstRef, Reference secondRef)
- LinearDimension.Create(Document, View, Line, IList)

## 2.8. Electrical API additions

Added functionality for high-leg delta and single-phase distribution system. User can now create and modify high-leg delta distribution system and single-phase load in electrical analytical distribution system, and get the per phase current on each node.
The new classes:
- Autodesk.Revit.DB.Electrical.ElectricalPerPhaseData – Represents per phase values including current and load.
- Autodesk.Revit.DB.Electrical.AnalyticalPowerDistributableNodeData – Represents the data and parameters of a power distributable node.
- Autodesk.Revit.DB.Electrical.AnalyticalTransformerData – Represents the data and parameters of analytical transformer node.
The new properties:
- Electrical.DistributionSysType.HighLegPhase – Represents the high-leg phase in the 3 phase 4 wires delta distribution system.
- AnalyticalPowerSourceData.ApparentPowerRating – Represents the apparent power rating value of the analytical power source.
- AnalyticalDistributionNodePropertyData.ConnectedPhases – Represents the electrical connected phases of the electrical analytical node to its upstream node.
- AnalyticalEquipmentLoadData.PhasesNumber – Represents the number of electrical phases of the analytical equipment load.
- AnalyticalEquipmentLoadData.PowerFactorState – Represents the PowerFactorState type of the analytical equipment load.
- AreaBasedLoadData.ConnectedPhases – Represents the electrical connected phases of the area based load to its upstream node.
- AreaBasedLoadData.PhasesNumber – Allows users to set the number of electrical phases of the area based load.
- AreaBasedLoadData.PowerFactorState – Represents the power factor state of the area based load.
- AreaBasedLoadType.PowerFactorState – Represents the power factor state of the area based load type.
The new enums:
- Autodesk.Revit.DB.Electrical.ElectricalPhaseLine – Defines the electrical phase.
- Autodesk.Revit.DB.Electrical.ElectricalConnectedPhases – Defines the electrical connected phases of an electrical analytical node.

## 2.9. Energy Analysis API additions
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
