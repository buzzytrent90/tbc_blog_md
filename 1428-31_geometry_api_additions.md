---
author: Jeremy Tammik
optimization_date: '2025-12-11T11:24:11.690387'
original_url: https://thebuildingcoder.typepad.com/blog/1428_whats_new_2017.html
parent_post: 1428_whats_new_2017.md
part_number: '31'
part_total: '43'
post_number: '1428'
series: geometry
slug: whats_new_2017_geometry_api_additions
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
title: API Changes - Geometry API additions
word_count: 363
---

### Geometry API additions
#### ShapeImporter class
The new utility class:
- ShapeImporter
supports conversion of geometry stored in external formats (such as SAT and Rhino) into a collection of Revit geometry objects. Use ShapeImporter.Convert() to generate the geometry objects (and where possible, corresponding materials and graphics styles in the associated document).
#### Builder for 3D boundary representations
The new builder class:
- BRepBuilder
offers the ability to construct Revit boundary representation geometry (either solids or "open sheets") as a result of inputs of surface, edges, and boundary loops of edges. If the construction of the boundary representation is successful, the resulting geometry objects can be used directly in any other Revit tool that accepts geometry, or the BRepBuilder can directly be passed to populate a DirectShape via:
- DirectShape.SetShape(ShapeBuilder)
- DirectShape.AppendShape(ShapeBuilder)
#### New Surface subclasses
Several new subclasses of Surface have been introduced:
- CylindricalSurface
- ConicalSurface
- RuledSurface
- RevolvedSurface
- HermiteSurface
These subclasses expose creation methods and read-only properties suitable for use in constructing import geometry.
#### Frame
New method added
- CanDefineRevitGeometry() – Tests whether the supplied Frame object may be used to define a Revit curve or surface. In order to satisfy the requirements the Frame must be orthonormal and its origin is expected to lie within the Revit design limits.
#### XYZ
New method added
- IsWithinLengthLimits() – Validates that the input point is within Revit design limits.
#### Fixed Reference Sweeps
The new static method:
- GeometryCreationUtilities.CreateFixedReferenceSweptGeometry()
allows creation of a solid using the "fixed reference sweep" method, similar to the method defined in the STEP ISO 10303-42 standard.
A typical use of this method is to create a swept solid for which a line in the cross-section of the solid remains horizontal all along the sweep. As an example, this can be used to construct railings to ensure that the top of the railing remains oriented to the horizontal steps of the stairs. In this example, the fixed reference direction would be chosen to be the upward vertical direction. See the function's description for further details.
As with other GeometryCreationUtilities methods, there is a second version of CreateFixedReferenceSweptGeometry that takes a SolidOptions input, allowing the user to assign a material or graphics style to the solid.
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
