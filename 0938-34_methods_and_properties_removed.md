---
author: Jeremy Tammik
optimization_date: '2025-12-11T11:23:50.409577'
original_url: https://thebuildingcoder.typepad.com/blog/0938_whats_new_2014.html
parent_post: 0938_whats_new_2014.md
part_number: '34'
part_total: '126'
post_number: 0938
series: geometry
slug: whats_new_2014_methods_and_properties_removed
source_file: 0938_whats_new_2014.htm
tags:
- csharp
- doors
- elements
- family
- filtering
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
- vbnet
- views
- walls
- windows
title: Major Changes and Renovations - Methods and Properties removed
word_count: 768
---

### Methods and Properties removed

#### Autodesk.Revit.Creation namespace

##### Application

- NewMaterialSet() – replaced by .net Generic collection classes
- NewElementArray() – replaced by .net Generic collection classes

##### Document

- NewAnnotationSymbol(XYZ ,AnnotationSymbolType ,View) – replaced by NewFamilyInstance(XYZ, FamilySymbol, View)
- NewAreaViewPlan(String,Level,AreaElemType) – replaced by ViewPlan.CreateAreaPlan()
- NewCurtainSystem(ReferenceArray,CurtainSystemType) – replaced by NewCurtainSystem2(ReferenceArray, CurtainSystemType)
- NewElectricalSystem(ElementSet,ElectricalSystemType) – replaced by NewElectricalSystem(ICollection<ElementId>, ElectricalSystemType)
- NewFamilyInstances(List<FamilyInstanceCreationData>) – replaced by NewFamilyInstances2(List<FamilyInstanceCreationData>)
- NewGroup(ElementSet) – replaced by NewGroup(System.Collections.Generic.ICollection<Autodesk.Revit.DB.ElementId>)
- NewRooms(Phase, Int32) – replaced by NewRooms2(Phase, Int32)
- NewRooms(Level,Phase) – replaced by NewRooms2(Level, Phase)
- NewRooms(Level) – replaced by NewRooms2(Level)
- NewRooms(List<RoomCreationData>) – there is no single equivalent that creates multiple rooms, this is not needed with Revit API control over regeneration
- NewSpaces(Phase,Int32) – replaced by NewSpaces2(Phase, Int32)
- NewSpaces(Level,Phase,View) – replaced by NewSpaces2(Level, Phase, View)
- NewTextNotes(List<TextNoteCreationData>) – replaced by NewTextNote()
- NewViewPlan(String,Level,ViewPlanType) – replaced by ViewPlan.Create(Document, ElementId, ElementId)
- NewView3D(XYZ) – replaced by View3D.CreateIsometric(Document, ElementId)
- NewViewSection(BoundingBoxXYZ) – replaced by ViewSection.CreateDetail()
- All Wall creation methods replaced by equivalent Wall.Create() methods:
- NewWall(CurveArray,WallType,Level,Boolean,XYZ)
- NewWall(CurveArray,WallType,Level,Boolean)
- NewWall(CurveArray,Boolean)
- NewWall(Curve,WallType,Level,Double,Double,Boolean,Boolean)
- NewWall(Curve,Level,Boolean)
- NewWalls(List<ProfiledWallCreationData> dataList) – there is no single equivalent that creates multiple walls, this is not needed with Revit API control over regeneration
- NewWalls(List<RectangularWallCreationData> dataList) – there is no single equivalent that creates multiple walls, this is not needed with Revit API control over regeneration

##### FamilyItemFactory

- NewDuctConnector(Reference,DuctSystemType) – replaced by ConnectorElement.CreateDuctConnector()
- NewPipeConnector(Reference,PipeSystemType) – replaced by ConnectorElement.CreatePipeConnector()
- NewElectricalConnector(Reference,ElectricalSystemType) – replaced by ConnectorElement.CreateElectricalConnector()

#### Autodesk.Revit.DB namespace

##### BaseArray

- CopyMembers – replaced by GetCopiedMemberIds()
- OrigMembers – replaced by GetOriginalMemberIds()

##### CurtainGrid

- UnlockedMullions – replaced by GetUnlockedMullionIds()
- Mullions – replaced by GetMullionIds()
- Cells – replaced by GetCurtainCells()
- UnlockedPanels – replaced by GetUnlockedPanelIds()
- Panels – replaced by GetPanelIds()
- VGridLines – replaced by GetVGridLineIds()
- UGridLines – replaced by GetUGridLineIds()

##### CurveElement

- LineStyles – replaced by GetLineStyleIds()

##### Document

- Delete(ElementSet elements) – replaced by Delete(System.Collections.Generic.ICollection<Autodesk.Revit.DB.ElementId> elementIds)
- WorksharingCentralFilename – replaced by ModelPathUtils.ConvertModelPathToUserVisiblePath(Document.GetWorksharingCentralModelPath())
- PrintSettings – replaced by GetPrintSettingIds()
- Element/get\_Element – replaced by GetElement method

##### Element

- PhaseDemolished – replaced by DemolishedPhaseId
- PhaseCreated – replaced by CreatedPhaseId

##### FamilyInstance

- GetCopings() – replaced by GetCopingIds()
- SetCopings(ElementSet) – replaced by SetCopingIds(ICollection<ElementId> cutters)
- SubComponents – replaced by GetSubComponentIds()

##### Floor

- SpanDirectionSymbols – replaced by GetSpanDirectionSymbolIds()

##### GeometryElement

- Objects – replaced by GetEnumerator()

##### Group

- Ungroup() – replaced by UngroupMembers()
- Members – replaced by GetMemberIds()

##### LinearArray

- CopyMembers – replaced by GetCopiedMemberIds()
- OrigMembers – replaced by GetOriginalMemberIds()

##### Material

- GetMaterialAspectPropertySet(MaterialAspect) – replaced by GetStructuralAssetId() and GetThermalAssetId()
- SetMaterialAspect(MaterialAspect,ElementId,Boolean) – replaced by SetStructuralAssetId() and SetThermalAssetId()
- SetMaterialAspectToIndependent(MaterialAspect) – replaced by SetStructuralAssetId() and SetThermalAssetId()

##### MEPSystem

- Remove(ElementSet) – replaced by Remove(ICollection<ElementId>)

##### Part

- ParentDividedElementId – no replacement, concept is removed from Revit
- OriginalDividedElementId – no replacement, concept is removed from Revit
- GetDividedParents() – no replacement, concept is removed from Revit

##### PartMaker

- IsElementDivided(ElementId elemId) – replaced by IsSourceElement(ElementId)
- GetDividedElementIds() – replaced by GetSourceElementIds()
- SetDividedElementIds(ICollection<ElementId>) – replaced by SetSourceElementIds(ICollection<ElementId>)

##### PartUtils

- AreElementsValidForDivide(Document, ICollection<ElementId>) – replaced by ArePartsValidForDivide(Document, ICollection<ElementId>)
- AreElementsValidIntersectingReferences(Document, ICollection<ElementId>) – replaced by PartMakerMethodToDivideVolumes.AreElementsValidIntersectingReferences(Document, ICollection<ElementId>)
- IsValidSketchPlane(Document, ElementId) – replaced by PartMakerMethodToDivideVolumes.IsValidSketchPlane(Document, ElementId)
- SetOffsetForIntersectingReference() – replaced by PartMakerMethodToDivideVolumes.SetOffsetForIntersectingReference()
- GetOffsetForIntersectingReference() – replaced by PartMakerMethodToDivideVolumes.GetOffsetForIntersectingReference()
- PartMakerUsesReference() – replaced by PartMakerMethodToDivideVolumes.PartMakerUsesReference()
- IsMaxDivisionDepthReached(Document, ElementId) – no replacement, concept is removed from Revit
- GetDividedParents(Part) – no replacement, concept is removed from Revit
- PlanTopology
- Rooms – replaced by GetRoomIds()

##### PropertySetElement

- Create(Document, MaterialAspect) – replaced by Create(Document, StructuralAsset)

##### RadialArray

- CopyMembers – replaced by GetCopiedMemberIds()
- OrigMembers – replaced by GetOriginalMemberIds()

##### SpatialFieldManager

- UpdateSpatialFieldPrimitive(Int32,FieldDomainPoints,FieldValues) – replaced by UpdateSpatialFieldPrimitive(Int32, FieldDomainPoints, FieldValues, Int32)
- SetUnits(IList<string>, IList<double>) – replaced by AnalysisResultSchema.SetUnits() and SetResultSchema()

##### View

- ApplyTemplate(View viewTemplate) – replaced by ViewTemplateId/ApplyViewTemplateParameters(View viewTemplate)
- Hide(ElementSet elemSet) – replaced by HideElements(System::Collections::Generic::ICollection<Autodesk::Revit::DB::ElementId^>^ elementIdSet)
- Unhide(ElementSet elemSet) – replaced by UnhideElements(System::Collections::Generic::ICollection<Autodesk::Revit::DB::ElementId^>^ elementIdSet)

##### View3D

- EyePosition – replaced by ViewOrientation3D.EyePosition/View.Origin
- ViewSheet
- AddView(View,UV) – replaced by Viewport.Create(Document, ElementId, ElementId, XYZ)

#### Autodesk.Revit.DB.Plumbing namespace

##### PipeType

- Roughness – replaced by Segment.Roughness
- Material – replaced by PipeSegment.MaterialId

#### Autodesk.Revit.DB.Structure namespace

##### AnalyticalModel

- CanDisable() – no replacement, concept removed from Revit
- IsValidAnalyticalProjectionType(AnalyticalDirection,AnalyticalProjectionType) – replaced by IsValidProjectionType(AnalyticalElementSelector, AnalyticalDirection, AnalyticalProjectionType)

##### AreaReinforcement

- NumBarDescriptions – replaced by GetRebarInSystemIds()
- BarDescription – replaced GetRebarInSystemIds()
- Curves – replaced by GetCurveElementIds()

##### BeamSystem

- GetAllBeams() – replaced by GetBeamIds()

##### PathReinforcement

- BarDescription – replaced by GetRebarInSystemIds()
- Curves – replaced by GetCurveElementIds()

##### Rebar

- GetCenterlineCurves(Boolean) – replaced by GetCenterlineCurves(Boolean,Boolean,Boolean)
- DistributionPath – replaced by GetDistributionPath()
- RebarShape – replaced by RebarShapeId
- Host – replaced by Rebar.GetHostId() and SetHostId(Document, ElementId)
- BarType – replaced by Element.GetTypeId() and Element.ChangeTypeId(ElementId)

# Major Enhancements to the Revit API

## Worksharing API enhancements
---

## Related Sections
- [Major Changes and Renovations - What's New in the Revit 2014 API](./0938-01_whats_new_in_the_revit_2014_api.md)
- [Major Changes and Renovations - Document.Save()](./0938-02_documentsave.md)
- [Major Changes and Renovations - Document.SaveAs()](./0938-03_documentsaveas.md)
- [Major Changes and Renovations - OpenOptions](./0938-04_openoptions.md)
- [Major Changes and Renovations - Application.OpenDocumentFile(ModelPath, OpenOptions)](./0938-05_applicationopendocumentfilemodelpath_openoptions.md)
- [Major Changes and Renovations - Application.OpenDocumentFile(String)](./0938-06_applicationopendocumentfilestring.md)
- [Major Changes and Renovations - UIApplication.OpenAndActivateDocument(ModelPath, OpenOptionsForUI)](./0938-07_uiapplicationopenandactivatedocumentmodelpath_open.md)
- [Major Changes and Renovations - UIApplication.OpenAndActivateDocument(String)](./0938-08_uiapplicationopenandactivatedocumentstring.md)
- [Major Changes and Renovations - Iteration and element deletion](./0938-09_iteration_and_element_deletion.md)
- [Major Changes and Renovations - Curve creation](./0938-10_curve_creation.md)
- [Major Changes and Renovations - Curve utilities](./0938-11_curve_utilities.md)
- [Major Changes and Renovations - Edge utilities](./0938-12_edge_utilities.md)
- [Major Changes and Renovations - Transform initialization](./0938-13_transform_initialization.md)
- [Major Changes and Renovations - Unit Formatting and Parsing](./0938-14_unit_formatting_and_parsing.md)
- [Major Changes and Renovations - Unit Conversion](./0938-15_unit_conversion.md)
- [Major Changes and Renovations - Viewport.Create behavioral change](./0938-16_viewportcreate_behavioral_change.md)
- [Major Changes and Renovations - View.ViewName obsolete](./0938-17_viewviewname_obsolete.md)
- [Major Changes and Renovations - View.SetVisibility()](./0938-18_viewsetvisibility.md)
- [Major Changes and Renovations - ViewSchedule changes](./0938-19_viewschedule_changes.md)
- [Major Changes and Renovations - Applying visual materials](./0938-20_applying_visual_materials.md)
- [Major Changes and Renovations - AssetProperty changes](./0938-21_assetproperty_changes.md)
- [Major Changes and Renovations - External commands now supported from Project Browser as active view](./0938-22_external_commands_now_supported_from_project_brows.md)
- [Major Changes and Renovations - Start/End Extension & Cutback](./0938-23_startend_extension_cutback.md)
- [Major Changes and Renovations - Justifications](./0938-24_justifications.md)
- [Major Changes and Renovations - Divided Surface API](./0938-25_divided_surface_api.md)
- [Major Changes and Renovations - PointCloudType.Create()](./0938-26_pointcloudtypecreate.md)
- [Major Changes and Renovations - PointCloudInstance.GetPoints()](./0938-27_pointcloudinstancegetpoints.md)
- [Major Changes and Renovations - Point cloud overrides](./0938-28_point_cloud_overrides.md)
- [Major Changes and Renovations - Point cloud scans](./0938-29_point_cloud_scans.md)
- [Major Changes and Renovations - IFC export now External Service](./0938-30_ifc_export_now_external_service.md)
- [Major Changes and Renovations - IFC APIs moved to new assembly](./0938-31_ifc_apis_moved_to_new_assembly.md)
- [Major Changes and Renovations - PrintParameters](./0938-32_printparameters.md)
- [Major Changes and Renovations - Classes removed](./0938-33_classes_removed.md)
- [Major Changes and Renovations - Reload Latest](./0938-35_reload_latest.md)
- [Major Changes and Renovations - Synchronize with Central](./0938-36_synchronize_with_central.md)
- [Major Changes and Renovations - Element ownership](./0938-37_element_ownership.md)
- [Major Changes and Renovations - Create new local](./0938-38_create_new_local.md)
- [Major Changes and Renovations - Enable Worksharing](./0938-39_enable_worksharing.md)
- [Major Changes and Renovations - Identifying links](./0938-40_identifying_links.md)
- [Major Changes and Renovations - Obtaining linked documents](./0938-41_obtaining_linked_documents.md)
- [Major Changes and Renovations - Link creation](./0938-42_link_creation.md)
- [Major Changes and Renovations - Link load and unload](./0938-43_link_load_and_unload.md)
- [Major Changes and Renovations - Link path type](./0938-44_link_path_type.md)
- [Major Changes and Renovations - Conversion of geometric references](./0938-45_conversion_of_geometric_references.md)
- [Major Changes and Renovations - Room tag creation from linked rooms](./0938-46_room_tag_creation_from_linked_rooms.md)
- [Major Changes and Renovations - Picking in links](./0938-47_picking_in_links.md)
- [Major Changes and Renovations - Graphic Display options](./0938-48_graphic_display_options.md)
- [Major Changes and Renovations - Category classes override](./0938-49_category_classes_override.md)
- [Major Changes and Renovations - Category override](./0938-50_category_override.md)
- [Major Changes and Renovations - Element Override](./0938-51_element_override.md)
- [Major Changes and Renovations - View Filters](./0938-52_view_filters.md)
- [Major Changes and Renovations - Non-rectangular crop region](./0938-53_non_rectangular_crop_region.md)
- [Major Changes and Renovations - Viewport](./0938-54_viewport.md)
- [Major Changes and Renovations - Exploded Views](./0938-55_exploded_views.md)
- [Major Changes and Renovations - Revisions on sheets](./0938-56_revisions_on_sheets.md)
- [Major Changes and Renovations - UIView](./0938-57_uiview.md)
- [Major Changes and Renovations - PreviewControl](./0938-58_previewcontrol.md)
- [Major Changes and Renovations - Command API](./0938-59_command_api.md)
- [Major Changes and Renovations - Dockable Dialog Panes](./0938-60_dockable_dialog_panes.md)
- [Major Changes and Renovations - Multi-reference annotations for rebar](./0938-61_multi_reference_annotations_for_rebar.md)
- [Major Changes and Renovations - Dimension alternate units](./0938-62_dimension_alternate_units.md)
- [Major Changes and Renovations - Dimension unit type](./0938-63_dimension_unit_type.md)
- [Major Changes and Renovations - Automatic load of add-ins without restarting Revit](./0938-64_automatic_load_of_add_ins_without_restarting_revit.md)
- [Major Changes and Renovations - MacroManager API](./0938-65_macromanager_api.md)
- [Major Changes and Renovations - Macro Attributes](./0938-66_macro_attributes.md)
- [Major Changes and Renovations - Shared parameter – create with specified GUID](./0938-67_shared_parameter_create_with_specified_guid.md)
- [Major Changes and Renovations - Dimension.Label](./0938-68_dimensionlabel.md)
- [Major Changes and Renovations - Family Parameters](./0938-69_family_parameters.md)
- [Major Changes and Renovations - JoinGeometryUtils](./0938-70_joingeometryutils.md)
- [Major Changes and Renovations - ExtensibleStorage API changes](./0938-71_extensiblestorage_api_changes.md)
- [Major Changes and Renovations - ExtensibleStorageFilter](./0938-72_extensiblestoragefilter.md)
- [Major Changes and Renovations - Export to Navisworks](./0938-73_export_to_navisworks.md)
- [Major Changes and Renovations - Import/Link SAT](./0938-74_importlink_sat.md)
- [Major Changes and Renovations - Import/Link SketchUp](./0938-75_importlink_sketchup.md)
- [Major Changes and Renovations - Import DWF Markups](./0938-76_import_dwf_markups.md)
- [Major Changes and Renovations - Export tables](./0938-77_export_tables.md)
- [Major Changes and Renovations - Editing a TopographySurface](./0938-78_editing_a_topographysurface.md)
- [Major Changes and Renovations - Reading points from a TopographySurface](./0938-79_reading_points_from_a_topographysurface.md)
- [Major Changes and Renovations - Validation](./0938-80_validation.md)
- [Major Changes and Renovations - SiteSubRegion](./0938-81_sitesubregion.md)
- [Major Changes and Renovations - BuildingPad](./0938-82_buildingpad.md)
- [Major Changes and Renovations - Externalized Calculations](./0938-83_externalized_calculations.md)
- [Major Changes and Renovations - ElectricalLoadClassificationData](./0938-84_electricalloadclassificationdata.md)
- [Major Changes and Renovations - CSV Fitting Parameter Removal](./0938-85_csv_fitting_parameter_removal.md)
- [Major Changes and Renovations - Fitting Angle Settings](./0938-86_fitting_angle_settings.md)
- [Major Changes and Renovations - Duct Settings](./0938-87_duct_settings.md)
- [Major Changes and Renovations - Curve Creation](./0938-88_curve_creation.md)
- [Major Changes and Renovations - ConnectorElement](./0938-89_connectorelement.md)
- [Major Changes and Renovations - Connect Air Terminal to Duct](./0938-90_connect_air_terminal_to_duct.md)
- [Major Changes and Renovations - General](./0938-91_general.md)
- [Major Changes and Renovations - CustomExporter](./0938-92_customexporter.md)
- [Major Changes and Renovations - IExportContext](./0938-93_iexportcontext.md)
- [Major Changes and Renovations - Render Node Classes](./0938-94_render_node_classes.md)
- [Major Changes and Renovations - CameraInfo](./0938-95_camerainfo.md)
- [Major Changes and Renovations - No transactions from outside threads](./0938-96_no_transactions_from_outside_threads.md)
- [Major Changes and Renovations - IsValidObject property](./0938-97_isvalidobject_property.md)
- [Major Changes and Renovations - Enumerated type validation](./0938-98_enumerated_type_validation.md)
- [Major Changes and Renovations - Copy & paste elements](./0938-99_copy_paste_elements.md)
- [Major Changes and Renovations - Materials](./0938-100_materials.md)
- [Major Changes and Renovations - FreeForm element](./0938-101_freeform_element.md)
- [Major Changes and Renovations - Solid & curve intersection](./0938-102_solid_curve_intersection.md)
- [Major Changes and Renovations - Face/Face Intersection](./0938-103_faceface_intersection.md)
- [Major Changes and Renovations - ReferenceIntersector & RVT Links](./0938-104_referenceintersector_rvt_links.md)
- [Major Changes and Renovations - Rulings of RuledFace](./0938-105_rulings_of_ruledface.md)
- [Major Changes and Renovations - Detail element draw order](./0938-106_detail_element_draw_order.md)
- [Major Changes and Renovations - StairsRunJustification](./0938-107_stairsrunjustification.md)
- [Major Changes and Renovations - StairsLanding](./0938-108_stairslanding.md)
- [Major Changes and Renovations - StairsRun](./0938-109_stairsrun.md)
- [Major Changes and Renovations - StairsComponentConnection](./0938-110_stairscomponentconnection.md)
- [Major Changes and Renovations - Parameter.AsValueString()](./0938-111_parameterasvaluestring.md)
- [Major Changes and Renovations - Parameter.Definition.UnitType](./0938-112_parameterdefinitionunittype.md)
- [Major Changes and Renovations - Parameter variance among group instances](./0938-113_parameter_variance_among_group_instances.md)
- [Major Changes and Renovations - FilterCategoryRule](./0938-114_filtercategoryrule.md)
- [Major Changes and Renovations - ThermalAsset.SpecificHeat](./0938-115_thermalassetspecificheat.md)
- [Major Changes and Renovations - AreaVolumeSettings](./0938-116_areavolumesettings.md)
- [Major Changes and Renovations - Document.Delete()](./0938-117_documentdelete.md)
- [Major Changes and Renovations - Document level updaters](./0938-118_document_level_updaters.md)
- [Major Changes and Renovations - UIThemeManager](./0938-119_uithememanager.md)
- [Major Changes and Renovations - Family category](./0938-120_family_category.md)
- [Major Changes and Renovations - SpatialElementCalculationLocation](./0938-121_spatialelementcalculationlocation.md)
- [Major Changes and Renovations - SpatialElementFromToCalculationPoints](./0938-122_spatialelementfromtocalculationpoints.md)
- [Major Changes and Renovations - Arc through points](./0938-123_arc_through_points.md)
- [Major Changes and Renovations - DocumentChangedEventArgs](./0938-124_documentchangedeventargs.md)
- [Major Changes and Renovations - GetAddElementIds(ElementFilter)/GetModifiedElementIds(ElementFilter)](./0938-125_getaddelementidselementfiltergetmodifiedelementids.md)
- [Major Changes and Renovations - Reinforcement Length Tolerance](./0938-126_reinforcement_length_tolerance.md)
