---
author: Jeremy Tammik
optimization_date: '2025-12-11T11:24:11.687412'
original_url: https://thebuildingcoder.typepad.com/blog/1428_whats_new_2017.html
parent_post: 1428_whats_new_2017.md
part_number: '29'
part_total: '43'
post_number: '1428'
series: geometry
slug: whats_new_2017_view_api_additions
source_file: 1428_whats_new_2017.md
tags:
- elements
- family
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
- views
- walls
- windows
title: API Changes - View API additions
word_count: 462
---

### View API additions
#### TemporaryViewModes
The new class:
- TemporaryViewModes
carries data related to the state and properties of available temporary view modes. Access to an instance of this class is via the property:
- View.TemporaryViewModes
The class has the following methods and properties:
- TemporaryViewModes.DeactivateAllModes() – Deactivates all temporary modes that are currently active.
- TemporaryViewModes.DeactivateMode() – Deactivates the given temporary mode.
- TemporaryViewModes.GetCaption() – A text caption to use for the given mode.
- TemporaryViewModes.IsModeActive() – Tests whether a given mode is currently active or not.
- TemporaryViewModes.IsModeAvailable() – Tests whether a temporary view mode is currently available in the associated view.
- TemporaryViewModes.IsModeEnabled() – Tests whether a temporary view mode is currently enabled in the associated view.
- TemporaryViewModes.IsValidState() – Tests whether the given state is valid for the associated view and the context the view is currently in.
- TemporaryViewModes.PreviewFamilyVisibility – The current state of the PreviewFamilyVisibility mode in the associated view.
- TemporaryViewModes.RevealConstraints – The current state of the RevealConstraints mode in the associated view.
- TemporaryViewModes.RevealHiddenElements – The current state of the RevealHiddenElements mode in the associated view.
- TemporaryViewModes.WorskaringDisplay – The current state of the WorksharingDisplay mode in the associated view.
#### Convert dependent view to independent
The new function:
- View.ConvertToIndependent()
converts a dependent view to be independent.
#### Plan view underlay
The new methods:
- ViewPlan.GetUnderlayBaseLevel()
- ViewPlan.GetUnderlayTopLevel()
- ViewPlan.SetUnderlayBaseLevel()
- ViewPlan.SetUnderlayRange()
- ViewPlan.SetUnderlayOrientation()
- ViewPlan.SetUnderlayOrientation()
provide access to the underlay levels and settings for plan views.
#### Assembly views creation
The creation of assembly views and schedules has been improved to allow the creation of an assembly view or schedule with template information. The new overloads for methods:
- AssemblyViewUtils.Create3DOrthographic()
- AssemblyViewUtils.CreateDetailSection()
- AssemblyViewUtils.CreateSingleCategorySchedule()
- AssemblyViewUtils.CreatePartList()
- AssemblyViewUtils.CreateMaterialTakeoff()
offer two new arguments:
- viewTemplateId – the id of the template from which the view is to be created.
- isAssigned – if true, the template passed in viewTemplateId will be assigned to the view; if false, the template information is only applied to the view.
#### Depth Cueing
The new class:
- ViewDisplayDepthCueing
allows users to control the display of distant objects in section and elevation views. When depth cueing is active, objects blend into the background colour (fade) with increasing distance from the viewer.
The class contains the following methods and properties:
- ViewDisplayDepthCueing.EnableDepthCueing
- ViewDisplayDepthCueing.StartPercentage – Indicates where depth cueing begins. A value of 0 indicates that depth cueing begins at the front clip plane of the view.
- ViewDisplayDepthCueing.EndPercentage – Indicates where depth cueing ends. Objects further than the end plane will fade the same amount as objects at the end plane. A value of 100 indicates the far clip plane.
- ViewDisplayDepthCueing.FadeTo – Indicates the maximum amount to fade objects via depth cueing. A value of 100 indicates complete invisibility.
- ViewDisplayDepthCueing.SetStartEndPercentages()
The new methods:
- DBView.GetDepthCueing()
- DBView.SetDepthCueing()
allow the user to get and set the depth cueing settings for the view.
---

## Related Sections
- [API Changes - What's New in the Revit 2017 API](./1428-01_whats_new_in_the_revit_2017_api.md)
- [API Changes - Major Revit API Changes and Renovations](./1428-02_major_revit_api_changes_and_renovations.md)
- [API Changes - API Changes](./1428-03_api_changes.md)
- [API Changes - .NET 4.6](./1428-04_net_46.md)
- [API Changes - Visual C++ Redistributable for Visual Studio 2015](./1428-05_visual_c_redistributable_for_visual_studio_2015.md)
- [API Changes - Automatic transaction mode obsolete](./1428-06_automatic_transaction_mode_obsolete.md)
- [API Changes - Code signing of Revit Addins](./1428-07_code_signing_of_revit_addins.md)
- [API Changes - Background processes can load DB applications](./1428-08_background_processes_can_load_db_applications.md)
- [API Changes - Application API changes](./1428-09_application_api_changes.md)
- [API Changes - Family API changes](./1428-10_family_api_changes.md)
- [API Changes - View API changes](./1428-11_view_api_changes.md)
- [API Changes - Text API changes](./1428-12_text_api_changes.md)
- [API Changes - Event API changes](./1428-13_event_api_changes.md)
- [API Changes - Alignment API changes](./1428-14_alignment_api_changes.md)
- [API Changes - Geometry API changes](./1428-15_geometry_api_changes.md)
- [API Changes - Structure API changes](./1428-16_structure_api_changes.md)
- [API Changes - MEP API changes](./1428-17_mep_api_changes.md)
- [API Changes - EnergyDataSettings API changes](./1428-18_energydatasettings_api_changes.md)
- [API Changes - Rendering API changes](./1428-19_rendering_api_changes.md)
- [API Changes - Plane API changes](./1428-20_plane_api_changes.md)
- [API Changes - DirectShape API changes](./1428-21_directshape_api_changes.md)
- [API Changes - Point Cloud API changes](./1428-22_point_cloud_api_changes.md)
- [API Changes - Schedule API changes](./1428-23_schedule_api_changes.md)
- [API Changes - UI API change](./1428-24_ui_api_change.md)
- [API Changes - Obsolete API removal](./1428-25_obsolete_api_removal.md)
- [API Changes - API Additions](./1428-26_api_additions.md)
- [API Changes - Application API additions](./1428-27_application_api_additions.md)
- [API Changes - Family API additions](./1428-28_family_api_additions.md)
- [API Changes - Text API additions](./1428-30_text_api_additions.md)
- [API Changes - Geometry API additions](./1428-31_geometry_api_additions.md)
- [API Changes - Parameter API additions](./1428-32_parameter_api_additions.md)
- [API Changes - CurveElement API additions](./1428-33_curveelement_api_additions.md)
- [API Changes - Railing API additions](./1428-34_railing_api_additions.md)
- [API Changes - Schedule API additions](./1428-35_schedule_api_additions.md)
- [API Changes - Tag API additions](./1428-36_tag_api_additions.md)
- [API Changes - UI API additions](./1428-37_ui_api_additions.md)
- [API Changes - Structure API additions](./1428-38_structure_api_additions.md)
- [API Changes - Fabrication API additions](./1428-39_fabrication_api_additions.md)
- [API Changes - Electrical API additions](./1428-40_electrical_api_additions.md)
- [API Changes - Other MEP API additions](./1428-41_other_mep_api_additions.md)
- [API Changes - Revit Link API additions](./1428-42_revit_link_api_additions.md)
- [API Changes - Category API additions](./1428-43_category_api_additions.md)
