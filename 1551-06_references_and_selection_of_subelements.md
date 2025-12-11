---
author: Jeremy Tammik
optimization_date: '2025-12-11T11:24:10.422874'
original_url: https://thebuildingcoder.typepad.com/blog/1551_whats_new_2018.html
parent_post: 1551_whats_new_2018.md
part_number: '06'
part_total: '23'
post_number: '1551'
series: geometry
slug: whats_new_2018_references_and_selection_of_subelements
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
title: API Changes - References and selection of subelements
word_count: 1400
---

### References and selection of subelements

The new enumerated value:

* ObjectType.Subelement

provides the ability to prompt a user to select subelements interactively using Selection.PickObject() or Selection.PickObjects().

The new enumerated value:

* ElementReferenceType.REFERENCE_TYPE_SUBELEMENT

identify a reference as a reference to a specific subelement.

The new methods:

* Reference.EqualTo()
* Reference.Contains()

provide useful checks related to the contents of a given Reference object, applicable to subelement references as well as other types of references.

## Changes to APIs for accessing version

The property:

* Application.IsSubscriptionUpdate

has been deprecated and replaced by:

* Application.SubVersionNumber

The new property returns a string representing the major-minor version number for the Revit application. For example, "2018.0.0". This number is updated for major and minor updates.

In RevitAddinUtility, the similar property:

* RevitProduct.IsSubscriptionUpdate

has been deprecated and replaced by:

* RevitProduct.ReleaseSubVersion

This new string property returns a similar major-minor version number for installed Revit versions.

## Asset API Changes

The following API classes moved from the Autodesk.Revit.Utility namespace to a new namespace Autodesk.Revit.DB.Visual:

* AssetProperty
* AssetProperties
* Asset
* AssetSet
* AssetType
* AssetPropertyType
* AssetPropertyDouble
* AssetPropertyDoubleArray2d
* AssetPropertyDoubleArray3d
* AssetPropertyDoubleArray4d
* AssetPropertyDoubleMatrix44
* AssetPropertyFloat
* AssetPropertyFloatArray
* AssetPropertyInt64
* AssetPropertyUInt64
* AssetPropertyBoolean
* AssetPropertyDistance
* AssetPropertyEnum
* AssetPropertyReference
* AssetPropertyString
* AssetPropertyTime
* AssetPropertyList

Two AssetProperty properties were deprecated and replaced:

* Deprecated → Replacement (new methods)
* AssetPropertyDouble3.Value → IList APropertyDouble3::GetValueAsDoubles() and XYZ APropertyDouble3::GetValueAsXYZ()
* AssetPropertyDouble4.Value → IList APropertyDouble4::GetValueAsDoubles()

Values for the enumerated type AssetPropertyType were renamed to better adhere to the API standards. Note that corresponding integer values are the same:

0. APT_Unknown → Unknown
1. APT_Properties → Properties
2. APT_Boolean → Boolean
3. APT_Enum → Enumeration
4. APT_Integer → Integer
5. APT_Float → Float
6. APT_Double → Double1
7. APT_DoubleArray2d → Double2
8. APT_DoubleArray3d → Double3
9. APT_DoubleArray4d → Double4
10. APT_Double44 → Double44
11. APT_String → String
12. APT_Time → Time
13. N/A
14. APT_Distance → Distance
15. APT_Asset → Asset
16. APT_Reference → Reference
17. APT_Int64 → Longlong
18. APT_UInt64 → ULonglong
19. APT_List → List
20. APT_FloatArray → Float3

## Dynamic Updaters on Reload Latest

Dynamic updaters are now triggered on Reload Latest for the elements added or changed in the central file.

## Export to DWG/DXF API change

The new AutoCAD version (R2018) has been added to the ACADVersion enumerated type. This is now the default version used when exporting to DWG and DXF.

## UIDocument.PromptForFamilyInstancePlacement() behavioral change

The behavior for UIDocument.PromptForFamilyInstancePlacement() was changed to be same as that of PickObject() methods – the placement operation will be cancelled when the "x" button of Revit is clicked during the placement operation instead of closing Revit directly.

## IndependentTag API changes

The new method:

* IndependentTag.Create()

replaces Autodesk.Revit.Creation.Document.NewTag(), which has been marked obsolete. The new version supports tagging of either elements or subelements.

The new method:

* IndependentTag.GetTaggedReference()

returns a reference to the item which has been tagged. This reference may be to a Subelement, which can also be identified by:

* IndependentTag.IsTaggedOnSubelement()

The following properties and their methods now throw more informative exceptions:

* IndependentTag.LeaderElbow:

+ IndependentTag.GetLeaderElbow() – "The tag does not have a leader or its leader is straight."
+ IndependentTag.SetLeaderElbow() – "The tag does not have a leader."

* IndependentTag.LeaderEnd:

+ IndependentTag.GetLeaderEnd() – "There is no leader end because the tag does not use a free end leader."
+ IndependentTag.SetLeaderEnd() – "There is no leader end because the tag does not use a free end leader."

The new property:

* IndependentTag.HasElbow

indicates if the leader of the tag has an elbow point or not.

## Shared Coordinates API changes

The following properties have been deprecated and replaced:

* Deprecated member → New/replacement member
* ProjectLocation.ProjectPosition → ProjectLocation.GetProjectPosition() and ProjectLocation.SetProjectPosition()
* ProjectLocation.SiteLocation → ProjectLocation.GetSiteLocation()

## New DirectShape behaviors

DirectShape elements now support new behaviors. Other than the limitations listed below, no code changes are required to enable these new behaviors for DirectShape elements.

* Tagging – if the element's assigned category supports tagging and a tag type exists, the DirectShape can be tagged with Revit tagging tools.
* Edge dimensions – if the DirectShape is referenceable, it will now support dimensioning to edge references as well as face references.
* Connector elements – in families, if the DirectShape is referenceable, DirectShape planar faces can be used to host connector elements.
* Rebar hosting – if the DirectShape is of any of the following categories it can now act as a host for rebar:

+ OST_StructuralFraming
+ OST_StructuralColumns
+ OST_StructuralFoundation
+ OST_StructuralConnections
+ OST_Walls
+ OST_Floors
+ OST_EdgeSlab
+ OST_Parts
+ OST_Stairs
+ OST_GenericModels

## Rebar API Changes

Rebar now supports two different layout options: shape-driven and free-form. Previously, all Rebar elements were shape-driven. The new methods:

* Rebar.GetShapeDrivenAccessor() – Returns the Shape Driven interface which exposes specific Shape Driven logic.
* Rebar.GetFreeFormAccessor() – Returns the Free Form interface which exposes specific Free Form logic.
* Rebar.IsRebarFreeForm – Returns true if the rebar is free form and false if shape driven.
* Rebar.IsRebarShapeDriven – Returns true if the rebar is shape driven and false if free form.

Rebar members which are applicable only for shape-driven rebar have been deprecated and replaced with equivalents in the class RebarShapeDrivenAccessor. Specifically, the following members are deprecated:

* Deprecated member → Replacement method
* Rebar.GetDistributionPath() → RebarShapeDrivenAccessor.GetDistributionPath()
* Rebar.ComputeDrivingCurves() → RebarShapeDrivenAccessor.ComputeDrivingCurves()
* Rebar.GetBarPositionTransform() → RebarShapeDrivenAccessor.GetBarPositionTransform()
* Rebar.ScaleToBox() → RebarShapeDrivenAccessor.ScaleToBox()
* Rebar.ScaleToBoxFor3D() → RebarShapeDrivenAccessor.ScaleToBoxFor3D()
* Rebar.SetLayoutAsSingle() → RebarShapeDrivenAccessor.SetLayoutAsSingle()
* Rebar.SetLayoutAsFixedNumber() → RebarShapeDrivenAccessor.SetLayoutAsFixedNumber()
* Rebar.SetLayoutAsMaximumSpacing() → RebarShapeDrivenAccessor.SetLayoutAsMaximumSpacing()
* Rebar.SetLayoutAsNumberWithSpacing() → RebarShapeDrivenAccessor.SetLayoutAsNumberWithSpacing ()
* Rebar.SetLayoutAsMinimumClearSpacing() → RebarShapeDrivenAccessor.SetLayoutAsMinimumClearSpacing()
* Rebar.Normal → RebarShapeDrivenAccessor.Normal
* Rebar.BarsOnNormalSide → RebarShapeDrivenAccessor.BarsOnNormalSide
* Rebar.Height → RebarShapeDrivenAccessor.Height
* Rebar.ArrayLength → RebarShapeDrivenAccessor.ArrayLength
* Rebar.BaseFinishingTurns → RebarShapeDrivenAccessor.BaseFinishingTurns
* Rebar.MultiplanarDepth → RebarShapeDrivenAccessor.MultiplanarDepth
* Rebar.TopFinishingTurns → RebarShapeDrivenAccessor.TopFinishingTurns
* Rebar.Pitch → RebarShapeDrivenAccessor.Pitch
* Rebar.RebarShapeId → getter – Rebar.GetShapeId() and setter – RebarShapeDrivenAccessor.SetRebarShapeId()

## FabricSheet API Changes

The following methods have been deprecated and replaced:

* Deprecated method → Replacement method – Notes
* FabricSheetType.SetLayoutAsCustomPattern(double, double , double , double , IList , IList) → FabricSheetType.SetLayoutAsCustomPattern(double, double , IList , IList) – Both end overhangs will now be read only and computed internally.
* FabricWireItem.Create(double distance, double wireLength, ElementId wireType) → FabricWireItem.Create(double distance, double wireLength, ElementId wireType, double wireOffset) – Older calls can use the new method with wireOffset set to 0.0.

## Structural Section API Changes

Several section properties were moved from subclasses to the base class StructuralSection:

* StructuralSection.ElasticModulusStrongAxis
* StructuralSection.ElasticModulusWeakAxis
* StructuralSection.MomentOfInertiaStrongAxis
* StructuralSection.MomentOfInertiaWeakAxis
* StructuralSection.NominalWeight
* StructuralSection.Perimeter
* StructuralSection.PlasticModulusStrongAxis
* StructuralSection.PlasticModulusWeakAxis
* StructuralSection.PrincipalAxesAngle
* StructuralSection.SectionArea
* StructuralSection.ShearAreaStrongAxis
* StructuralSection.ShearAreaWeakAxis
* StructuralSection.TorsionalModulus
* StructuralSection.TorsionalMomentOfInertia
* StructuralSection.WarpingConstant

Several specific structural sections offer new constructors with additional input parameters. Their original constructors have been deprecated.

* StructuralSectionCSlopedFlange
* StructuralSectionISlopedFlange
* StructuralSectionISplitSlopedFlange
* StructuralSectionLAngle
* StructuralSectionStructuralTees

Several new specific structural section classes have been introduced:

* StructuralSectionGeneralC – Defines parameters for Channel Cold Formed shape.
* StructuralSectionGeneralCEx – Defines parameters for Channel with Fold Cold Formed shape.
* StructuralSectionGeneralF – Defines parameters for Flat Bar.
* StructuralSectionGeneralH – Defines parameters for Rectangular Pipe structural section.
* StructuralSectionGeneralI – Defines parameters for general Double T shape.
* StructuralSectionGeneralLA – Defines parameters for Angle Cold Formed structural section.
* StructuralSectionGeneralLZ – Defines parameters for Z Cold Formed shape.
* StructuralSectionGeneralR – Defines parameters for pipes.
* StructuralSectionGeneralS – Defines parameters for Round Bar structural section.
* StructuralSectionGeneralT – Defines parameters for Tees shape.
* StructuralSectionGeneralU – Defines parameters for general Channel shape.
* StructuralSectionGeneralW – Defines parameters for Angle structural section.

In addition, the API for structural sections offers a few other new capabilities:

* StructuralSection.GetBoundarySize() – returns the size of the section boundary.
* StructuralSection.AnalysisParams – accesses a common set of parameters for structural analysis which can be associated to a section.
* StructuralSection.StructuralSectionGeneralShape – returns an enumerated value identifying the general shape for the structural section representing geometry only.
* StructuralSectionHotRolled.FlangeThicknessLocation – this new property has been introduced for this class and the specific sections that derive from it.
* StructuralSectionHotRolled.WebThicknessLocation – this new property has been introduced for this class and the specific sections that derive from it.
* StructuralSectionUtils.GetStructuralElementDefinitionData() – returns data defining the section and the position of the structural element.

## ElectricalSystem API changes

The following functions have been deprecated and replaced:

* Deprecated member → New/replacement member
* Autodesk.Revit.Creation.Document.NewElectricalSystem(Connector, ElectricalSystemType) → ElectricalSystem.Create(Connector, ElectricalSystemType)
* Autodesk.Revit.Creation.Document.NewElectricalSystem(ICollection < ElementId > , ElectricalSystemType) → ElectricalSystem.Create(Document, IList, ElectricalSystemType)

## Pipe Pressure Loss Calculation change

The methods:

* Autodesk.Revit.DB.Plumbing.PipeSettings.GetPressLossCalculationServerInfo()
* Autodesk.Revit.DB.Plumbing.PipeSettings.SetPressLossCalculationServerInfo()

have been deprecated in Revit 2018 and will be removed in the next version of Revit. Custom pipe pressure loss calculations will no longer be supported. Similar functionality can be accessed by setting Autodesk.Revit.DB.Plumbing.PipeSettings.AnalysisForClosedLoopHydronicPipingNetworks.

## Corrected names of AutoRouteFailures values

The following BuiltInFailures.AutoRouteFailures values were renamed due to spelling errors:

* AttemptToConnectNonSlopingElementToSlopedPipeWarning (renamed from AttemptToComnnectNonSlopingElementToSlopedPipeWarning)
* AttemptToConnectNonSlopingElementToSlopedPipeError (renamed from AttemptToComnnectNonSlopingElementToSlopedPipeError)

## Obsolete API removal

The following API members and classes which had previously been marked Obsolete have been removed in this release. Consult the API documentation from prior releases for information on the replacements to use:
---

## Related Sections
- [API Changes - What's New in the Revit 2018 API](./1551-01_whats_new_in_the_revit_2018_api.md)
- [API Changes - Table of Contents](./1551-02_table_of_contents.md)
- [API Changes - API Changes](./1551-03_api_changes.md)
- [API Changes - Element modifications and contextual commands](./1551-04_element_modifications_and_contextual_commands.md)
- [API Changes - External commands and applications](./1551-05_external_commands_and_applications.md)
- [API Changes - Classes](./1551-07_classes.md)
- [API Changes - Methods](./1551-08_methods.md)
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
