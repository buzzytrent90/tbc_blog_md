---
author: Jeremy Tammik
optimization_date: '2025-12-11T11:24:10.425334'
original_url: https://thebuildingcoder.typepad.com/blog/1551_whats_new_2018.html
parent_post: 1551_whats_new_2018.md
part_number: 08
part_total: '23'
post_number: '1551'
series: geometry
slug: whats_new_2018_methods
source_file: 1551_whats_new_2018.md
tags:
- doors
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
title: API Changes - Methods
word_count: 235
---

### Methods

* Autodesk.Revit.DB.NurbSpline.Create(IList, IList) : NurbSpline
* Autodesk.Revit.DB.NurbSpline.Create(IList, IList, IList, Int32, Boolean, Boolean) : NurbSpline
* Autodesk.Revit.Creation.Application.NewPlane(CurveArray) : Plane
* Autodesk.Revit.Creation.Application.NewPlane(XYZ, XYZ) : Plane
* Autodesk.Revit.Creation.Application.NewPlane(XYZ, XYZ, XYZ) : Plane
* Autodesk.Revit.CurveAPIUtils.CreateNurbSpline(IList, IList) : NurbSpline
* Autodesk.Revit.CurveAPIUtils.CreateNurbSpline(IList, IList, IList, Int32, Boolean, Boolean) : NurbSpline
* Autodesk.Revit.Creation.Document.NewDuct(Connector, Connector, DuctType) : Duct
* Autodesk.Revit.Creation.Document.NewDuct(XYZ, Connector, DuctType) : Duct
* Autodesk.Revit.Creation.Document.NewDuct(XYZ, XYZ, DuctType) : Duct
* Autodesk.Revit.DB.Structure.LoadCase.Create(Document, String, ElementId, LoadNatureCategory) : LoadCase
* Autodesk.Revit.DB.WallFoundation.GetFoundationType() : WallFoundationType
* Autodesk.Revit.DB.WallFoundation.SetFoundationType(WallFoundationType) : Void
* Autodesk.Revit.DB.ViewSchedule.GroupHeaders(Int32, Int32, Int32, Int32) : Void
* Autodesk.Revit.DB.PointClouds.PointCloudOverrides.GetPointCloudOverrideSettings(ElementId, String, Document) : PointCloudOverrideSettings
* Autodesk.Revit.DB.PointClouds.PointCloudOverrides.GetPointCloudOverrideSettings(ElementId) : PointCloudOverrideSettings
* Autodesk.Revit.DB.PointClouds.PointCloudOverrides.SetPointCloudOverrideSettings(ElementId, PointCloudOverrideSettings, String, Document) : Void
* Autodesk.Revit.DB.PointClouds.PointCloudOverrides.SetPointCloudOverrideSettings(ElementId, PointCloudOverrideSettings) : Void
* Autodesk.Revit.DB.DirectShape.CreateElement(Document, ElementId, String, String) : DirectShape
* Autodesk.Revit.DB.DirectShape.CreateElementInstance(Document, ElementId, ElementId, String, Transform, String, String) : DirectShape
* Autodesk.Revit.DB.DirectShape.SetGUIDs(String, String) : Void
* Autodesk.Revit.DB.TextNode.GetFormattedTextRuns() : IList
* Autodesk.Revit.DB.View.GetVisibility(Category) : Boolean
* Autodesk.Revit.DB.View.SetVisibility(Category, Boolean) : Void
* Autodesk.Revit.Creation.Document.NewFoundationWall(WallFoundationType, Wall) : WallFoundation
* Autodesk.Revit.DB.Ellipse.Create(XYZ, double, double, XYZ, XYZ, double, double) : Ellipse
---

## Related Sections
- [API Changes - What's New in the Revit 2018 API](./1551-01_whats_new_in_the_revit_2018_api.md)
- [API Changes - Table of Contents](./1551-02_table_of_contents.md)
- [API Changes - API Changes](./1551-03_api_changes.md)
- [API Changes - Element modifications and contextual commands](./1551-04_element_modifications_and_contextual_commands.md)
- [API Changes - External commands and applications](./1551-05_external_commands_and_applications.md)
- [API Changes - References and selection of subelements](./1551-06_references_and_selection_of_subelements.md)
- [API Changes - Classes](./1551-07_classes.md)
- [API Changes - Properties](./1551-09_properties.md)
- [API Changes - Enumerated types](./1551-10_enumerated_types.md)
- [API Changes - API Additions](./1551-11_api_additions.md)
- [API Changes - Railings API additions related to MultistoryStairs](./1551-12_railings_api_additions_related_to_multistorystairs.md)
- [API Changes - DimensionEqualityLabelFormating API](./1551-13_dimensionequalitylabelformating_api.md)
- [API Changes - UnitsFormatOptions in DimensionType](./1551-14_unitsformatoptions_in_dimensiontype.md)
- [API Changes - OrdinateDimensionSetting](./1551-15_ordinatedimensionsetting.md)
- [API Changes - Surface and Face API](./1551-16_surface_and_face_api.md)
- [API Changes - RevolvedSurface API](./1551-17_revolvedsurface_api.md)
- [API Changes - RuledSurface API](./1551-18_ruledsurface_api.md)
- [API Changes - View update for DirectContext3D](./1551-19_view_update_for_directcontext3d.md)
- [API Changes - Acquire and Publish coordinates API additions](./1551-20_acquire_and_publish_coordinates_api_additions.md)
- [API Changes - SiteLocation API additions](./1551-21_sitelocation_api_additions.md)
- [API Changes - ProjectLocation API additions](./1551-22_projectlocation_api_additions.md)
- [API Changes - Revit Link API additions](./1551-23_revit_link_api_additions.md)
