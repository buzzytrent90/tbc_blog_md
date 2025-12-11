---
author: Jeremy Tammik
optimization_date: '2025-12-11T11:23:48.156926'
original_url: https://thebuildingcoder.typepad.com/blog/0903_whats_new_2012.html
parent_post: 0903_whats_new_2012.md
part_number: '38'
part_total: '45'
post_number: 0903
series: whats_new_in_revit_api
slug: whats_new_2012_rebar_changes
source_file: 0903_whats_new_2012.htm
tags:
- csharp
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
title: What's New in Revit 2012 API - Rebar changes
word_count: 592
---

### Rebar changes

#### RebarShape creation and modification

RebarShape elements are no longer modifiable (except for Allowed Bar Types).
Instead of changing a RebarShape, create a new one based on the old one.
This change results in a simplified API that works in the same way as the UI.

Also, the syntax for RebarShape creation has been changed.
In 2011, the steps were:

1. Create a RebarShape inside the Document.- Create a RebarShapeDefinition inside the RebarShape.- Add data to the RebarShapeDefinition.- Commit the RebarShape.

In 2012, the steps are:

1. Create a RebarShapeDefinition. (It refers to the Document, but is not inside the document.)- Add data to the RebarShapeDefinition.- Create a RebarShape inside the Document based on the RebarShapeDefinition. (Only now is the document modified.)

Specifically, the following methods are removed:

- Autodesk.Revit.Creation.Document.NewRebarShape()- RebarShape.NewDefinitionBySegments()- RebarShape.NewDefinitionByArc()

and replaced by:

- RebarShape.Create(Document, RebarShapeDefinition, ... )- The RebarShapeDefinitionBySegments constructor.- The RebarShapeDefinitionByArc constructors.

The ability to modify hook angle, hook orientation, and style has been removed. These can only be set at creation.

- There is no SetHookAngle(int) method.- There is no SetHookOrientation(int) method.- The SetRebarStyle(RebarStyle) method has been removed.

#### Rebar Shape Parameters

The interface for Rebar Shape Parameters has changed.
Throughout the Rebar API, all methods now use ElementId to stand for a shared parameter, instead of ExternalDefinition.
Methods are provided in the new RebarShapeParameters class to convert between ElementId and ExternalDefinition:

- RebarShapeParameters.GetAllRebarShapeParameters(Document), which returns an array of ElementIds.- RebarShapeParameters.GetElementIdForExternalDefinition(Document, ExternalDefinition), which retrieves or adds the parameter to the document.- RebarShapeParameters.GetExternalDefinitionForElementId(Document, ElementId, DefinitionFile), which retrieves a shared parameter corresponding to an id.- The class RebarShapeMultiplanarDefinition and the RebarShape method getMultiplanarDefinition() have been added to support 3D rebar shape definitions.- Families of category "Structural Connection" support the Structural Material Type parameter. Structural Connection families of type Concrete or Precast Concrete are considered corbels. Corbels support the following features:
          - Hosting rebar.- Autojoining to columns and walls.- Manual joining to other concrete elements.- Families of category "Generic Model" support the "Can Host Rebar" parameter. Turning on this yes/no parameter allows instances to host rebar. Instances also now support the "Volume" parameter. (Join behaviour does not change.)

The following RebarShape properties were replaced with get methods:

- Property int HookAngle[int] -> GetHookAngle(int)- Property RebarHookOrientation HookOrientation[int] -> GetHookOrientation(int)- Property bool SameShapeIgnoringHooks[RebarShape] -> IsSameShapeIgnoringHooks(RebarShape)

The ability to modify hook angle, hook orientation, and style has been removed.

- There is no SetHookAngle(int) method.- There is no SetHookOrientation(int) method.- The SetRebarStyle(RebarStyle) method has been removed.

The return type of RebarShape.GetCurvesForBrowser() was changed from CurveArray to IList<Curve>.

#### RebarHookType

The method

- Autodesk.Revit.Creation.Document.NewRebarHookType(double angle, double multiplier)

was replaced by

- RebarHookType.Create(int angleDegrees, double multiplier)

New members were added to the RebarHookType class are new (its properties were only available through the Parameters interface before).

#### RebarHostData class

The RebarHostData class has the following changes:

- Method getRebarHostData(Element) renamed to GetRebarHostData(Element).- Property CoverType[Reference] replaced with GetCoverType(Reference) and SetCoverType(Reference, CoverType).- Property Valid deprecated; use IsValidHost() instead.- Method HasCoverTypeForReference(Reference) deprecated; use IsFaceExposed(Reference) instead.- New methods pertaining to cover: GetExposedFaces(), GetCommonCoverType(), SetCommonCoverType(CoverType).- Other new methods: GetRebarsInHost(), GetAreaReinforcementsInHost(), GetPathReinforcementsInHost().

#### Rebar class

The two methods Creation.Document.NewRebar() were replaced by:

- Rebar.CreateFromCurves()- Rebar.CreateFromRebarShape()

The new methods are similar to the old ones, except that they no longer return null for invalid arguments, and they no longer regenerate the document.

The new class RebarBendData is to support functionality that is not part of Alpha 3.

## MEP API small enhancements and changes
---

## Related Sections
- [What's New in Revit 2012 API - Unexpected Places, and Warmer](./0903-01_unexpected_places_and_warmer.md)
- [What's New in Revit 2012 API - Move method replacements](./0903-02_move_method_replacements.md)
- [What's New in Revit 2012 API - Mirror method replacements](./0903-03_mirror_method_replacements.md)
- [What's New in Revit 2012 API - Rotate method replacements](./0903-04_rotate_method_replacements.md)
- [What's New in Revit 2012 API - New Copy methods](./0903-05_new_copy_methods.md)
- [What's New in Revit 2012 API - LinearArray creation replacement methods](./0903-06_lineararray_creation_replacement_methods.md)
- [What's New in Revit 2012 API - RadialArray creation replacement methods](./0903-07_radialarray_creation_replacement_methods.md)
- [What's New in Revit 2012 API - AnalyticalModel now an Element](./0903-08_analyticalmodel_now_an_element.md)
- [What's New in Revit 2012 API - AnalyticalModelSelector](./0903-09_analyticalmodelselector.md)
- [What's New in Revit 2012 API - SlabFoundationType](./0903-10_slabfoundationtype.md)
- [What's New in Revit 2012 API - Get original geometry of a FamilyInstance](./0903-11_get_original_geometry_of_a_familyinstance.md)
- [What's New in Revit 2012 API - Extrusion analysis of a solid](./0903-12_extrusion_analysis_of_a_solid.md)
- [What's New in Revit 2012 API - GeometryCreationUtilities](./0903-13_geometrycreationutilities.md)
- [What's New in Revit 2012 API - Find 3D elements by intersection](./0903-14_find_3d_elements_by_intersection.md)
- [What's New in Revit 2012 API - Boolean operations](./0903-15_boolean_operations.md)
- [What's New in Revit 2012 API - HostObject – top, bottom, side faces](./0903-16_hostobject_top_bottom_side_faces.md)
- [What's New in Revit 2012 API - Get host face of a FamilyInstance](./0903-17_get_host_face_of_a_familyinstance.md)
- [What's New in Revit 2012 API - Element.Geometry](./0903-18_elementgeometry.md)
- [What's New in Revit 2012 API - GeometryObject.GraphicsStyleId](./0903-19_geometryobjectgraphicsstyleid.md)
- [What's New in Revit 2012 API - Curve representation of an Edge](./0903-20_curve_representation_of_an_edge.md)
- [What's New in Revit 2012 API - Centroid of a Solid](./0903-21_centroid_of_a_solid.md)
- [What's New in Revit 2012 API - Transforming geometry](./0903-22_transforming_geometry.md)
- [What's New in Revit 2012 API - Instance.GetTransform() and Instance.GetTotalTransform()](./0903-23_instancegettransform_and_instancegettotaltransform.md)
- [What's New in Revit 2012 API - Serialization/deserialization of References](./0903-24_serializationdeserialization_of_references.md)
- [What's New in Revit 2012 API - Face.HasRegions & Face.GetRegions()](./0903-25_facehasregions_facegetregions.md)
- [What's New in Revit 2012 API - Face.MaterialElementId replaces Face.MaterialElement](./0903-26_facematerialelementid_replaces_facematerialelement.md)
- [What's New in Revit 2012 API - NewHermiteSpline tangency control](./0903-27_newhermitespline_tangency_control.md)
- [What's New in Revit 2012 API - New NewNurbSpline overload](./0903-28_new_newnurbspline_overload.md)
- [What's New in Revit 2012 API - PolyLine returned from Element.Geometry](./0903-29_polyline_returned_from_elementgeometry.md)
- [What's New in Revit 2012 API - Pipe settings and sizes](./0903-30_pipe_settings_and_sizes.md)
- [What's New in Revit 2012 API - Placeholder ducts and pipes](./0903-31_placeholder_ducts_and_pipes.md)
- [What's New in Revit 2012 API - Duct & pipe insulation & lining](./0903-32_duct_pipe_insulation_lining.md)
- [What's New in Revit 2012 API - VSTA enabled for multiple Revit sessions](./0903-33_vsta_enabled_for_multiple_revit_sessions.md)
- [What's New in Revit 2012 API - Track Changes UI](./0903-34_track_changes_ui.md)
- [What's New in Revit 2012 API - LineLoad.UniformLoad](./0903-35_lineloaduniformload.md)
- [What's New in Revit 2012 API - NewBeamSystem() changes](./0903-36_newbeamsystem_changes.md)
- [What's New in Revit 2012 API - NewTruss() change](./0903-37_newtruss_change.md)
- [What's New in Revit 2012 API - Spare and space circuits](./0903-39_spare_and_space_circuits.md)
- [What's New in Revit 2012 API - Cable tray and conduit domain](./0903-40_cable_tray_and_conduit_domain.md)
- [What's New in Revit 2012 API - Connector](./0903-41_connector.md)
- [What's New in Revit 2012 API - MEPSystem](./0903-42_mepsystem.md)
- [What's New in Revit 2012 API - Graphical warnings for disconnects](./0903-43_graphical_warnings_for_disconnects.md)
- [What's New in Revit 2012 API - Space properties](./0903-44_space_properties.md)
- [What's New in Revit 2012 API - Fitting methods](./0903-45_fitting_methods.md)
