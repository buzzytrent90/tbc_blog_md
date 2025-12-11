---
author: Jeremy Tammik
optimization_date: '2025-12-11T11:24:11.691659'
original_url: https://thebuildingcoder.typepad.com/blog/1428_whats_new_2017.html
parent_post: 1428_whats_new_2017.md
part_number: '32'
part_total: '43'
post_number: '1428'
series: geometry
slug: whats_new_2017_parameter_api_additions
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
title: API Changes - Parameter API additions
word_count: 525
---

### Parameter API additions
#### Global Parameters
Global Parameters support controlling geometry constraints through special parameters defined in a project document. Global Parameters can be used for both labeling and reporting to/from dimensions, as well as setting values of instance parameters.
The new class
- GlobalParametersManager
provides the main access point to managing global parameters in project document. It offers the following members:
- AreGlobalParametersAllowed() – tests whether global parameters are allowed in a document
- GetAllGlobalParameters() – returns all global parameters in a document
- FindByName() – find a global parameter by its name
- IsUniqueName() – test uniqueness of the name of a prospective global parameters
- IsValidGlobalParameter() – test if an Id is of a valid global parameter element
The new class:
- GlobalParameter
contains methods to control and manipulate a single global parameter. It's most important members include:
- [static] Create() – Creates a new Global Parameter in the given document.
- GetAffectedElements() – Returns all elements of which properties are driven by this global parameter.
- GetAffectedGlobalParameters() – Returns all other global parameters which refer to this global parameter in their formulas.
- GetLabeledDimensions() – Returns all dimension elements that are currently labeled by this global parameter.
- CanLabelDimension() – Tests whether a dimension can be labeled by the global parameter.
- LabelDimension() – Labels a dimension with this global parameter.
- UnlabelDimension() – Un-labels a dimension that is currently labeled by this global parameter.
- GetLabelName() – Returns the name of this parameter's label, which is used to label dimension elements.
- SetDrivingDimension() – Set a dimension to drive the value of this parameter.
- IsValidFormula() – Tests that the given expression is a valid as formula for this parameter.
- GetFormula() – Returns the parameter's expression in form of a string.
- SetFormula() – Sets a formula expression for this parameter.
- GetValue() – Obtains the current value of the global parameter.
- SetValue() – Sets a new value of the global parameter.
- HasValidTypeForReporting() – Tests that the global parameter has data of a type that supports reporting.
- [static] IsValidDataType() – Tests whether the input Data Type is valid as a type of a global parameter.
- IsDrivenByDimension – Indicates whether this parameter is driven by a dimension or not.
- IsDrivenByFormula – Indicates whether this parameter is driven by a formula or not.
- IsReporting – Indicates whether this is a reporting global parameter or not.
The new class:
- ParameterValue
contains a value of a corresponding global parameter. It is a base class for derived concrete classes, one per each type of a parameter value:
- IntegerParameterValue
- DoubleParameterValue
- StringParameterValue
- ElementIdParameterValue
- NullParameterValue
All the derived classes have only one property:
- Value – gets or sets the value as the corresponding type.
New methods added to the Parameter class:
- CanBeAssociatedWithGlobalParameter() – Tests whether a parameter can be associated with the given global parameter.
- CanBeAssociatedWithGlobalParameters() – Tests whether a parameter can be associated with any global parameter.
- AssociateWithGlobalParameter() – Associates a parameter with a global parameter in the same document.
- DissociateFromGlobalParameter() – Dissociates a parameter from a global parameter.
- GetAssociatedGlobalParameter() – Returns a global parameter, if any, currently associated with a parameter.
#### InternalDefinition.Id
The new property:
- InternalDefinition.Id
returns the id for the associated parameter. This is the id of the associated ParameterElement if the parameter is not built-in.
#### Multiline Text parameter support
The new enumerated value:
- ParameterType.MultilineText
was added for creation and use of multi-line text parameters.
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
- [API Changes - View API additions](./1428-29_view_api_additions.md)
- [API Changes - Text API additions](./1428-30_text_api_additions.md)
- [API Changes - Geometry API additions](./1428-31_geometry_api_additions.md)
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
