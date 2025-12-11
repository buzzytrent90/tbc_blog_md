---
author: Jeremy Tammik
optimization_date: '2025-12-11T11:24:06.645380'
original_url: https://thebuildingcoder.typepad.com/blog/0902_whats_new_2011.html
parent_post: 0902_whats_new_2011.md
part_number: '10'
part_total: '52'
post_number: 0902
series: whats_new_in_revit_api
slug: whats_new_2011_new_element_iteration_interfaces
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
title: What's New in Revit 2011 API - New element iteration interfaces
word_count: 560
---

### New element iteration interfaces

New interfaces have been added to the API to permit iteration of elements and element ids. These new interfaces have been designed to provide a more flexible and usable interface for a variety of applications, while being completely implemented in native Revit code to provide the best possible performance.

Existing element iteration routines have been replaced, including:

- Document.Elements (all versions of this property)- ElementIterator- Filter and all subclasses of it- Application.Create.Filter- Classes and properties related to the ParameterFilter:
          - CriteriaFilterType- Document.FilterTypeSupported- FilterCriterion- ParameterStorage

The new element iteration interfaces offer several enhanced capabilities from their predecessors:

- The ability to iterate and filter elements from a document, or only elements from an arbitrary list of element ids, or elements visible in a view (replacing View.Elements)- The ability to clearly identify filters which are designed for best performance ("Quick Filters"), which do not expand the element in memory when evaluating whether the element passes the filter- The ability to use chained shortcuts which automatically apply commonly used filters, e.g.

```csharp
  FilteredElementCollector collector
    = new FilteredElementCollector( document );

  // Finds all walls in a certain design option

  ICollection<ElementId> walls = collector
    .OfClass( typeof( Wall ) )
    .ContainedInDesignOption( myDesignOptionId )
    .ToElementIds();
```

- The ability to logically group more than two filters.- The ability to match derived types automatically when using the type filter and type filter shortcut.- The ability to iterate elements from all design options or from any specific design option.- The ability to use foreach on the collector element, and to use the class with LINQ queries. This is due to the connection between the collector class and System.Collections.Generic.IEnumerable. Note that because the ElementFilters and the shortcut methods offered by this class process elements in native code before their managed wrappers are generated, better performance will be obtained by using as many native filters as possible on the collector before attempting to process the results using LINQ queries.

```csharp
  FilteredElementCollector collector
    = new FilteredElementCollector( doc );

  // First apply a built-in filter to minimize
  // the number of elements processed by LINQ

  collector.WherePasses( new ElementCategoryFilter(
    BuiltInCategory.OST\_Levels ) );

  // LINQ query to find level with name == "Level 1"

  var levelElements = from element in collector
                      where element.Name == "Level 1"
                      select element;

  Level level1 = levelElements.Cast<Level>()
    .ElementAt<Level>( 0 );
```

Detailed information on the new element iteration interfaces can be found in the document "Element iteration APIs" in the SDK.

Note that there are a few behavioral changes in the new iteration mechanism when compared to the old:

- View templates were previously returned as Element. They are now returned as the correct View subclass (ViewPlan, View3d, etc). You can identify these using the new View.IsTemplate property.- Previous element iteration methods only returned elements of the main model, the active design option, and primary design options of each option set. Elements of design options which were not active and not primary were excluded. In the new iteration API, by default, design option membership is no longer considered when iterating elements. In order to find the same elements which were iterated in the 2010 API, you should use the following filters combined using 'OR':
    - An ElementDesignOptionFilter with id = InvalidElementId, to match all elements not associated to a design option.- An ElementDesignOptionFilter with id = DesignOption.GetActiveDesignOptionId(), to match elements in the active design option.- A PrimaryDesignOptionMemberFilter, to match all elements in primary design options.
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
- [What's New in Revit 2011 API - Create and display Revit-style Task Dialogs](./0902-24_create_and_display_revit_style_task_dialogs.md)
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
