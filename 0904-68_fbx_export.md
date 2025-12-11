---
author: Jeremy Tammik
optimization_date: '2025-12-11T11:24:26.288393'
original_url: https://thebuildingcoder.typepad.com/blog/0904_whats_new_2013.html
parent_post: 0904_whats_new_2013.md
part_number: '68'
part_total: '90'
post_number: 0904
series: whats_new_in_revit_api
slug: whats_new_2013_fbx_export
source_file: 0904_whats_new_2013.htm
tags:
- doors
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
- transactions
- views
- walls
- windows
title: What's New in Revit 2013 API - FBX Export
word_count: 687
---

### FBX Export

Three new options have been added:

- FBXExportOptions.UseLevelsOfDetail – true to use levels of detail, false otherwise- FBXExportOptions.LevelsOfDetailValue – the value of the levels of detail- FBXExportOptions.WithoutBoundaryEdges – true to export without boundary edges, false otherwise

## DividedPath

A DividedPath is an element supported in the massing environment that consists of a set of points distributed along a connected set of curves and edges. The points can be the result of a uniform distribution along the curves. The type of the distribution is determined by a selected 'layout'. The distance between the layout points depends on the curves, the layout, and layout specific settings. In addition, points can also be the result of intersecting the curves with other elements.

The class

- DividedPath

exposes the interface for this element type. It permits creation of a new DividedPath from input curves and edges, as well as optionally, intersecting elements. It allows control over the layout type and parameters, as well as other miscellaneous settings.

## Analysis Visualization Framework

The following additions have been made to support deformed shapes in analysis visualization display:

- AnalysisDisplayStyleDeformedShapeTextLabelType class- AnalysisDisplayDeformedShapeSettings class

A new override to CreateAnalysisDisplayStyle accepts an input argument of AnalysisDisplayDeformedShapeSettings

## Revit Link creation

The API now contains two creation methods for Revit links.

- RevitLinkType.Create(Document, ModelPath, RevitLinkOptions) will create a new Revit link type and load the associated linked file into the document. This returns a RevitLinkLoadResult, which stores the ElementId of the newly-created RevitLinkType and contains any errors which occurred when trying to load the linked file (The RevitLinkLoadResultType enumeration contains the full list.)- RevitLinkInstance.Create(Document, ElementId) will create a new instance of an already-loaded RevitLinkType.

## FilledRegion

The class

- FilledRegion

has been extended to offer the ability to create Filled regions, to get the boundaries of the region, and to apply a linestyle to all boundary segments.

The class

- FilledRegionType

has been added providing access to the visible properties of a filled region type.

## Light API

The classes

- LightType- LightFamily

have been added to offer the ability to get and set photometric data and all other lighting parameters from both family instances in project documents and all family types in family documents. The LightType and LightFamily classes expose the same data except for light shape and distribution types which are only available from the LightFamily class.

Some examples of lighting parameters exposed are;

- Initial color- Initial intensity- Loss factor- Color filter- Dimming color

## Light Group API

The classes

- LightGroupManager- LightGroup

have been added to offer the ability to manage light groups to allow control over which lights are used when rendering the scene.

The LightGroupManager class gives the user the ability to

- Create a new light group- Delete an existing light group- Turn on or off all the lights in a light group- Turn on or off individual lights- Set the dimmer value for individual lights

The LightGroup class gives the user the ability to

- Get the name of a light group- Rename light group- Add a light to a light group- Remove a light from a light group

Light groups are used in rendering options to turn lights on or off when rendering.

## ReferenceIntersector

The new class ReferenceIntersector allows ray-cast selection of elements, given a point and direction, similar to FindReferencesWithContextByDirection(), but with support for filtering the output based on element or reference type.

The enum FindReferenceTarget is used with ReferenceIntersector to filter selection of elements, meshes, edges, curves, and faces.

Key members of ReferenceIntersector

- ReferenceIntersector(ElementId targetElementId, FindReferenceTarget targetType, View3d view3d) – constructor specifying a single element to search for in the intersection test.- ReferenceIntersector(ElementIdSet targetElementIds, FindReferenceTarget targetType, View3d view3d) – constructor specifying a set of ElementIds to search for in the intersection test.- ReferenceIntersector(ElementFilter filter, FindReferenceTarget targetType, View3d view3d) – constructor specifying an ElementFilter for the intersection test.- Find(XYZ origin, XYZ direction) – Finds all references intersecting the origin-direction ray given the selection criteria set up in the ReferenceIntersector constructor- FindNearest (XYZ origin, XYZ direction) – Finds the reference closest to the origin in the origin-direction ray given the selection criteria set up in the ReferenceIntersector constructor

# Small Enhancements & API Interface Changes

## Elements & filtering
---

## Related Sections
- [What's New in Revit 2013 API - ElementArray and ElementSet are now obsolete](./0904-01_elementarray_and_elementset_are_now_obsolete.md)
- [What's New in Revit 2013 API - Definitions and DefinitionGroups](./0904-02_definitions_and_definitiongroups.md)
- [What's New in Revit 2013 API - GeometryElement.Objects](./0904-03_geometryelementobjects.md)
- [What's New in Revit 2013 API - Idling event frequency](./0904-04_idling_event_frequency.md)
- [What's New in Revit 2013 API - Idling event with no active document](./0904-05_idling_event_with_no_active_document.md)
- [What's New in Revit 2013 API - External Events framework](./0904-06_external_events_framework.md)
- [What's New in Revit 2013 API - Calling OpenAndActivateDocument during events](./0904-07_calling_openandactivatedocument_during_events.md)
- [What's New in Revit 2013 API - Worksharing properties](./0904-08_worksharing_properties.md)
- [What's New in Revit 2013 API - New overloads for Application.OpenDocumentFile() and UIApplication.OpenAndActivateDocument()](./0904-09_new_overloads_for_applicationopendocumentfile_and_.md)
- [What's New in Revit 2013 API - BasicFileInfo](./0904-10_basicfileinfo.md)
- [What's New in Revit 2013 API - Free()](./0904-11_free.md)
- [What's New in Revit 2013 API - GetOffset()](./0904-12_getoffset.md)
- [What's New in Revit 2013 API - CreatePointSetIterator()](./0904-13_createpointsetiterator.md)
- [What's New in Revit 2013 API - Rebar.GetCenterlineCurves()](./0904-14_rebargetcenterlinecurves.md)
- [What's New in Revit 2013 API - Rebar member changes](./0904-15_rebar_member_changes.md)
- [What's New in Revit 2013 API - AreaReinforcement and PathReinforcement](./0904-16_areareinforcement_and_pathreinforcement.md)
- [What's New in Revit 2013 API - ConnectorProfileType, PartType](./0904-17_connectorprofiletype_parttype.md)
- [What's New in Revit 2013 API - ConnectorElement](./0904-18_connectorelement.md)
- [What's New in Revit 2013 API - Classes removed](./0904-19_classes_removed.md)
- [What's New in Revit 2013 API - Methods and properties removed](./0904-20_methods_and_properties_removed.md)
- [What's New in Revit 2013 API - Stairs and stairs components](./0904-21_stairs_and_stairs_components.md)
- [What's New in Revit 2013 API - Railings and railing components](./0904-22_railings_and_railing_components.md)
- [What's New in Revit 2013 API - Stairs annotations](./0904-23_stairs_annotations.md)
- [What's New in Revit 2013 API - View Creation](./0904-24_view_creation.md)
- [What's New in Revit 2013 API - DisplayStyle](./0904-25_displaystyle.md)
- [What's New in Revit 2013 API - ViewDetailLevel](./0904-26_viewdetaillevel.md)
- [What's New in Revit 2013 API - ViewRange](./0904-27_viewrange.md)
- [What's New in Revit 2013 API - 3D View Locking](./0904-28_3d_view_locking.md)
- [What's New in Revit 2013 API - View.Duplicate](./0904-29_viewduplicate.md)
- [What's New in Revit 2013 API - UIView class and UIDocument.GetOpenUIViews()](./0904-30_uiview_class_and_uidocumentgetopenuiviews.md)
- [What's New in Revit 2013 API - Pan and Zoom](./0904-31_pan_and_zoom.md)
- [What's New in Revit 2013 API - PlanViewDirection](./0904-32_planviewdirection.md)
- [What's New in Revit 2013 API - ViewFamilyType class and ViewFamily enum](./0904-33_viewfamilytype_class_and_viewfamily_enum.md)
- [What's New in Revit 2013 API - Temporary view modes](./0904-34_temporary_view_modes.md)
- [What's New in Revit 2013 API - Schedule API](./0904-35_schedule_api.md)
- [What's New in Revit 2013 API - Running API commands in Schedule Views](./0904-36_running_api_commands_in_schedule_views.md)
- [What's New in Revit 2013 API - Schedule export](./0904-37_schedule_export.md)
- [What's New in Revit 2013 API - Linked element parents for parts](./0904-38_linked_element_parents_for_parts.md)
- [What's New in Revit 2013 API - Merged parts](./0904-39_merged_parts.md)
- [What's New in Revit 2013 API - Excluded parts](./0904-40_excluded_parts.md)
- [What's New in Revit 2013 API - Part category](./0904-41_part_category.md)
- [What's New in Revit 2013 API - Part division](./0904-42_part_division.md)
- [What's New in Revit 2013 API - Assembly views](./0904-43_assembly_views.md)
- [What's New in Revit 2013 API - Assembly instance transform and location](./0904-44_assembly_instance_transform_and_location.md)
- [What's New in Revit 2013 API - New Rebar members](./0904-45_new_rebar_members.md)
- [What's New in Revit 2013 API - ReinforcementSettings](./0904-46_reinforcementsettings.md)
- [What's New in Revit 2013 API - AreaReinforcement/PathReinforcement](./0904-47_areareinforcementpathreinforcement.md)
- [What's New in Revit 2013 API - Analytical model](./0904-48_analytical_model.md)
- [What's New in Revit 2013 API - Routing Preferences](./0904-49_routing_preferences.md)
- [What's New in Revit 2013 API - MEP Sections](./0904-50_mep_sections.md)
- [What's New in Revit 2013 API - FluidType and FluidTemperature](./0904-51_fluidtype_and_fluidtemperature.md)
- [What's New in Revit 2013 API - Panel schedule – spare values](./0904-52_panel_schedule_spare_values.md)
- [What's New in Revit 2013 API - LabelUtils](./0904-53_labelutils.md)
- [What's New in Revit 2013 API - Thermal properties](./0904-54_thermal_properties.md)
- [What's New in Revit 2013 API - Structural properties](./0904-55_structural_properties.md)
- [What's New in Revit 2013 API - Material assets](./0904-56_material_assets.md)
- [What's New in Revit 2013 API - GBXML Export](./0904-57_gbxml_export.md)
- [What's New in Revit 2013 API - Contextual help support](./0904-58_contextual_help_support.md)
- [What's New in Revit 2013 API - Support for Keyboard Shortcuts and Quick Access Toolbar](./0904-59_support_for_keyboard_shortcuts_and_quick_access_to.md)
- [What's New in Revit 2013 API - Replace implementation of commands](./0904-60_replace_implementation_of_commands.md)
- [What's New in Revit 2013 API - Preview control](./0904-61_preview_control.md)
- [What's New in Revit 2013 API - Options dialog customization](./0904-62_options_dialog_customization.md)
- [What's New in Revit 2013 API - Drag & Drop support](./0904-63_drag_drop_support.md)
- [What's New in Revit 2013 API - DGN import](./0904-64_dgn_import.md)
- [What's New in Revit 2013 API - DXF import](./0904-65_dxf_import.md)
- [What's New in Revit 2013 API - DWG and DXF export settings changes](./0904-66_dwg_and_dxf_export_settings_changes.md)
- [What's New in Revit 2013 API - DGN export](./0904-67_dgn_export.md)
- [What's New in Revit 2013 API - Element.PhaseCreated and Element.PhaseDemolished](./0904-69_elementphasecreated_and_elementphasedemolished.md)
- [What's New in Revit 2013 API - Phase filter](./0904-70_phase_filter.md)
- [What's New in Revit 2013 API - SelectionFilterElement](./0904-71_selectionfilterelement.md)
- [What's New in Revit 2013 API - Ceiling, Floor and CeilingAndFloor](./0904-72_ceiling_floor_and_ceilingandfloor.md)
- [What's New in Revit 2013 API - Split volumes](./0904-73_split_volumes.md)
- [What's New in Revit 2013 API - Solid tessellation](./0904-74_solid_tessellation.md)
- [What's New in Revit 2013 API - Face.Triangulate()](./0904-75_facetriangulate.md)
- [What's New in Revit 2013 API - Ellipse – axes](./0904-76_ellipse_axes.md)
- [What's New in Revit 2013 API - GeometryElement.GetBoundingBox()](./0904-77_geometryelementgetboundingbox.md)
- [What's New in Revit 2013 API - ItemFactoryBase.NewAlignment()](./0904-78_itemfactorybasenewalignment.md)
- [What's New in Revit 2013 API - CylindricalHelix curve type](./0904-79_cylindricalhelix_curve_type.md)
- [What's New in Revit 2013 API - CurveLoop as IEnumerable<Curve>](./0904-80_curveloop_as_ienumerablecurve.md)
- [What's New in Revit 2013 API - CurveElement – center point reference](./0904-81_curveelement_center_point_reference.md)
- [What's New in Revit 2013 API - NewSketchPlane(Document, Reference)](./0904-82_newsketchplanedocument_reference.md)
- [What's New in Revit 2013 API - MultiSegmentGrid class](./0904-83_multisegmentgrid_class.md)
- [What's New in Revit 2013 API - Grid geometry](./0904-84_grid_geometry.md)
- [What's New in Revit 2013 API - Diameter dimensions](./0904-85_diameter_dimensions.md)
- [What's New in Revit 2013 API - DimensionSegment overrides](./0904-86_dimensionsegment_overrides.md)
- [What's New in Revit 2013 API - Detail element draw order](./0904-87_detail_element_draw_order.md)
- [What's New in Revit 2013 API - SpatialElementCalculationPoint](./0904-88_spatialelementcalculationpoint.md)
- [What's New in Revit 2013 API - RPC content assets](./0904-89_rpc_content_assets.md)
- [What's New in Revit 2013 API - Family.PlacementType](./0904-90_familyplacementtype.md)
