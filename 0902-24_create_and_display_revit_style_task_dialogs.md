---
author: Jeremy Tammik
optimization_date: '2025-12-11T11:24:06.663004'
original_url: https://thebuildingcoder.typepad.com/blog/0902_whats_new_2011.html
parent_post: 0902_whats_new_2011.md
part_number: '24'
part_total: '52'
post_number: 0902
series: whats_new_in_revit_api
slug: whats_new_2011_create_and_display_revit-style_task_dialogs
source_file: 0902_whats_new_2011.htm
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
- rooms
- schedules
- selection
- sheets
- transactions
- vbnet
- views
- walls
- windows
title: What's New in Revit 2011 API - Create and display Revit-style Task Dialogs
word_count: 135
---

### Create and display Revit-style Task Dialogs

The API now offers the ability to create and display Revit-style Task Dialogs. It is intended to provide capabilities similar to System.Windows.Forms.MessageBox with a Revit look-and-feel such as:

![Task dialogue](img/whats_new_2011_1.png)

Construct an instance of the class:

- Autodesk.Revit.UI.TaskDialog

and use the members of that class to set the instructions, detailed text, icons, buttons and command links to be displayed in the task dialog. Then display the dialog using the Show() method. After the user responds to the dialog by clicking one of the buttons or links, the Show() method returns an identifier indicating the user's preferred choice.

There are also shortcut static Show() methods on this class providing a quick way to show simple task dialogs.
---

## Related Sections
- [What's New in Revit 2011 API - Changes to the Revit API namespaces](./0902-02_changes_to_the_revit_api_namespaces.md)
- [What's New in Revit 2011 API - Split of Revit API DLL](./0902-03_split_of_revit_api_dll.md)
- [What's New in Revit 2011 API - New classes for XYZ, UV, and ElementId](./0902-04_new_classes_for_xyz_uv_and_elementid.md)
- [What's New in Revit 2011 API - Replacement for Symbol and properties that access types](./0902-05_replacement_for_symbol_and_properties_that_access_.md)
- [What's New in Revit 2011 API - New transaction interfaces](./0902-06_new_transaction_interfaces.md)
- [What's New in Revit 2011 API - New ExternalCommand and ExternalApplication registration mechanism](./0902-07_new_externalcommand_and_externalapplication_regist.md)
- [What's New in Revit 2011 API - External Command and External Application registration utility](./0902-08_external_command_and_external_application_registra.md)
- [What's New in Revit 2011 API - Attributes for configuring ExternalCommand and ExternalApplication behavior](./0902-09_attributes_for_configuring_externalcommand_and_ext.md)
- [What's New in Revit 2011 API - New element iteration interfaces](./0902-10_new_element_iteration_interfaces.md)
- [What's New in Revit 2011 API - Replacement for View.Elements](./0902-11_replacement_for_viewelements.md)
- [What's New in Revit 2011 API - Replacement for 3 structural enums](./0902-12_replacement_for_3_structural_enums.md)
- [What's New in Revit 2011 API - Replacement for AnalyticalModel](./0902-13_replacement_for_analyticalmodel.md)
- [What's New in Revit 2011 API - Replacement for gbXMLParamElem](./0902-14_replacement_for_gbxmlparamelem.md)
- [What's New in Revit 2011 API - Revit exceptions](./0902-15_revit_exceptions.md)
- [What's New in Revit 2011 API - Removed deprecated events](./0902-16_removed_deprecated_events.md)
- [What's New in Revit 2011 API - Changes to VSTA](./0902-17_changes_to_vsta.md)
- [What's New in Revit 2011 API - Dynamic Model Update](./0902-18_dynamic_model_update.md)
- [What's New in Revit 2011 API - Elements changed event](./0902-19_elements_changed_event.md)
- [What's New in Revit 2011 API - Failure API](./0902-20_failure_api.md)
- [What's New in Revit 2011 API - Select element(s), point(s) on element(s), face(s) or edge(s)](./0902-21_select_elements_points_on_elements_faces_or_edges.md)
- [What's New in Revit 2011 API - Pick point on the view active workplane](./0902-22_pick_point_on_the_view_active_workplane.md)
- [What's New in Revit 2011 API - Additional options for Ribbon customization](./0902-23_additional_options_for_ribbon_customization.md)
- [What's New in Revit 2011 API - Idling event](./0902-25_idling_event.md)
- [What's New in Revit 2011 API - Sun and shadows settings element](./0902-26_sun_and_shadows_settings_element.md)
- [What's New in Revit 2011 API - MEP Electrical API](./0902-27_mep_electrical_api.md)
- [What's New in Revit 2011 API - Analysis Visualization Framework](./0902-28_analysis_visualization_framework.md)
- [What's New in Revit 2011 API - Family & massing API enhancements](./0902-29_family_massing_api_enhancements.md)
- [What's New in Revit 2011 API - View API changes](./0902-30_view_api_changes.md)
- [What's New in Revit 2011 API - Import, export and print changes](./0902-31_import_export_and_print_changes.md)
- [What's New in Revit 2011 API - Parameter API changes](./0902-32_parameter_api_changes.md)
- [What's New in Revit 2011 API - Alignment and ItemFactoryBase.NewAlignment() method](./0902-33_alignment_and_itemfactorybasenewalignment_method.md)
- [What's New in Revit 2011 API - DimensionType – access to the dimension style](./0902-34_dimensiontype_access_to_the_dimension_style.md)
- [What's New in Revit 2011 API - Create sloped wall on mass face](./0902-35_create_sloped_wall_on_mass_face.md)
- [What's New in Revit 2011 API - Face.GetBoundingBox() method](./0902-36_facegetboundingbox_method.md)
- [What's New in Revit 2011 API - NewFootPrintRoof() input](./0902-37_newfootprintroof_input.md)
- [What's New in Revit 2011 API - Floor slope and elevation](./0902-38_floor_slope_and_elevation.md)
- [What's New in Revit 2011 API - Converting between Model Curves and Detail/Symbolic Curves](./0902-39_converting_between_model_curves_and_detailsymbolic.md)
- [What's New in Revit 2011 API - CurveElement type](./0902-40_curveelement_type.md)
- [What's New in Revit 2011 API - Shared base classes for Room, Space, Area and their tags](./0902-41_shared_base_classes_for_room_space_area_and_their_.md)
- [What's New in Revit 2011 API - View-specific elements](./0902-42_view_specific_elements.md)
- [What's New in Revit 2011 API - Monitoring elements](./0902-43_monitoring_elements.md)
- [What's New in Revit 2011 API - Design Options](./0902-44_design_options.md)
- [What's New in Revit 2011 API - Family template path](./0902-45_family_template_path.md)
- [What's New in Revit 2011 API - Write comments to the Revit Journal File](./0902-46_write_comments_to_the_revit_journal_file.md)
- [What's New in Revit 2011 API - Window extents](./0902-47_window_extents.md)
- [What's New in Revit 2011 API - City.WeatherStation](./0902-48_cityweatherstation.md)
- [What's New in Revit 2011 API - Labels matching commonly used enumerated type values](./0902-49_labels_matching_commonly_used_enumerated_type_valu.md)
- [What's New in Revit 2011 API - Keyboard shortcut support for API buttons](./0902-50_keyboard_shortcut_support_for_api_buttons.md)
- [What's New in Revit 2011 API - MEP API](./0902-51_mep_api.md)
- [What's New in Revit 2011 API - Structural API](./0902-52_structural_api.md)
