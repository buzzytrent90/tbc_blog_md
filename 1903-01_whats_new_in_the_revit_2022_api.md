---
author: Jeremy Tammik
optimization_date: '2025-12-11T11:24:36.684250'
original_url: https://thebuildingcoder.typepad.com/blog/1903_whats_new_2022.html
parent_post: 1903_whats_new_2022.md
part_number: '01'
part_total: '61'
post_number: '1903'
series: geometry
slug: whats_new_2022_what's_new_in_the_revit_2022_api
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
title: API Changes - What's New in the Revit 2022 API
word_count: 2440
---

### What's New in the Revit 2022 API
The Revit 2022 API includes an exceptional list of exciting enhancements for add-in developers, including numerous developer wishes and requests that have now been explicitly addressed:
- [Information sources](#1)
- [What's new in previous versions](#2)
- [Detailed table of contents](#4)
Table of contents top level:

* 1 [API Changes](#4.1)

+ 1.1 [Parameter API changes (migration to ForgeTypeId)](#4.1.1)
+ 1.2 [Renamed/replaced Parameter Group members](#4.1.2)
+ 1.3 [Document & Worksharing API changes](#4.1.3)
+ 1.4 [Sketched element API changes](#4.1.4)
+ 1.5 [IndependentTag API Changes](#4.1.5)
+ 1.6 [Revision API changes](#4.1.6)
+ 1.7 [Structural reinforcement API changes](#4.1.7)
+ 1.8 [Import/Export API changes](#4.1.8)
+ 1.9 [MEP API changes](#4.1.9)
+ 1.10 [PostableCommand enumeration update](#4.1.10)
+ 1.11 [BuiltInFailures changes](#4.1.11)
+ 1.12 [Filter API change](#4.1.12)
+ 1.13 [Ruby macros no longer supported](#4.1.13)
+ 1.14 [TransmissionData API change](#4.1.14)
+ 1.15 [Removal of obsoleted APIs](#4.1.15)

* 2 [API Additions](#4.2)

+ 2.1 [Sketched Elements API](#4.2.1)
+ 2.2 [Color Fill API](#4.2.2)
+ 2.3 [Export API additions](#4.2.3)
+ 2.4 [Civil Alignments API](#4.2.4)
+ 2.5 [DirectShape API Additions](#4.2.5)
+ 2.6 [Document & Worksharing API additions](#4.2.6)
+ 2.7 [View API additions](#4.2.7)
+ 2.8 [Graphics API additions](#4.2.8)
+ 2.9 [Label API additions](#4.2.9)
+ 2.10 [Parameter API additions](#4.2.10)
+ 2.11 [Dimension API additions](#4.2.11)
+ 2.12 [Category API additions](#4.2.12)
+ 2.13 [Shared Coordinates API additions](#4.2.13)
+ 2.14 [Revision API additions](#4.2.14)
+ 2.15 [Phasing API additions](#4.2.15)
+ 2.16 [Geometry API additions](#4.2.16)
+ 2.17 [Point Cloud API additions](#4.2.17)
+ 2.18 [Schedule API additions](#4.2.18)
+ 2.19 [Import API additions](#4.2.19)
+ 2.20 [Structural API additions](#4.2.20)
+ 2.21 [MEP API additions](#4.2.21)
+ 2.22 [Energy Analysis API additions](#4.2.22)

#### Information Sources
The information below is based on the contents of the \*Revit Platform API Changes and Additions.docx\* document included with
the Revit 2022 SDK, the software developers kit available from
the [Revit Developer Centre](https://www.autodesk.com/developer-network/platform-technologies/revit).
It is also provided in the section on \*What's New\* in the Revit 2022 API help file `RevitAPI.chm` included with the SDK:
![Revit 2022 API help on What's New](img/whats_new_2022.png "Revit 2022 API help on What's New")
For convenient, easy, and effective web searching, this blog post provides a cleaned-up online HTML version of that information with numbering and table of contents added, as well as the following PDF printout of the original document included in the SDK:
- [Revit_Platform_API_Changes_and_Additions_2022.pdf](zip/Revit_Platform_API_Changes_and_Additions_2022.pdf)
The \*What's New\* section and the \*Changes and Additions\* document provide important information, both for discovering and exploring the newly added API functionality and for later reference.
If you encounter any issues migrating your existing add-ins between different versions, this is one of the first places to look.
For detailed information on all other aspects of the Revit API, please refer to the rest of the API documentation and samples provided in the SDK.
The most important things to install and always keep at hand are:
- The Revit API help file `RevitAPI.chm`
- The Visual Studio solution containing all the SDK samples, `Samples\SDKSamples.sln`
You will regularly need both for research on how to solve specific Revit API programming tasks.
More in-depth official explanations and background information is provided by the
online [Revit API Developers Guide](http://help.autodesk.com/view/RVT/2022/ENU/?guid=Revit_API_Revit_API_Developers_Guide_html) included
in the [Revit 2022 online help](http://help.autodesk.com/view/RVT/2022/ENU).
#### What's New in Previous Versions
Here are links to previous discussions of \*What's New in the Revit API\*:
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
- [Revit 2018.1 and the Visual Materials API](http://thebuildingcoder.typepad.com/blog/2017/08/revit-20181-and-the-visual-materials-api.html)
- [Revit 2018.1.1 and 2018.1 API Docs Online](http://thebuildingcoder.typepad.com/blog/2017/09/revit-201811-fixes-cropbox-setting.html)
- [Revit 2018.1 Visual Materials API](http://thebuildingcoder.typepad.com/blog/2017/11/modifying-material-visual-appearance.html)
- [What's New in the Revit 2018.2 API](http://thebuildingcoder.typepad.com/blog/2017/12/whats-new-in-the-revit-20182-api.html)
- [What's New in the Revit 2019 API](http://thebuildingcoder.typepad.com/blog/2018/04/whats-new-in-the-revit-2019-api.html)
- [What's New in the Revit 2020 API](https://thebuildingcoder.typepad.com/blog/2019/04/whats-new-in-the-revit-2020-api.html)
- [What's New in the Revit 2020.1 API](https://thebuildingcoder.typepad.com/blog/2019/09/whats-new-in-the-revit-20201-api.html)
- [What's New in the Revit 2021 API](https://thebuildingcoder.typepad.com/blog/2020/04/whats-new-in-the-revit-2021-api.html)
- [What's New in the Revit 2021.1 API](https://thebuildingcoder.typepad.com/blog/2020/08/revit-20211-sdk-and-whats-new.html)
#### Detailed Table of Contents

* 1 [API Changes](#4.1)

+ 1.1 [Parameter API changes (migration to ForgeTypeId)](#4.1.1)

- 1.1.1 [Changes to Parameter Access APIs](#4.1.1.1)
- 1.1.2 [Changes to Parameter Definition APIs](#4.1.1.2)
- 1.1.3 [Changes to Shared Parameter Creation and Binding APIs](#4.1.1.3)
- 1.1.4 [Changes to GlobalParameter APIs](#4.1.1.4)
- 1.1.5 [New and changed utility APIs](#4.1.1.5)

+ 1.2 [Renamed/replaced Parameter Group members](#4.1.2)
+ 1.3 [Document & Worksharing API changes](#4.1.3)

- 1.3.1 [SaveAsCloudModel() now supports save of a new Revit Cloud Worksharing central model](#4.1.3.1)
- 1.3.2 [Revit Links to Cloud Models](#4.1.3.2)
- 1.3.3 [Worksharing API exception changes](#4.1.3.3)
- 1.3.4 [Worksharing API replacements](#4.1.3.4)

+ 1.4 [Sketched element API changes](#4.1.4)

- 1.4.1 [Floor creation API replacements](#4.1.4.1)

+ 1.5 [IndependentTag API Changes](#4.1.5)

- 1.5.1 [Rotation](#4.1.5.1)
- 1.5.2 [Multiple References](#4.1.5.2)

+ 1.6 [Revision API changes](#4.1.6)
+ 1.7 [Structural reinforcement API changes](#4.1.7)
+ 1.8 [Import/Export API changes](#4.1.8)

- 1.8.1 [Import() and Link() methods now treat View as optional](#4.1.8.1)
- 1.8.2 [Export API replacements](#4.1.8.2)

+ 1.9 [MEP API changes](#4.1.9)

- 1.9.1 [Electrical API changes](#4.1.9.1)
- 1.9.2 [Fabrication API changes](#4.1.9.2)

+ 1.10 [PostableCommand enumeration update](#4.1.10)
+ 1.11 [BuiltInFailures changes](#4.1.11)
+ 1.12 [Filter API change](#4.1.12)
+ 1.13 [Ruby macros no longer supported](#4.1.13)
+ 1.14 [TransmissionData API change](#4.1.14)
+ 1.15 [Removal of obsoleted APIs](#4.1.15)

* 2 [API Additions](#4.2)

+ 2.1 [Sketched Elements API](#4.2.1)

- 2.1.1 [Ceiling creation](#4.2.1.1)
- 2.1.2 [Floor APIs](#4.2.1.2)
- 2.1.3 [Wall APIs](#4.2.1.3)
- 2.1.4 [Slanted and Tapered Walls](#4.2.1.4)
- 2.1.5 [Sketch APIs](#4.2.1.5)
- 2.1.6 [Editing Sketches with SketchEditScope](#4.2.1.6)
- 2.1.7 [Boundary validation for sketched elements](#4.2.1.7)
- 2.1.8 [CompoundStructure API](#4.2.1.8)

+ 2.2 [Color Fill API](#4.2.2)

- 2.2.1 [Color Fill Schemes](#4.2.2.1)
- 2.2.2 [Color Fill Scheme Entries](#4.2.2.2)
- 2.2.3 [View access to Color Fills](#4.2.2.3)
- 2.2.4 [Color Fill Legends](#4.2.2.4)

+ 2.3 [Export API additions](#4.2.3)

- 2.3.1 [Export to PDF](#4.2.3.1)
- 2.3.2 [PDF Export Settings](#4.2.3.2)

+ 2.4 [Civil Alignments API](#4.2.4)
+ 2.5 [DirectShape API Additions](#4.2.5)

- 2.5.1 [Reference Curves, Planes, and Points](#4.2.5.1)
- 2.5.2 [External Geometry](#4.2.5.2)
- 2.5.3 [Custom Family names](#4.2.5.3)

+ 2.6 [Document & Worksharing API additions](#4.2.6)

- 2.6.1 [Cloud Model API additions](#4.2.6.1)
- 2.6.2 [OpenOptions API](#4.2.6.2)

+ 2.7 [View API additions](#4.2.7)

- 2.7.1 [Label for views on sheets (Viewport)](#4.2.7.1)
- 2.7.2 [Callout view](#4.2.7.2)
- 2.7.3 [Grids in 3D views](#4.2.7.3)

+ 2.8 [Graphics API additions](#4.2.8)

- 2.8.1 [Temporary in-canvas graphics](#4.2.8.1)

+ 2.9 [Label API additions](#4.2.9)
+ 2.10 [Parameter API additions](#4.2.10)
+ 2.11 [Dimension API additions](#4.2.11)

- 2.11.1 [Dimension layout](#4.2.11.1)
- 2.11.2 [DimensionType prefix/suffix](#4.2.11.2)

+ 2.12 [Category API additions](#4.2.12)

- 2.12.1 [Built in Categories](#4.2.12.1)

+ 2.13 [Shared Coordinates API additions](#4.2.13)

- 2.13.1 [Reset Shared Coordinates](#4.2.13.1)
- 2.13.2 [Clipped state of BasePoint](#4.2.13.2)

+ 2.14 [Revision API additions](#4.2.14)
+ 2.15 [Phasing API additions](#4.2.15)
+ 2.16 [Geometry API additions](#4.2.16)
+ 2.17 [Point Cloud API additions](#4.2.17)
+ 2.18 [Schedule API additions](#4.2.18)

- 2.18.1 [Multiple Value Indication – schedule customization](#4.2.18.1)

+ 2.19 [Import API additions](#4.2.19)

- 2.19.1 [Import/Link Rhino](#4.2.19.1)

+ 2.20 [Structural API additions](#4.2.20)

- 2.20.1 [Moving individual rebar in a Rebar Set](#4.2.20.1)
- 2.20.2 [Removing individual bars from a Rebar Set](#4.2.20.2)
- 2.20.3 [Moving and Removing bars owned by PathReinforcement](#4.2.20.3)
- 2.20.4 [Area Reinforcement Layers](#4.2.20.4)
- 2.20.5 [Rebar conversion](#4.2.20.5)
- 2.20.6 [Rebar geometry](#4.2.20.6)
- 2.20.7 [Freeform Rebar](#4.2.20.7)

+ 2.21 [MEP API additions](#4.2.21)

- 2.21.1 [Building and Space Type additions](#4.2.21.1)
- 2.21.2 [Zone additions](#4.2.21.2)
- 2.21.3 [Mechanical Systems Analysis Set Points](#4.2.21.3)
- 2.21.4 [Analysis Report Style](#4.2.21.4)
- 2.21.5 [MEP Hidden Line Settings](#4.2.21.5)
- 2.21.6 [Electrical Panel Schedule](#4.2.21.6)
- 2.21.7 [Conversion to Fabrication](#4.2.21.7)

+ 2.22 [Energy Analysis API additions](#4.2.22)

# API Changes

## 1.1. Parameter API changes (migration to ForgeTypeId)

In Revit 2021, Revit API functionality for units of measurement migrated from enumerated identifiers to extensible identifiers represented by the ForgeTypeId class. ForgeTypeId is now used as the identifier type for more data structures in the Revit API.
A ForgeTypeId instance holds a string, called a "typeid", that uniquely identifies a Forge schema. A Forge schema is a JSON document describing a data structure, supporting data interchange between applications. A typeid string includes a namespace and version number and may look something like "autodesk.spec.aec:length-1.0.0" or "autodesk.unit.unit:meters-1.0.0". By default, comparison of ForgeTypeId values in the Revit API ignores the version number.
The new classes:
- Autodesk.Revit.DB.DisciplineTypeId
- Autodesk.Revit.DB.GroupTypeId
- Autodesk.Revit.DB.ParameterTypeId
contain properties of type ForgeTypeId, following the pattern established in Revit 2021 with the UnitTypeId, SymbolTypeId, and SpecTypeId classes. Values from the DisciplineTypeId class can be used in code to replace values of the deprecated UnitGroup enumeration. For example, where you previously used UnitGroup.Structural, you would now use DisciplineTypeId.Structural. Values from the GroupTypeId and ParameterTypeId classes can be used to replace values of the BuiltInParameter and BuiltInParameterGroup enumerations, respectively, which have not yet been deprecated.
The SpecTypeId class contains several new nested classes containing ForgeTypeId properties identifying non-floating-point data types, such as integers and strings. ForgeTypeId properties in SpecTypeId and its nested classes can be used to replace values of the deprecated ParameterType enumeration. This table lists the deprecated types and their replacements:
- Deprecated enumeration → Replacement
- UnitGroup → DisciplineTypeId
- ParameterType → SpecTypeId, SpecTypeId.Boolean, SpecTypeId.Int, SpecTypeId.Reference, SpecTypeId.String
Here are some further migrated types where the original enumeration is not yet fully deprecated:
- Original enumeration → Preferred replacements as ForgeTypeId
- BuiltInParameterGroup → GroupTypeId
- BuiltInParameter → ParameterTypeId
The mapping of enumerations to different ForgeTypeId members has implications for previously existing APIs related to parameter access, parameter definitions, shared parameter creation and binding, global parameters and other utilities.
---

## Related Sections
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
- [API Changes - 2.2.3. View access to Color Fills](./1903-28_223_view_access_to_color_fills.md)
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
