---
author: Jeremy Tammik
optimization_date: '2025-12-11T11:24:41.325257'
original_url: https://thebuildingcoder.typepad.com/blog/2036_whats_new_2025.html
parent_post: 2036_whats_new_2025.md
part_number: '04'
part_total: '30'
post_number: '2036'
series: geometry
slug: whats_new_2025_1.8.2._parameter_naming
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
title: 1. API Changes - 1.8.2. Parameter Naming
word_count: 427
---

### 1.8.2. Parameter Naming

Improved naming for the following electrical parameters to remove ambiguity, improve accuracy, and better adhere with industry terminology.
- Parameter/String Attribute: Text(before) → Text(after)
- RBS_ELEC_PANEL_TOTALESTLOAD_PARAM: Total Estimated Demand → Total Demand Apparent Power
- RBS_ELEC_PANEL_TOTALLOAD_PARAM: Total Connected → Total Connected Apparent Power
- RBS_ELEC_APPARENT_LOAD_PHASEA: Apparent Load Phase A → Apparent Power Phase A
- RBS_ELEC_APPARENT_LOAD_PHASEB: Apparent Load Phase B → Apparent Power Phase B
- RBS_ELEC_APPARENT_LOAD_PHASEC: Apparent Load Phase C → Apparent Power Phase C
- RBS_ELEC_DEMAND_LOAD_PHASEA: Demand Load Phase A → Demand Apparent Power Phase A
- RBS_ELEC_DEMAND_LOAD_PHASEB: Demand Load Phase B → Demand Apparent Power Phase B
- RBS_ELEC_DEMAND_LOAD_PHASEC: Demand Load Phase C → Demand Apparent Power Phase C
- RBS_ELEC_APPARENT_LOAD: Apparent Load → Apparent Power
- RBS_ELEC_APPARENT_LOAD_PHASE1: Apparent Load Phase 1 → Apparent Power Phase 1
- RBS_ELEC_APPARENT_LOAD_PHASE2: Apparent Load Phase 2 → Apparent Power Phase 2
- RBS_ELEC_APPARENT_LOAD_PHASE3: Apparent Load Phase 3 → Apparent Power Phase 3
- RBS_ELEC_TRUE_LOAD: True Load → True Power
- RBS_ELEC_TRUE_LOAD_PHASE1: True Load Phase 1 → True Power Phase 1
- RBS_ELEC_TRUE_LOAD_PHASE2: True Load Phase 2 → True Power Phase 2
- RBS_ELEC_TRUE_LOAD_PHASE3: True Load Phase 3 → True Power Phase 3
- RBS_ELEC_TRUE_LOAD_PHASEA: True Load Phase A → True Power Phase A
- RBS_ELEC_TRUE_LOAD_PHASEB: True Load Phase B → True Power Phase B
- RBS_ELEC_TRUE_LOAD_PHASEC: True Load Phase C → True Power Phase C
- RBS_ELEC_DEMANDFACTOR_DEMANDLOAD_PARAM: Estimated Demand Load → Demand Apparent Power
- RBS_ELEC_DEMANDFACTOR_LOAD_PARAM: Connected Load → Connected Apparent Power
- RBS_ELEC_PANEL_BRANCH_CIRCUIT_APPARENT_LOAD_PHASEA: Branch Circuit Apparent Load Phase A → Branch Circuit Apparent Power Phase A
- RBS_ELEC_PANEL_BRANCH_CIRCUIT_APPARENT_LOAD_PHASEB: Branch Circuit Apparent Load Phase B → Branch Circuit Apparent Power Phase B
- RBS_ELEC_PANEL_BRANCH_CIRCUIT_APPARENT_LOAD_PHASEC: Branch Circuit Apparent Load Phase C → Branch Circuit Apparent Power Phase C
- RBS_ELEC_PANEL_FEED_THRU_LUGS_APPARENT_LOAD_PHASEA: Feed Through Lugs Apparent Load Phase A → Feed Through Lugs Apparent Power Phase A
- RBS_ELEC_PANEL_FEED_THRU_LUGS_APPARENT_LOAD_PHASEB: Feed Through Lugs Apparent Load Phase B → Feed Through Lugs Apparent Power Phase B
- RBS_ELEC_PANEL_FEED_THRU_LUGS_APPARENT_LOAD_PHASEC: Feed Through Lugs Apparent Load Phase C → Feed Through Lugs Apparent Power Phase C
- RBS_ELEC_PANEL_TOTALESTLOAD_OTHER_PARAM: Other Total Estimated Demand → Other Total Demand Apparent Power
- RBS_ELEC_LOADSUMMARY_CONNECTED_LOAD_PARAM: Connected Load (VA) → Connected Apparent Power
- RBS_ELEC_LOADSUMMARY_DEMAND_LOAD_PARAM: Estimated Demand (VA) → Demand Apparent Power
- RBS_ELEC_PANEL_TOTALESTLOAD_POWER_PARAM: Power Total Estimated Demand → Power Total Demand Apparent Power
- RBS_ELEC_PANEL_TOTALESTLOAD_LIGHT_PARAM: Lighting Total Estimated Demand → Lighting Total Demand Apparent Power
- RBS_ELEC_PANEL_TOTALESTLOAD_HVAC_PARAM: HVAC Total Estimated Demand → HVAC Total Demand Apparent Power
- RBS_ELEC_PANEL_TOTALLOAD_HVAC_PARAM: HVAC Total Connected → HVAC Total Connected Apparent Power
- RBS_ELEC_PANEL_TOTALLOAD_LIGHT_PARAM: Lighting Total Connected → Lighting Total Connected Apparent Power
- RBS_ELEC_PANEL_TOTALLOAD_POWER_PARAM: Power Total Connected → Power Total Connected Apparent Power
- RBS_ELEC_PANEL_TOTALLOAD_OTHER_PARAM: Other Total Connected → Other Total Connected Apparent Power

## 1.9. Link Visibility/Graphic Override API changes

The following methods were modified to support custom settings type.
- Revit.DB.RevitLinkGraphicsSettings.LinkVisibilityType
- Revit.DB.View.SetLinkOverrides(ElementId, RevitLinkGraphicsSettings) – Supports RevitLinkGraphicsSettings of Custom type.
- Revit.DB.View.GetLinkOverrides(ElementId) – Removed two validators:
- AreGraphicsOverridesAllowed()
- IsLinkOverridesSupported()

## 1.10. MEP changes
---

## Related Sections
- [1. API Changes - What's New in the Revit 2025 API](./2036-01_whats_new_in_the_revit_2025_api.md)
- [1. API Changes - 1.3.1. MacroManager API](./2036-02_131_macromanager_api.md)
- [1. API Changes - 1.8.1. Distribution system](./2036-03_181_distribution_system.md)
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
