---
author: Jeremy Tammik
optimization_date: '2025-12-11T11:24:26.211967'
original_url: https://thebuildingcoder.typepad.com/blog/0904_whats_new_2013.html
parent_post: 0904_whats_new_2013.md
part_number: '01'
part_total: '90'
post_number: 0904
series: whats_new_in_revit_api
slug: whats_new_2013_elementarray_and_elementset_are_now_obsolete
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
title: What's New in Revit 2013 API - ElementArray and ElementSet are now obsolete
word_count: 825
---

﻿

# What's New in the Revit 2013 API

This is the fourth and last instalment of a series publishing the information provided in the 'What's New' sections of the past few Revit API releases help file RevitAPI.chm.

The first instalment covering
[What's New in the Revit 2010 API](http://thebuildingcoder.typepad.com/blog/2013/02/whats-new-in-the-revit-2010-api.html) explains
my motivation for this and provides an overview of the other releases.

We now move on to the Revit 2013 API, looking at:

- [Major changes and renovations](#2)
- [Major enhancements](#3)
- [Small enhancements & API interface changes](#4)

First, however, another update on my vacation.

#### Turning North Again

I am still on holiday in Italy, so please do not expect any immediate responses to comments for a while.

![Almond tree blossoming](file:///j/photo/jeremy/2013/2013-02-28_taranto/p1000259_jeremy_under_almond_tree.jpg)

After withstanding a snowstorm further north, I did indeed find some warmth further south.

Almond trees are in bloom, and the grass is as green as it will ever get.

![Alberobello trulli garden](file:///j/photo/jeremy/2013/2013-02-28_taranto/p1000290_alberobello_trulli_garden.jpg)

I explored the sweet Apulian local train system.

![Galatina train station](file:///j/photo/jeremy/2013/2013-02-28_taranto/p1000299_galatina_train_station.jpg)

We had some cold and rainy days in the south also.

![Portoselvaggio Torre Alto under stormy clouds](file:///j/photo/jeremy/2013/2013-02-28_taranto/p1000322_portoselvaggio_torre_alto_cloud.jpg)

Turning back northwards towards Napoli under splendid blue skies, I passed this house beside a cafe I visited changing trains in Taranto.

![Taranto house with washing](file:///j/photo/jeremy/2013/2013-02-28_taranto/p1000363_taranto_house_with_washing.jpg)

I walked the wonderful
[Sentiero degli Dei](http://it.wikipedia.org/wiki/Sentiero_degli_Dei) that
I remember so well from my
[visit in 2009](http://thebuildingcoder.typepad.com/blog/2009/03/more-questions.html).

![Sentiero degli Dei](file:///j/photo/jeremy/2013/2013-03-02_napoli/p1000425_sentiero_degli_dei.jpg)

This time around, I started at the bottom, in Praiano, walked up to the Convento San Domenico, spent the night there, and walked on through Nocelle back down to Positano next morning.

![Positano](file:///j/photo/jeremy/2013/2013-03-02_napoli/p1000431_positano.jpg)

We had a terrible struggle to get there and back out again due to a bus strike.
We finally reached Napoli, though, and first of all had a pizza.

![Pizzeria Costa in Napoli](file:///j/photo/jeremy/2013/2013-03-02_napoli/p1000444_pizzeria_costa_napoli.jpg)

The clouds returned and covered the Vesuvio.

![Vesuvio in clouds](file:///j/photo/jeremy/2013/2013-03-02_napoli/p1000452_vesuvio_in_clouds.jpg)

Spring is eagerly awaited here also, even inspiring the pasticceria.

![Spring cakes](file:///j/photo/jeremy/2013/2013-03-02_napoli/p1000455_spring_cakes.jpg)

Now for the last instalment of the previous Revit API releases' news, bringing us up to date with the current day.

# Major Changes and Renovations to the Revit 2013 API

## .NET 4.0 for compilation

All Revit components including RevitAPI.dll and RevitAPIUI.dll are now built with the .NET 4.0 compiler. Thus, add-in projects which reference Revit DLLs must be set as .NET 4.0 Framework projects as well.

## New macro environment

VSTA has been replaced by SharpDevelop. This provides identical capabilities for macro creation, editing, and debugging as well as for deployment (application-level and document-level macros) and a simple upgrade procedure for existing macros.

SharpDevelop also provides expanded IDE capabilities, .NET 4.0, extended refactoring tools, code analysis tools, a regular expression toolkit, and macro IDE add-in support.

## Revit.ini registration disabled

Add-ins will no longer be read from Revit.ini. Revit will look only to .addin files in the machine-wide and user-specific locations when deciding what add-ins to load.

## Document.Element properties replaced

The indexed properties:

- Document.Element[ElementId]- Document.Element[String]

have been replaced by methods:

- Document.GetElement(ElementId)- Document.GetElement(String)

The old indexed properties have been marked obsolete.

The behavior for the new methods is identical to what they replaced.

## Replacements for use of old-style collections

### ElementArray and ElementSet are now obsolete

With a few exceptions, API methods using ElementArray and ElementSet have been replaced with equivalents using .NET generic collections of ElementIds (ICollection<ElementId>).

Some APIs have been obsoleted with no replacement; these methods are batch element creation methods which dated to the time when regeneration was automatic after every database change. Now that regeneration is manual, having separate batch creation methods provides no extra benefit as multiple element creation calls can be chained together with no delay caused by regeneration.
Thus these methods are obsolete:

- ItemFactoryBase.NewTextNotes()- Creation.Document.NewRooms(List<RoomCreationData>)- Creation.Document.NewWalls() (all overloads)

When replacing calls to these methods, you should use the corresponding creation method which creates a single element in a loop, while delaying calls to Regenerate() until the loop has concluded. Because Regenerate() is delayed, you should not make inquiries to the newly created elements' geometry as a part of the loop.
---

## Related Sections
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
- [What's New in Revit 2013 API - FBX Export](./0904-68_fbx_export.md)
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
