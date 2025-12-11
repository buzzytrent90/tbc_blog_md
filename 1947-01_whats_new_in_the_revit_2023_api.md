---
author: Jeremy Tammik
optimization_date: '2025-12-11T11:24:42.684284'
original_url: https://thebuildingcoder.typepad.com/blog/1947_whats_new_2023.html
parent_post: 1947_whats_new_2023.md
part_number: '01'
part_total: '71'
post_number: '1947'
series: geometry
slug: whats_new_2023_what's_new_in_the_revit_2023_api
source_file: 1947_whats_new_2023.md
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
- selection
- sheets
- views
- walls
title: API Changes - What's New in the Revit 2023 API
word_count: 2238
---

### What's New in the Revit 2023 API
The Revit 2023 API contains many exciting enhancements for add-in developers:
- [Information sources](#1)
- [What's new in previous versions](#2)
- [Detailed table of contents](#4)
Top level table of contents:

* 1. [API changes](#4.1)

+ 1.1. [New version of CEFSharp](#4.1.1)
+ 1.2. [Replacement for Structural Analytical Model elements](#4.1.2)
+ 1.3. [Geometry API changes](#4.1.3)
+ 1.4. [Custom Export API changes](#4.1.4)
+ 1.5. [Export API changes](#4.1.5)
+ 1.6. [Energy Analysis API changes](#4.1.6)
+ 1.7. [Reinforcement API changes](#4.1.7)
+ 1.8. [Project Browser API removal](#4.1.8)
+ 1.9. [Obsolete API removal](#4.1.9)

* 2. [API additions](#4.2)

+ 2.1. [Schedule API additions](#4.2.1)
+ 2.2. [Worksharing API additions](#4.2.2)
+ 2.3. [Geometry API additions](#4.2.3)
+ 2.4. [DirectShape API additions](#4.2.4)
+ 2.5. [Import/Export API additions](#4.2.5)
+ 2.6. [View API additions](#4.2.6)
+ 2.7. [Electrical API additions](#4.2.7)
+ 2.8. [Mechanical API additions](#4.2.8)
+ 2.9. [ElementId API additions](#4.2.9)
+ 2.10. [Element API additions](#4.2.10)
+ 2.11. [Document API additions](#4.2.11)
+ 2.12. [Annotation API additions](#4.2.12)
+ 2.13. [Energy Analysis API additions](#4.2.13)
+ 2.14. [Slab API additions](#4.2.14)
+ 2.15. [Applications API additions](#4.2.15)
+ 2.16. [Sketched Elements API additions](#4.2.16)
+ 2.17. [Project Browser API additions](#4.2.17)
+ 2.18. [Structure API additions](#4.2.18)
+ 2.19. [Selection API additions](#4.2.19)
+ 2.20. [MEP API additions](#4.2.20)
+ 2.21. [Parameter API additions](#4.2.21)
+ 2.22. [Print API additions](#4.2.22)

#### Information Sources
The information below is based on the contents of the \*Revit Platform API Changes and Additions.docx\* document included with
the Revit 2023 SDK, the software developers kit available from
the [Revit Developer Centre](https://www.autodesk.com/developer-network/platform-technologies/revit).
It is also provided in the section on \*What's New\* in the Revit 2023 API help file `RevitAPI.chm` included with the SDK:
![Revit 2023 API help on What's New](img/revit_2023_api_chm.png "Revit 2023 API help on What's New")
For convenient, easy, and effective web searching, this blog post provides a cleaned-up online HTML version of that information with numbering and table of contents added, as well as the following PDF printout of the original document included in the SDK with table of contents and page numbers added:
- [Revit_Platform_API_Changes_and_Additions_2023.pdf](zip/Revit_Platform_API_Changes_and_Additions_2023_08.pdf)
The \*What's New\* section and the \*Changes and Additions\* document provide important information, both for discovering and exploring the newly added API functionality and for later reference.
If you encounter any issues migrating your existing add-ins between different versions, this is one of the first places to look.
For detailed information on all other aspects of the Revit API, please refer to the rest of the API documentation and samples provided in the SDK.
The most important things to install and always keep at hand are:
- The Revit API help file `RevitAPI.chm`
- The Visual Studio solution containing all the SDK samples, `Samples\SDKSamples.sln`
You will regularly need both for research on how to solve specific Revit API programming tasks.
More in-depth official explanations and background information is provided by the
online [Revit API Developers Guide](http://help.autodesk.com/view/RVT/2023/ENU/?guid=Revit_API_Revit_API_Developers_Guide_html) included
in the [Revit 2023 online help](http://help.autodesk.com/view/RVT/2023/ENU).
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
- [What's New in the Revit 2022 API](https://thebuildingcoder.typepad.com/blog/2021/04/whats-new-in-the-revit-2022-api.html)
- [What's New in the Revit 2022.1 API](https://thebuildingcoder.typepad.com/blog/2021/11/whats-new-in-the-revit-20221-api.html)
#### Detailed Table of Contents

* 1. [API changes](#4.1)

+ 1.1. [New version of CEFSharp](#4.1.1)
+ 1.2. [Replacement for Structural Analytical Model elements](#4.1.2)

- 1.2.1. [Analytical Model classes](#4.1.2.1)
- 1.2.2. [Analytical Models classes additions](#4.1.2.2)
- 1.2.3. [Analytical Model enums](#4.1.2.3)
- 1.2.4. [Element](#4.1.2.4)
- 1.2.5. [StructuralSettings](#4.1.2.5)
- 1.2.6. [Line Load, Area Load, Point Load](#4.1.2.6)
- 1.2.7. [Load Base](#4.1.2.7)
- 1.2.8. [Boundary Conditions](#4.1.2.8)

+ 1.3. [Geometry API changes](#4.1.3)

- 1.3.1. [Bounding Box](#4.1.3.1)
- 1.3.2. [Geometry Instance](#4.1.3.2)

+ 1.4. [Custom Export API changes](#4.1.4)
+ 1.5. [Export API changes](#4.1.5)
+ 1.6. [Energy Analysis API changes](#4.1.6)
+ 1.7. [Reinforcement API changes](#4.1.7)

- 1.7.1. [Reinforcement Elements](#4.1.7.1)

+ 1.8. [Project Browser API removal](#4.1.8)
+ 1.9. [Obsolete API removal](#4.1.9)

- 1.9.1. [Classes](#4.1.9.1)
- 1.9.2. [Methods](#4.1.9.2)
- 1.9.3. [Properties](#4.1.9.3)
- 1.9.4. [Enums](#4.1.9.4)

* 2. [API additions](#4.2)

+ 2.1. [Schedule API additions](#4.2.1)

- 2.1.1. [Schedule heights on sheets](#4.2.1.1)
- 2.1.2. [Managing schedule segments](#4.2.1.2)
- 2.1.3. [ScheduleDefinition](#4.2.1.3)

+ 2.2. [Worksharing API additions](#4.2.2)

- 2.2.1. [Delete Workset API](#4.2.2.1)

+ 2.3. [Geometry API additions](#4.2.3)

- 2.3.1. [Bounding Box API](#4.2.3.1)
- 2.3.2. [Geometry managed by symbol element](#4.2.3.2)

+ 2.4. [DirectShape API additions](#4.2.4)

- 2.4.1. [Type assignability](#4.2.4.1)

+ 2.5. [Import/Export API additions](#4.2.5)

- 2.5.1. [AXM Import](#4.2.5.1)
- 2.5.2. [OBJ Export](#4.2.5.2)
- 2.5.3. [STL and OBJ Import & Link](#4.2.5.3)
- 2.5.4. [ShapeImporter API additions](#4.2.5.4)

+ 2.6. [View API additions](#4.2.6)

- 2.6.1. [Duplicating Sheets](#4.2.6.1)
- 2.6.2. [Transforming from Model Space to View Projection Space](#4.2.6.2)
- 2.6.3. [Transforming from View Projection Space to Sheet Space](#4.2.6.3)
- 2.6.4. [View Placement on Sheet](#4.2.6.4)
- 2.6.5. [Swapping viewports on sheets to another view](#4.2.6.5)

+ 2.7. [Electrical API additions](#4.2.7)

- 2.7.1. [Load classification API](#4.2.7.1)
- 2.7.2. [Panel schedule API](#4.2.7.2)
- 2.7.3. [Electrical analytical node API](#4.2.7.3)
- 2.7.4. [Bus data API](#4.2.7.4)
- 2.7.5. [Distribution node property data API](#4.2.7.5)
- 2.7.6. [Equipment load data API](#4.2.7.6)
- 2.7.7. [Area based load type API](#4.2.7.7)
- 2.7.8. [Area based load data API](#4.2.7.8)
- 2.7.9. [Electrical load area data API](#4.2.7.9)
- 2.7.10. [Area based load boundary line data API](#4.2.7.10)
- 2.7.11. [Analytical transfer switch data API](#4.2.7.11)
- 2.7.12. [Analytical Power Source API](#4.2.7.12)

+ 2.8. [Mechanical API additions](#4.2.8)
+ 2.9. [ElementId API additions](#4.2.9)

- 2.9.1. [Converting strings to ElementId](#4.2.9.1)

+ 2.10. [Element API additions](#4.2.10)

- 2.10.1. [IsModifiable property](#4.2.10.1)
- 2.10.2. [Elements with multiple ExternalResourceReferences](#4.2.10.2)
- 2.10.3. [Category](#4.2.10.3)

+ 2.11. [Document API additions](#4.2.11)

- 2.11.1. [Elements changed since a previous version](#4.2.11.1)

+ 2.12. [Annotation API additions](#4.2.12)

- 2.12.1. [Tag Leader API](#4.2.12.1)

+ 2.13. [Energy Analysis API additions](#4.2.13)
+ 2.14. [Slab API additions](#4.2.14)
+ 2.15. [Applications API additions](#4.2.15)

- 2.15.1. [Application.ShowGraphicalOpenEndsAreaBasedLoadBoundaryDisconnects](#4.2.15.1)

+ 2.16. [Sketched Elements API additions](#4.2.16)

- 2.16.1. [Editing Sketches with SketchEditScope](#4.2.16.1)
- 2.16.2. [Dimension creation in Sketch Edit mode](#4.2.16.2)
- 2.16.3. [Filled Region for sketch plane](#4.2.16.3)
- 2.16.4. [Boundary Validation for sketched elements](#4.2.16.4)

+ 2.17. [Project Browser API additions](#4.2.17)

- 2.17.1. [BrowserOrganization](#4.2.17.1)
- 2.17.2. [ProjectBrowserOptions](#4.2.17.2)

+ 2.18. [Structure API additions](#4.2.18)

- 2.18.1. [Line Load](#4.2.18.1)
- 2.18.2. [Area Load](#4.2.18.2)
- 2.18.3. [Point Load](#4.2.18.3)
- 2.18.4. [RebarPropagation](#4.2.18.4)
- 2.18.5. [RebarHostData](#4.2.18.5)
- 2.18.6. [RebarCoupler](#4.2.18.6)

+ 2.19. [Selection API additions](#4.2.19)

- 2.19.1. [Selection](#4.2.19.1)
- 2.19.2. [UIApplication](#4.2.19.2)
- 2.19.3. [Events](#4.2.19.3)

+ 2.20. [MEP API additions](#4.2.20)

- 2.20.1. [ConnectorElement](#4.2.20.1)
- 2.20.2. [SpatialElement](#4.2.20.2)
- 2.20.3. [Flipping fabrication parts](#4.2.20.3)

+ 2.21. [Parameter API additions](#4.2.21)

- 2.21.1. [ParameterUtils](#4.2.21.1)

+ 2.22. [Print API additions](#4.2.22)

- 2.22.1. [IViewSheetSet](#4.2.22.1)

# API Changes

## 1.1. New version of CEFSharp

Revit and Autodesk add-ins use the CEFsharp library internally for several features. Some third-party add-ins do so as well. Occasionally, when different versions of the library are used, it leads to instability issues for Revit. In order to avoid version conflicts, we are clarifying what CEFsharp version is being used, and loading it prior to all add-in initializations.
- In this version, Revit uses CEFsharp version 92.0.260.

## 1.2. Replacement for Structural Analytical Model elements

The elements used for the Structural Analytical Model have been completely replaced and the interactions overhauled in this version of Revit. Because of these changes, many pre-existing API classes and methods have been removed immediately in this release as deprecation was not possible. The classes that are impacted are all in the Autodesk.Revit.DB.Structure namespace.
---

## Related Sections
- [API Changes - 1.2.1. Analytical Model classes](./1947-02_121_analytical_model_classes.md)
- [API Changes - 1.2.2. Analytical Models classes additions](./1947-03_122_analytical_models_classes_additions.md)
- [API Changes - 1.2.3. Analytical Model enums](./1947-04_123_analytical_model_enums.md)
- [API Changes - 1.2.4. Element](./1947-05_124_element.md)
- [API Changes - 1.2.5. StructuralSettings](./1947-06_125_structuralsettings.md)
- [API Changes - 1.2.6. Line Load, Area Load, Point Load](./1947-07_126_line_load_area_load_point_load.md)
- [API Changes - 1.2.7. Load Base](./1947-08_127_load_base.md)
- [API Changes - 1.2.8. Boundary Conditions](./1947-09_128_boundary_conditions.md)
- [API Changes - 1.3.1. Bounding Box](./1947-10_131_bounding_box.md)
- [API Changes - 1.3.2. Geometry Instance](./1947-11_132_geometry_instance.md)
- [API Changes - 1.7.1. Reinforcement Elements](./1947-12_171_reinforcement_elements.md)
- [API Changes - 1.9.1. Classes](./1947-13_191_classes.md)
- [API Changes - 1.9.2. Methods](./1947-14_192_methods.md)
- [API Changes - 1.9.3. Properties](./1947-15_193_properties.md)
- [API Changes - 1.9.4. Enums](./1947-16_194_enums.md)
- [API Changes - 2.1.1. Schedule heights on sheets](./1947-17_211_schedule_heights_on_sheets.md)
- [API Changes - 2.1.2. Managing schedule segments](./1947-18_212_managing_schedule_segments.md)
- [API Changes - 2.1.3. ScheduleDefinition](./1947-19_213_scheduledefinition.md)
- [API Changes - 2.2.1. Delete Workset API](./1947-20_221_delete_workset_api.md)
- [API Changes - 2.3.1. Bounding Box API](./1947-21_231_bounding_box_api.md)
- [API Changes - 2.3.2. Geometry managed by symbol element](./1947-22_232_geometry_managed_by_symbol_element.md)
- [API Changes - 2.4.1. Type assignability](./1947-23_241_type_assignability.md)
- [API Changes - 2.5.1. AXM Import](./1947-24_251_axm_import.md)
- [API Changes - 2.5.2. OBJ Export](./1947-25_252_obj_export.md)
- [API Changes - 2.5.3. STL and OBJ Import & Link](./1947-26_253_stl_and_obj_import_link.md)
- [API Changes - 2.5.4. ShapeImporter API additions](./1947-27_254_shapeimporter_api_additions.md)
- [API Changes - 2.6.1. Duplicating Sheets](./1947-28_261_duplicating_sheets.md)
- [API Changes - 2.6.2. Transforming from Model Space to View Projection Space](./1947-29_262_transforming_from_model_space_to_view_projecti.md)
- [API Changes - 2.6.3. Transforming from View Projection Space to Sheet Space](./1947-30_263_transforming_from_view_projection_space_to_she.md)
- [API Changes - 2.6.4. View Placement on Sheet](./1947-31_264_view_placement_on_sheet.md)
- [API Changes - 2.6.5. Swapping viewports on sheets to another view](./1947-32_265_swapping_viewports_on_sheets_to_another_view.md)
- [API Changes - 2.7.1. Load classification API](./1947-33_271_load_classification_api.md)
- [API Changes - 2.7.2. Panel schedule API](./1947-34_272_panel_schedule_api.md)
- [API Changes - 2.7.3. Electrical analytical node API](./1947-35_273_electrical_analytical_node_api.md)
- [API Changes - 2.7.4. Bus data API](./1947-36_274_bus_data_api.md)
- [API Changes - 2.7.5. Distribution node property data API](./1947-37_275_distribution_node_property_data_api.md)
- [API Changes - 2.7.6. Equipment load data API](./1947-38_276_equipment_load_data_api.md)
- [API Changes - 2.7.7. Area based load type API](./1947-39_277_area_based_load_type_api.md)
- [API Changes - 2.7.8. Area based load data API](./1947-40_278_area_based_load_data_api.md)
- [API Changes - 2.7.9. Electrical load area data API](./1947-41_279_electrical_load_area_data_api.md)
- [API Changes - 2.7.10. Area based load boundary line data API](./1947-42_2710_area_based_load_boundary_line_data_api.md)
- [API Changes - 2.7.11. Analytical transfer switch data API](./1947-43_2711_analytical_transfer_switch_data_api.md)
- [API Changes - 2.7.12. Analytical Power Source API](./1947-44_2712_analytical_power_source_api.md)
- [API Changes - 2.9.1. Converting strings to ElementId](./1947-45_291_converting_strings_to_elementid.md)
- [API Changes - 2.10.1. IsModifiable property](./1947-46_2101_ismodifiable_property.md)
- [API Changes - 2.10.2. Elements with multiple ExternalResourceReferences](./1947-47_2102_elements_with_multiple_externalresourcerefere.md)
- [API Changes - 2.10.3. Category](./1947-48_2103_category.md)
- [API Changes - 2.11.1. Elements changed since a previous version](./1947-49_2111_elements_changed_since_a_previous_version.md)
- [API Changes - 2.12.1. Tag Leader API](./1947-50_2121_tag_leader_api.md)
- [API Changes - 2.15.1. Application.ShowGraphicalOpenEndsAreaBasedLoadBoundaryDisconnects](./1947-51_2151_applicationshowgraphicalopenendsareabasedload.md)
- [API Changes - 2.16.1. Editing Sketches with SketchEditScope](./1947-52_2161_editing_sketches_with_sketcheditscope.md)
- [API Changes - 2.16.2. Dimension creation in Sketch Edit mode](./1947-53_2162_dimension_creation_in_sketch_edit_mode.md)
- [API Changes - 2.16.3. Filled Region for sketch plane](./1947-54_2163_filled_region_for_sketch_plane.md)
- [API Changes - 2.16.4. Boundary Validation for sketched elements](./1947-55_2164_boundary_validation_for_sketched_elements.md)
- [API Changes - 2.17.1. BrowserOrganization](./1947-56_2171_browserorganization.md)
- [API Changes - 2.17.2. ProjectBrowserOptions](./1947-57_2172_projectbrowseroptions.md)
- [API Changes - 2.18.1. Line Load](./1947-58_2181_line_load.md)
- [API Changes - 2.18.2. Area Load](./1947-59_2182_area_load.md)
- [API Changes - 2.18.3. Point Load](./1947-60_2183_point_load.md)
- [API Changes - 2.18.4. RebarPropagation](./1947-61_2184_rebarpropagation.md)
- [API Changes - 2.18.5. RebarHostData](./1947-62_2185_rebarhostdata.md)
- [API Changes - 2.18.6. RebarCoupler](./1947-63_2186_rebarcoupler.md)
- [API Changes - 2.19.1. Selection](./1947-64_2191_selection.md)
- [API Changes - 2.19.2. UIApplication](./1947-65_2192_uiapplication.md)
- [API Changes - 2.19.3. Events](./1947-66_2193_events.md)
- [API Changes - 2.20.1. ConnectorElement](./1947-67_2201_connectorelement.md)
- [API Changes - 2.20.2. SpatialElement](./1947-68_2202_spatialelement.md)
- [API Changes - 2.20.3. Flipping fabrication parts](./1947-69_2203_flipping_fabrication_parts.md)
- [API Changes - 2.21.1. ParameterUtils](./1947-70_2211_parameterutils.md)
- [API Changes - 2.22.1. IViewSheetSet](./1947-71_2221_iviewsheetset.md)
