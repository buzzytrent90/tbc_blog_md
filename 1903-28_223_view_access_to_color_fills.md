---
author: Jeremy Tammik
optimization_date: '2025-12-11T11:24:36.716884'
original_url: https://thebuildingcoder.typepad.com/blog/1903_whats_new_2022.html
parent_post: 1903_whats_new_2022.md
part_number: '28'
part_total: '61'
post_number: '1903'
series: geometry
slug: whats_new_2022_2.2.3._view_access_to_color_fills
source_file: 1903_whats_new_2022.md
tags:
- elements
- family
- filtering
- geometry
- levels
- parameters
- references
- revit-api
- rooms
- schedules
- sheets
- transactions
- views
- walls
title: API Changes - 2.2.3. View access to Color Fills
word_count: 33
---

### 2.2.3. View access to Color Fills

The new members:
- View.GetColorFillSchemeId()
- View.SetColorFillSchemeId()
provide access to read and apply the color fill scheme associated with a particular category in the view.
---

## Related Sections
- [API Changes - What's New in the Revit 2022 API](./1903-01_whats_new_in_the_revit_2022_api.md)
- [API Changes - 1.1.1. Changes to Parameter Access APIs](./1903-02_111_changes_to_parameter_access_apis.md)
- [API Changes - 1.1.2. Changes to Parameter Definition APIs](./1903-03_112_changes_to_parameter_definition_apis.md)
- [API Changes - 1.1.3. Changes to Shared Parameter Creation and Binding APIs](./1903-04_113_changes_to_shared_parameter_creation_and_bindi.md)
- [API Changes - 1.1.4. Changes to GlobalParameter APIs](./1903-05_114_changes_to_globalparameter_apis.md)
- [API Changes - 1.1.5. New and changed utility APIs](./1903-06_115_new_and_changed_utility_apis.md)
- [API Changes - 1.3.1. SaveAsCloudModel() now supports save of a new Revit Cloud Worksharing central model](./1903-07_131_saveascloudmodel_now_supports_save_of_a_new_re.md)
- [API Changes - 1.3.2. Revit Links to Cloud Models](./1903-08_132_revit_links_to_cloud_models.md)
- [API Changes - 1.3.3. Worksharing API exception changes](./1903-09_133_worksharing_api_exception_changes.md)
- [API Changes - 1.3.4. Worksharing API replacements](./1903-10_134_worksharing_api_replacements.md)
- [API Changes - 1.4.1. Floor creation API replacements](./1903-11_141_floor_creation_api_replacements.md)
- [API Changes - 1.5.1. Rotation](./1903-12_151_rotation.md)
- [API Changes - 1.5.2. Multiple References](./1903-13_152_multiple_references.md)
- [API Changes - 1.8.1. Import() and Link() methods now treat View as optional](./1903-14_181_import_and_link_methods_now_treat_view_as_opti.md)
- [API Changes - 1.8.2. Export API replacements](./1903-15_182_export_api_replacements.md)
- [API Changes - 1.9.1. Electrical API changes](./1903-16_191_electrical_api_changes.md)
- [API Changes - 1.9.2. Fabrication API changes](./1903-17_192_fabrication_api_changes.md)
- [API Changes - 2.1.1. Ceiling creation](./1903-18_211_ceiling_creation.md)
- [API Changes - 2.1.2. Floor APIs](./1903-19_212_floor_apis.md)
- [API Changes - 2.1.3. Wall APIs](./1903-20_213_wall_apis.md)
- [API Changes - 2.1.4. Slanted and Tapered Walls](./1903-21_214_slanted_and_tapered_walls.md)
- [API Changes - 2.1.5. Sketch APIs](./1903-22_215_sketch_apis.md)
- [API Changes - 2.1.6. Editing Sketches with SketchEditScope](./1903-23_216_editing_sketches_with_sketcheditscope.md)
- [API Changes - 2.1.7. Boundary validation for sketched elements](./1903-24_217_boundary_validation_for_sketched_elements.md)
- [API Changes - 2.1.8. CompoundStructure API](./1903-25_218_compoundstructure_api.md)
- [API Changes - 2.2.1. Color Fill Schemes](./1903-26_221_color_fill_schemes.md)
- [API Changes - 2.2.2. Color Fill Scheme Entries](./1903-27_222_color_fill_scheme_entries.md)
- [API Changes - 2.2.4. Color Fill Legends](./1903-29_224_color_fill_legends.md)
- [API Changes - 2.3.1. Export to PDF](./1903-30_231_export_to_pdf.md)
- [API Changes - 2.3.2. PDF Export Settings](./1903-31_232_pdf_export_settings.md)
- [API Changes - 2.5.1. Reference Curves, Planes, and Points](./1903-32_251_reference_curves_planes_and_points.md)
- [API Changes - 2.5.2. External Geometry](./1903-33_252_external_geometry.md)
- [API Changes - 2.5.3. Custom Family names](./1903-34_253_custom_family_names.md)
- [API Changes - 2.6.1. Cloud Model API additions](./1903-35_261_cloud_model_api_additions.md)
- [API Changes - 2.6.2. OpenOptions API](./1903-36_262_openoptions_api.md)
- [API Changes - 2.7.1. Label for views on sheets (Viewport)](./1903-37_271_label_for_views_on_sheets_viewport.md)
- [API Changes - 2.7.2. Callout view](./1903-38_272_callout_view.md)
- [API Changes - 2.7.3. Grids in 3D views](./1903-39_273_grids_in_3d_views.md)
- [API Changes - 2.8.1. Temporary in-canvas graphics](./1903-40_281_temporary_in_canvas_graphics.md)
- [API Changes - 2.11.1. Dimension layout](./1903-41_2111_dimension_layout.md)
- [API Changes - 2.11.2. DimensionType prefix/suffix](./1903-42_2112_dimensiontype_prefixsuffix.md)
- [API Changes - 2.12.1. Built in Categories](./1903-43_2121_built_in_categories.md)
- [API Changes - 2.13.1. Reset Shared Coordinates](./1903-44_2131_reset_shared_coordinates.md)
- [API Changes - 2.13.2. Clipped state of BasePoint](./1903-45_2132_clipped_state_of_basepoint.md)
- [API Changes - 2.18.1. Multiple Value Indication – schedule customization](./1903-46_2181_multiple_value_indication_schedule_customizat.md)
- [API Changes - 2.19.1. Import/Link Rhino](./1903-47_2191_importlink_rhino.md)
- [API Changes - 2.20.1. Moving individual rebar in a Rebar Set](./1903-48_2201_moving_individual_rebar_in_a_rebar_set.md)
- [API Changes - 2.20.2. Removing individual bars from a Rebar Set](./1903-49_2202_removing_individual_bars_from_a_rebar_set.md)
- [API Changes - 2.20.3. Moving and Removing bars owned by PathReinforcement](./1903-50_2203_moving_and_removing_bars_owned_by_pathreinfor.md)
- [API Changes - 2.20.4. Area Reinforcement Layers](./1903-51_2204_area_reinforcement_layers.md)
- [API Changes - 2.20.5. Rebar conversion](./1903-52_2205_rebar_conversion.md)
- [API Changes - 2.20.6. Rebar geometry](./1903-53_2206_rebar_geometry.md)
- [API Changes - 2.20.7. Freeform Rebar](./1903-54_2207_freeform_rebar.md)
- [API Changes - 2.21.1. Building and Space Type additions](./1903-55_2211_building_and_space_type_additions.md)
- [API Changes - 2.21.2. Zone additions](./1903-56_2212_zone_additions.md)
- [API Changes - 2.21.3. Mechanical Systems Analysis Set Points](./1903-57_2213_mechanical_systems_analysis_set_points.md)
- [API Changes - 2.21.4. Analysis Report Style](./1903-58_2214_analysis_report_style.md)
- [API Changes - 2.21.5. MEP Hidden Line Settings](./1903-59_2215_mep_hidden_line_settings.md)
- [API Changes - 2.21.6. Electrical Panel Schedule](./1903-60_2216_electrical_panel_schedule.md)
- [API Changes - 2.21.7. Conversion to Fabrication](./1903-61_2217_conversion_to_fabrication.md)
