---
author: Jeremy Tammik
optimization_date: '2025-12-11T11:24:11.673104'
original_url: https://thebuildingcoder.typepad.com/blog/1428_whats_new_2017.html
parent_post: 1428_whats_new_2017.md
part_number: '17'
part_total: '43'
post_number: '1428'
series: geometry
slug: whats_new_2017_mep_api_changes
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
title: API Changes - MEP API changes
word_count: 399
---

### MEP API changes
#### Duct API
The following duct creation methods have been deprecated and replaced by new methods. Many of the new methods offer additional parameters supporting the assignment of duct system type and reference level:
- \*\*Deprecated member → New or replacement member\*\*
- Document.NewDuct(XYZ, XYZ, DuctType) → Duct.Create(Document, ElementId ductSystemTypeId, ElementId ductTypeId, ElementId levelId, XYZ, XYZ)
- Document.NewDuct(XYZ, Connector, DuctType) → Duct.Create(Document, ElementId ductTypeId, ElementId levelId, Connector, XYZ)
- Document.NewDuct(Connector, Connector, DuctType) → Duct.Create(Document, ElementId ductTypeId, ElementId levelId, Connector, Connector)
- DuctFittingAndAccessoryConnectorData.Coordination → DuctFittingAndAccessoryConnectorData.GetCoordination()
#### Pipe API
The following members have been deprecated and replaced:
- \*\*Deprecated member → New or replacement member\*\*
- PipeType.Class → PipeSegment.ScheduleTypeId
- PipeFittingAndAccessoryConnectorData.Coordination → PipeFittingAndAccessoryConnectorData.GetCoordination()
The deprecated property returns one pipe schedule type element on the pipe type. In case that the pipe type contains multiple pipe segments and schedule types in its routing preference definition, only the first pipe schedule type is returned in the deprecated property. Instead, the correct usage is to use the new property Pipe.PipeSegment, which provides the correct pipe schedule type and other segment properties, just as Revit property palette shows.
Additionally, the following new methods are available related to PipeScheduleType:
- PipeScheduleType.Create(Document, String) – Creates a new pipe schedule type with the given name.
- PipeScheduleType.GetPipeScheduleId(Document, String) – Returns an existing pipe schedule type with the given name.
#### MEP System API
The following properties have been deprecated and replaced by methods:
- \*\*Deprecated member → New or replacement member\*\*
- MechanicalSystem.Flow → MechanicalSystem.GetFlow()
- MechanicalSystem.StaticPressure → MechancalSystem.GetStaticPressure()
- PipingSystem.FixtureUnits → PipingSystem.GetFixtureUnits()
- PipingSystem.Flow → PipingSystem.GetFlow()
- PipingSystem.StaticPressure → PipingSystem.GetStaticPressure()
A new method is also added:
- PipingSystem.GetVolume()
Internally, these MEP system values are now calculated asynchronously on a non-blocking evaluation framework. In order to handle asynchronous calculation results, the caller needs to define callback methods to react on background calculation results (e.g., to refresh the user interface). API developers cannot define callbacks but will still get the correct value. If no callback methods are defined (e.g., in third party applications), the calculation is automatically switched to synchronous calculation.
These values have been exposed via built-in parameters in the past. They are still supported. For example, PipingSystem.get_ParameterValue(BuiltInParameter.RBS_PIPE_FLOW_PARAM) will get the correct flow value synchronously, assuming no callback is detected. The caveat is that, due to the internal support of asynchronous calculation, these parameters no longer support dynamic model update.
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
- [API Changes - View API additions](./1428-29_view_api_additions.md)
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
