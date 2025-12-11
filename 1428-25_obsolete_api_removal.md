---
author: Jeremy Tammik
optimization_date: '2025-12-11T11:24:11.682621'
original_url: https://thebuildingcoder.typepad.com/blog/1428_whats_new_2017.html
parent_post: 1428_whats_new_2017.md
part_number: '25'
part_total: '43'
post_number: '1428'
series: geometry
slug: whats_new_2017_obsolete_api_removal
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
title: API Changes - Obsolete API removal
word_count: 615
---

### Obsolete API removal
The following API members and classes which had previously been marked Obsolete have been removed in this release. Consult the API documentation from prior releases for information on the replacements to use:
#### Methods
- Autodesk.Revit.Creation.Application.NewPoint(XYZ)
- Autodesk.Revit.Creation.Document.NewGrid(Arc)
- Autodesk.Revit.Creation.Document.NewGrid(Line)
- Autodesk.Revit.Creation.Document.NewGrids(CurveArray)
- Autodesk.Revit.Creation.ItemFactoryBase.NewLevel(Double)
- Autodesk.Revit.Creation.ItemFactoryBase.NewTextNote(View, XYZ, XYZ, XYZ, Double, TextAlignFlags, TextNoteLeaderTypes, TextNoteLeaderStyles, XYZ, XYZ, String)
- Autodesk.Revit.Creation.ItemFactoryBase.NewTextNote(View, XYZ, XYZ, XYZ, Double, TextAlignFlags, String)
- Autodesk.Revit.Creation.Document.NewPointLoad(Reference, XYZ, XYZ, Boolean, PointLoadType, SketchPlane)
- Autodesk.Revit.Creation.Document.NewPointLoad(XYZ, XYZ, XYZ, Boolean, PointLoadType, SketchPlane)
- Autodesk.Revit.Creation.Document.NewLineLoad(Reference, IList, IList, Boolean, Boolean, Boolean, LineLoadType, SketchPlane) Autodesk.Revit.Creation.Document.NewLineLoad(Element, IList, IList, Boolean, Boolean, Boolean, LineLoadType, SketchPlane)
- Autodesk.Revit.Creation.Document.NewLineLoad(IList, IList, IList, Boolean, Boolean, Boolean, LineLoadType, SketchPlane)
- Autodesk.Revit.Creation.Document.NewLineLoad(XYZ, XYZ, XYZ, XYZ, XYZ, XYZ, Boolean, Boolean, Boolean, LineLoadType, SketchPlane)
- Autodesk.Revit.Creation.Document.NewAreaLoad(Element, XYZ, Boolean, AreaLoadType) Autodesk.Revit.Creation.Document.NewAreaLoad(CurveArray, Int32[], Int32[], XYZ, XYZ, XYZ, Boolean, AreaLoadType)
- Autodesk.Revit.Creation.Document.NewAreaLoad(CurveArray, Int32[], Int32[], IList, Boolean, AreaLoadType) Autodesk.Revit.Creation.Document.NewAreaLoad(IList, XYZ, Boolean, AreaLoadType)
- Autodesk.Revit.Creation.Document.NewLoadNature(String)
- Autodesk.Revit.Creation.Document.NewLoadUsage(String)
- Autodesk.Revit.Creation.Document.NewLoadCase(String, LoadNature, Category)
- Autodesk.Revit.Creation.Document.NewLoadCombination(String, Int32, Int32, Double[], LoadCaseArray, LoadCombinationArray, LoadUsageArray)
- Autodesk.Revit.DB.ElementTransformUtils::MirrorElements(Document,ElementIdSet,Plane)
- Autodesk.Revit.DB.Grid.ExtendToAllLevels()
- Autodesk.Revit.DB.Family.HasStructuralSection()
- Autodesk.Revit.DB.FamilySymbol.HasStructuralSection()
- Autodesk.Revit.DB.TessellatedShapeBuilder.Build(TessellatedShapeBuilderTarget, TessellatedShapeBuilderFallback, ElementId)
- Autodesk.Revit.DB.ViewShapeBuilder.SetShape(DirectShape)
- Autodesk.Revit.DB.ViewShapeBuilder.SetShape(DirectShapeType)
- Autodesk.Revit.DB.ViewCropRegionShapeManager.SetCropRegionShape(CurveLoop)
- Autodesk.Revit.DB.ViewCropRegionShapeManager.SetCropRegionEmptyShape()
- Autodesk.Revit.DB.ViewCropRegionShapeManager.GetCropRegionShape()
- Autodesk.Revit.DB.Structure.ReinforcementSettings.DocumentContainsNoAreaOrPathReinforcement()
- Autodesk.Revit.DB.Structure.ReinforcementSettings.DocumentContainsNoRebar()
#### Properties
- Autodesk.Revit.ApplicationServices.Application.IsQuiescent
- Autodesk.Revit.ApplicationServices.ControlledApplication.IsQuiescent
- Autodesk.Revit.DB.AnnotationSymbol.Leaders
- Autodesk.Revit.DB.BoundarySegment.Document
- Autodesk.Revit.DB.BoundarySegment.Element
- Autodesk.Revit.DB.BoundarySegment.Curve
- Autodesk.Revit.DB.CustomExporter.IncludeFaces
- Autodesk.Revit.DB.Floor.StructuralUsage
- Autodesk.Revit.DB.Grid.GridType
- Autodesk.Revit.DB.Level.LevelType
- Autodesk.Revit.DB.Level.PlaneReference
- Autodesk.Revit.DB.PlanarFace.Normal
- Autodesk.Revit.DB.PlanarFace.Vector
- Autodesk.Revit.DB.ReferencePlane.Plane
- Autodesk.Revit.DB.ReferencePlane.Reference
- Autodesk.Revit.DB.RevisionSettings.RevisionAlphabet
- Autodesk.Revit.DB.TextElement.Align
- Autodesk.Revit.DB.TextElement.LineWidth
- Autodesk.Revit.DB.TextNote.Leaders
- Autodesk.Revit.DB.ViewCropRegionShapeManager.Valid
- Autodesk.Revit.DB.Structure.AreaLoad.Force
- Autodesk.Revit.DB.Structure.AreaLoad.Force1
- Autodesk.Revit.DB.Structure.AreaLoad.Force2
- Autodesk.Revit.DB.Structure.AreaLoad.Force3
- Autodesk.Revit.DB.Structure.AreaLoad.RefPoint
- Autodesk.Revit.DB.Structure.AreaLoad.NumCurves
- Autodesk.Revit.DB.Structure.AreaLoad.NumLoops
- Autodesk.Revit.DB.Structure.AreaLoad.Curve
- Autodesk.Revit.DB.Structure.LineLoad.Point
- Autodesk.Revit.DB.Structure.LineLoad.Force
- Autodesk.Revit.DB.Structure.LineLoad.Force1
- Autodesk.Revit.DB.Structure.LineLoad.Force2
- Autodesk.Revit.DB.Structure.LineLoad.UniformLoad
- Autodesk.Revit.DB.Structure.LineLoad.ProjectedLoad
- Autodesk.Revit.DB.Structure.LineLoad.Moment
- Autodesk.Revit.DB.Structure.LineLoad.Moment1
- Autodesk.Revit.DB.Structure.LineLoad.Moment2
- Autodesk.Revit.DB.Structure.PointLoad.Force
- Autodesk.Revit.DB.Structure.PointLoad.Moment
- Autodesk.Revit.DB.Structure.LoadCombination.CombinationTypeIndex
- Autodesk.Revit.DB.Structure.LoadCombination.CombinationStateIndex
- Autodesk.Revit.DB.Structure.LoadCombination.CombinationType
- Autodesk.Revit.DB.Structure.LoadCombination.CombinationState
- Autodesk.Revit.DB.Structure.LoadCombination.NumberOfComponents
- Autodesk.Revit.DB.Structure.LoadCombination.Factor
- Autodesk.Revit.DB.Structure.LoadCombination.CombinationCaseName
- Autodesk.Revit.DB.Structure.LoadCombination.CombinationNatureName
- Autodesk.Revit.DB.Structure.LoadCombination.NumberOfUsages
- Autodesk.Revit.DB.Structure.LoadCombination.UsageName
- Autodesk.Revit.DB.Structure.BoundaryConditions.NumCurves
- Autodesk.Revit.DB.Structure.BoundaryConditions.Curve
- Autodesk.Revit.DB.Structure.BoundaryConditions.AssociatedLoad
#### Enumerated types
- Autodesk.Revit.DB.SlabFoundationType
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
