---
author: Jeremy Tammik
optimization_date: '2025-12-11T11:24:14.304142'
original_url: https://thebuildingcoder.typepad.com/blog/1141_whats_new_2015.html
parent_post: 1141_whats_new_2015.md
part_number: '34'
part_total: '97'
post_number: '1141'
series: geometry
slug: whats_new_2015_curveloop.createviathicken()
source_file: 1141_whats_new_2015.htm
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
title: API Changes - CurveLoop.CreateViaThicken()
word_count: 804
---

### CurveLoop.CreateViaThicken()

Previously, when this function couldn't create a compatible CurveLoop, it would return null. It now throws an exception in this situation.

## Obsolete API removal

The following obsolete APIs and classes have been removed:

- Autodesk.Revit.Creation.Application.NewArc(Plane, Double, Double, Double)
- Autodesk.Revit.Creation.Application.NewArc(XYZ, Double, Double, Double, XYZ, XYZ)
- Autodesk.Revit.Creation.Application.NewArc(XYZ, XYZ, XYZ)
- Autodesk.Revit.Creation.Application.NewEllipse(XYZ, Double, Double, XYZ, XYZ, Double, Double)
- Autodesk.Revit.Creation.Application.NewHermiteSpline(IList<XYZ>, Boolean)
- Autodesk.Revit.Creation.Application.NewHermiteSpline(IList<XYZ>, Boolean, XYZ, XYZ)
- Autodesk.Revit.Creation.Application.NewLine(XYZ, XYZ, Boolean)
- Autodesk.Revit.Creation.Application.NewLineBound(XYZ, XYZ)
- Autodesk.Revit.Creation.Application.NewLineUnbound(XYZ, XYZ)
- Autodesk.Revit.Creation.Application.NewNurbSpline(IList<XYZ>, DoubleArray, DoubleArray, Int32, Boolean, Boolean)
- Autodesk.Revit.Creation.Application.NewNurbSpline(IList<XYZ>, IList<Double>)
- Autodesk.Revit.Creation.Application.NewSATExportOptions()
- Autodesk.Revit.Creation.Document.NewAreaReinforcement(Element, CurveArray, XYZ)
- Autodesk.Revit.Creation.Document.NewBeamSystem(CurveArray, Level)
- Autodesk.Revit.Creation.Document.NewBeamSystem(CurveArray, Level, XYZ, Boolean)
- Autodesk.Revit.Creation.Document.NewBeamSystem(CurveArray, SketchPlane)
- Autodesk.Revit.Creation.Document.NewBeamSystem(CurveArray, SketchPlane, XYZ, Boolean)
- Autodesk.Revit.Creation.Document.NewPathReinforcement(Element, CurveArray, Boolean)
- Autodesk.Revit.Creation.Document.NewRebarBarType()
- Autodesk.Revit.Creation.Document.NewRoomTag(Room, UV, View)
- Autodesk.Revit.Creation.Document.NewTopographySurface(IList<XYZ>)
- Autodesk.Revit.Creation.Document.NewTruss(TrussType, SketchPlane, Curve)
- Autodesk.Revit.Creation.Document.NewViewSheet(FamilySymbol)
- Autodesk.Revit.Creation.FamilyItemFactory.NewDividedSurface(Reference)
- Autodesk.Revit.Creation.ItemFactoryBase.NewSketchPlane(PlanarFace)
- Autodesk.Revit.Creation.ItemFactoryBase.NewSketchPlane(Plane)
- Autodesk.Revit.Creation.ItemFactoryBase.NewSketchPlane(Reference)
- Autodesk.Revit.DB.Architecture.BoundaryLocationType
- Autodesk.Revit.DB.Architecture.StairsRun.ExtensionBelowBase
- Autodesk.Revit.DB.Curve.EndParameter[Int32]
- Autodesk.Revit.DB.Curve.EndPoint[Int32]
- Autodesk.Revit.DB.Curve.EndPointReference[Int32]
- Autodesk.Revit.DB.Curve.Transformed[Transform]
- Autodesk.Revit.DB.Dimension.Label
- Autodesk.Revit.DB.DividedSurfaceData
- Autodesk.Revit.DB.Document.AnnotationSymbolTypes
- Autodesk.Revit.DB.Document.BeamSystemTypes
- Autodesk.Revit.DB.Document.ContFootingTypes
- Autodesk.Revit.DB.Document.CurtainSystemTypes
- Autodesk.Revit.DB.Document.DeckProfiles
- Autodesk.Revit.DB.Document.Delete(Element)
- Autodesk.Revit.DB.Document.DimensionTypes
- Autodesk.Revit.DB.Document.ElectricalEquipmentTypes
- Autodesk.Revit.DB.Document.Export(String, String, ViewSet, SATExportOptions)
- Autodesk.Revit.DB.Document.FasciaTypes
- Autodesk.Revit.DB.Document.FindReferencesWithContextByDirection(XYZ, XYZ, View3D)
- Autodesk.Revit.DB.Document.FloorTypes
- Autodesk.Revit.DB.Document.GridTypes
- Autodesk.Revit.DB.Document.GutterTypes
- Autodesk.Revit.DB.Document.LevelTypes
- Autodesk.Revit.DB.Document.LightingDeviceTypes
- Autodesk.Revit.DB.Document.LightingFixtureTypes
- Autodesk.Revit.DB.Document.MechanicalEquipmentTypes
- Autodesk.Revit.DB.Document.RebarBarTypes
- Autodesk.Revit.DB.Document.RebarCoverTypes
- Autodesk.Revit.DB.Document.RebarHookTypes
- Autodesk.Revit.DB.Document.RebarShapes
- Autodesk.Revit.DB.Document.RoofTypes
- Autodesk.Revit.DB.Document.RoomTagTypes
- Autodesk.Revit.DB.Document.SlabEdgeTypes
- Autodesk.Revit.DB.Document.SpaceTagTypes
- Autodesk.Revit.DB.Document.SpotDimensionTypes
- Autodesk.Revit.DB.Document.TextNoteTypes
- Autodesk.Revit.DB.Document.TitleBlocks
- Autodesk.Revit.DB.Document.TrussTypes
- Autodesk.Revit.DB.Document.ViewSheetSets
- Autodesk.Revit.DB.Document.WallTypes
- Autodesk.Revit.DB.Edge.EndPointReference[Int32]
- Autodesk.Revit.DB.Edge.Face[Int32]
- Autodesk.Revit.DB.Element.GetDividedSurfaceData()
- Autodesk.Revit.DB.Element.GetMaterialArea(Material)
- Autodesk.Revit.DB.Element.GetMaterialVolume(Material)
- Autodesk.Revit.DB.Element.Group
- Autodesk.Revit.DB.Element.Level
- Autodesk.Revit.DB.Element.Materials
- Autodesk.Revit.DB.IFC.IFCDoorWindowInfo
- Autodesk.Revit.DB.Line.Bound[XYZ, XYZ]
- Autodesk.Revit.DB.Line.Unbound[XYZ, XYZ]
- Autodesk.Revit.DB.Material.CutPattern
- Autodesk.Revit.DB.Material.GetCutPatternColor()
- Autodesk.Revit.DB.Material.GetCutPatternId()
- Autodesk.Revit.DB.Material.GetRenderAppearance()
- Autodesk.Revit.DB.Material.RenderAppearance
- Autodesk.Revit.DB.Material.SetRenderAppearance(Asset)
- Autodesk.Revit.DB.Material.SurfacePattern
- Autodesk.Revit.DB.MEPSystem.IsDefaultSystem
- Autodesk.Revit.DB.ParameterFilterElement.AllCategoriesFilterable(ICollection<ElementId>)
- Autodesk.Revit.DB.Plumbing.PipeConnectionType
- Autodesk.Revit.DB.Plumbing.PipeSettings.ElbowAngleIncrement
- Autodesk.Revit.DB.Plumbing.PipeType.ConnectionType
- Autodesk.Revit.DB.PointCloudInstance.GetPoints(PointCloudFilter, Int32)
- Autodesk.Revit.DB.SaveAsOptions.Rename
- Autodesk.Revit.DB.Settings.VolumeCalculationSetting
- Autodesk.Revit.DB.SketchPlane.Plane
- Autodesk.Revit.DB.SketchPlane.PlaneReference
- Autodesk.Revit.DB.StairsEditScope.Commit()
- Autodesk.Revit.DB.Structure.FabricArea.Create(Document, Element, IList<CurveLoop>, XYZ)
- Autodesk.Revit.DB.Structure.FabricArea.SetFabricLocation(FabricLocation)
- Autodesk.Revit.DB.Structure.FabricArea.SetFabricSheetTypeId(ElementId)
- Autodesk.Revit.DB.Structure.FabricArea.SetMajorSheetAlignment(FabricSheetAlignment)
- Autodesk.Revit.DB.Structure.FabricArea.SetMinorSheetAlignment(FabricSheetAlignment)
- Autodesk.Revit.DB.Structure.FabricSheet.SheetTypeId
- Autodesk.Revit.DB.Structure.FabricSheetType.PhysicalMaterialAsset
- Autodesk.Revit.DB.Structure.RebarShape.GetHookAngle(Int32)
- Autodesk.Revit.DB.Structure.RebarShape.GetHookOrientation(Int32)
- Autodesk.Revit.DB.Structure.RebarShapeDefinitionBySegments.AddBendDefaultRadius(Int32, Int32, RebarShapeBendAngle)
- Autodesk.Revit.DB.Structure.RebarShapeDefinitionBySegments.AddBendVariableRadius(Int32, Int32, RebarShapeBendAngle, ElementId, Boolean)
- Autodesk.Revit.DB.Transform.Reflection[Plane]
- Autodesk.Revit.DB.Transform.Rotation[XYZ, XYZ, Double]
- Autodesk.Revit.DB.Transform.Translation[XYZ]
- Autodesk.Revit.DB.View.CutColorOverrideByElement[ICollection<ElementId>]
- Autodesk.Revit.DB.View.CutLinePatternOverrideByElement[ICollection<ElementId>]
- Autodesk.Revit.DB.View.CutLineWeightOverrideByElement[ICollection<ElementId>]
- Autodesk.Revit.DB.View.GetVisibility(Category)
- Autodesk.Revit.DB.View.ProjColorOverrideByElement[ICollection<ElementId>]
- Autodesk.Revit.DB.View.ProjLinePatternOverrideByElement[ICollection<ElementId>]
- Autodesk.Revit.DB.View.ProjLineWeightOverrideByElement[ICollection<ElementId>]
- Autodesk.Revit.DB.View.SetVisibility(Category, Boolean)
- Autodesk.Revit.DB.View3D.SectionBox
- Autodesk.Revit.DB.VolumeCalculationOptions
- Autodesk.Revit.DB.VolumeCalculationSetting
- Autodesk.Revit.Utility.AssetPropertyReference.Value

# Major API Additions

## View API changes
---

## Related Sections
- [API Changes - What's New in the Revit 2015 API](./1141-01_whats_new_in_the_revit_2015_api.md)
- [API Changes - Major changes and renovations to the Revit API](./1141-02_major_changes_and_renovations_to_the_revit_api.md)
- [API Changes - Element.Parameter[String]](./1141-03_elementparameterstring.md)
- [API Changes - Element.Parameters](./1141-04_elementparameters.md)
- [API Changes - Shared parameter creation – description and user modifiability](./1141-05_shared_parameter_creation_description_and_user_mod.md)
- [API Changes - WorksetConfiguration methods](./1141-06_worksetconfiguration_methods.md)
- [API Changes - SynchronizeWithCentralOptions.CompactCentralFile](./1141-07_synchronizewithcentraloptionscompactcentralfile.md)
- [API Changes - FamilyBase class removed](./1141-08_familybase_class_removed.md)
- [API Changes - Family.Symbols](./1141-09_familysymbols.md)
- [API Changes - Family.CurtainPanelHorizontalSpacing and Family.CurtainPanelVerticalSpacing](./1141-10_familycurtainpanelhorizontalspacing_and_familycurt.md)
- [API Changes - View display settings](./1141-11_view_display_settings.md)
- [API Changes - ViewSheet members related to Revisions](./1141-12_viewsheet_members_related_to_revisions.md)
- [API Changes - AnalyticalModel members obsoleted](./1141-13_analyticalmodel_members_obsoleted.md)
- [API Changes - AreaReinforcement API changes](./1141-14_areareinforcement_api_changes.md)
- [API Changes - PathReinforcement API changes](./1141-15_pathreinforcement_api_changes.md)
- [API Changes - FabricArea API changes](./1141-16_fabricarea_api_changes.md)
- [API Changes - RebarHookType API changes](./1141-17_rebarhooktype_api_changes.md)
- [API Changes - Miscellaneous changes](./1141-18_miscellaneous_changes.md)
- [API Changes - Connector properties removed](./1141-19_connector_properties_removed.md)
- [API Changes - Creation.Document.NewWire()](./1141-20_creationdocumentnewwire.md)
- [API Changes - Related Wire API additions](./1141-21_related_wire_api_additions.md)
- [API Changes - Obsoleted functions, classes, and enums](./1141-22_obsoleted_functions_classes_and_enums.md)
- [API Changes - Changed functions, classes, and enums](./1141-23_changed_functions_classes_and_enums.md)
- [API Changes - New functions, classes, and enums](./1141-24_new_functions_classes_and_enums.md)
- [API Changes - ElementId properties in EnergyAnaysisDetailModel contents](./1141-25_elementid_properties_in_energyanaysisdetailmodel_c.md)
- [API Changes - Material API](./1141-26_material_api.md)
- [API Changes - TableSectionData.InsertColumn(int index, bool bCreateCellData)](./1141-27_tablesectiondatainsertcolumnint_index_bool_bcreate.md)
- [API Changes - BoundaryConditions](./1141-28_boundaryconditions.md)
- [API Changes - BuiltInCategory.OST\_MassWindow](./1141-29_builtincategoryost_masswindow.md)
- [API Changes - ElementIntersectsElementFilter](./1141-30_elementintersectselementfilter.md)
- [API Changes - ExtensibleStorageFilter](./1141-31_extensiblestoragefilter.md)
- [API Changes - MeshTriangle](./1141-32_meshtriangle.md)
- [API Changes - CurtainGridLine.Move()](./1141-33_curtaingridlinemove.md)
- [API Changes - Active graphical view](./1141-35_active_graphical_view.md)
- [API Changes - Sketchy lines settings](./1141-36_sketchy_lines_settings.md)
- [API Changes - Family Types](./1141-37_family_types.md)
- [API Changes - Non-family Types](./1141-38_non_family_types.md)
- [API Changes - Reinforcement numbering](./1141-39_reinforcement_numbering.md)
- [API Changes - Reinforcement in parts](./1141-40_reinforcement_in_parts.md)
- [API Changes - Rebar presentation mode](./1141-41_rebar_presentation_mode.md)
- [API Changes - Place FabricSheet directly in host](./1141-42_place_fabricsheet_directly_in_host.md)
- [API Changes - Creating default reinforcement types](./1141-43_creating_default_reinforcement_types.md)
- [API Changes - Miscellaneous reinforcement API additions](./1141-44_miscellaneous_reinforcement_api_additions.md)
- [API Changes - AnalyticalModel API additions](./1141-45_analyticalmodel_api_additions.md)
- [API Changes - Loads and Boundary Conditions API](./1141-46_loads_and_boundary_conditions_api.md)
- [API Changes - StructuralFramingUtils](./1141-47_structuralframingutils.md)
- [API Changes - RevisionSettings class](./1141-48_revisionsettings_class.md)
- [API Changes - Revision class](./1141-49_revision_class.md)
- [API Changes - RevisionCloud class](./1141-50_revisioncloud_class.md)
- [API Changes - Revision cloud geometry](./1141-51_revision_cloud_geometry.md)
- [API Changes - Parameter order](./1141-52_parameter_order.md)
- [API Changes - Family parameter creation – description](./1141-53_family_parameter_creation_description.md)
- [API Changes - Stacked wall](./1141-54_stacked_wall.md)
- [API Changes - Wall Function](./1141-55_wall_function.md)
- [API Changes - Schedule filters](./1141-56_schedule_filters.md)
- [API Changes - ScheduleDefinition.GrandTotalTitle](./1141-57_scheduledefinitiongrandtotaltitle.md)
- [API Changes - Images in schedules](./1141-58_images_in_schedules.md)
- [API Changes - IFC import options and operations](./1141-59_ifc_import_options_and_operations.md)
- [API Changes - ImporterIFC new properties and functions](./1141-60_importerifc_new_properties_and_functions.md)
- [API Changes - Built-in parameter changes](./1141-61_built_in_parameter_changes.md)
- [API Changes - DirectShape](./1141-62_directshape.md)
- [API Changes - TessellatedShapeBuilder](./1141-63_tessellatedshapebuilder.md)
- [API Changes - IExternalResourceServer](./1141-64_iexternalresourceserver.md)
- [API Changes - IExternalResourceUIServer](./1141-65_iexternalresourceuiserver.md)
- [API Changes - ExternalResourceReference](./1141-66_externalresourcereference.md)
- [API Changes - KeyBasedTreeEntryTable](./1141-67_keybasedtreeentrytable.md)
- [API Changes - KeyBasedTreeEntry](./1141-68_keybasedtreeentry.md)
- [API Changes - gbXML export options](./1141-69_gbxml_export_options.md)
- [API Changes - BuildingEnvelopeAnalyzer Class](./1141-70_buildingenvelopeanalyzer_class.md)
- [API Changes - Document.IsDetached](./1141-71_documentisdetached.md)
- [API Changes - Document.DocumentWorksharingEnabled](./1141-72_documentdocumentworksharingenabled.md)
- [API Changes - UIDocument operations and additions](./1141-73_uidocument_operations_and_additions.md)
- [API Changes - External Resource compatibility](./1141-74_external_resource_compatibility.md)
- [API Changes - RevitLinkType.AttachmentType](./1141-75_revitlinktypeattachmenttype.md)
- [API Changes - Category.CategoryType](./1141-76_categorycategorytype.md)
- [API Changes - ElementType.FamilyName](./1141-77_elementtypefamilyname.md)
- [API Changes - ElementType duplicating events](./1141-78_elementtype_duplicating_events.md)
- [API Changes - View.IsAssemblyView](./1141-79_viewisassemblyview.md)
- [API Changes - View.Title](./1141-80_viewtitle.md)
- [API Changes - Categories hidden status](./1141-81_categories_hidden_status.md)
- [API Changes - Drafting view creation](./1141-82_drafting_view_creation.md)
- [API Changes - Orient 3D view](./1141-83_orient_3d_view.md)
- [API Changes - Reference callouts and sections](./1141-84_reference_callouts_and_sections.md)
- [API Changes - FamilyManager.IsUserAssignableParameterGroup](./1141-85_familymanagerisuserassignableparametergroup.md)
- [API Changes - LocationPoint.Rotation for view-specific family instances](./1141-86_locationpointrotation_for_view_specific_family_ins.md)
- [API Changes - PromptForFamilyInstancePlacement() option for Air Terminals on Ducts](./1141-87_promptforfamilyinstanceplacement_option_for_air_te.md)
- [API Changes - Family loading events](./1141-88_family_loading_events.md)
- [API Changes - Curve](./1141-89_curve.md)
- [API Changes - CurveLoop](./1141-90_curveloop.md)
- [API Changes - Face](./1141-91_face.md)
- [API Changes - CurveElement](./1141-92_curveelement.md)
- [API Changes - FreeFormElement](./1141-93_freeformelement.md)
- [API Changes - Material.UseRenderAppearanceForShading](./1141-94_materialuserenderappearanceforshading.md)
- [API Changes - Region edges mask coincident lines](./1141-95_region_edges_mask_coincident_lines.md)
- [API Changes - New connector properties](./1141-96_new_connector_properties.md)
- [API Changes - Drag & drop API](./1141-97_drag_drop_api.md)
