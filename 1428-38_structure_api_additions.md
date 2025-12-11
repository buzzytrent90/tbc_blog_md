---
author: Jeremy Tammik
optimization_date: '2025-12-11T11:24:11.698936'
original_url: https://thebuildingcoder.typepad.com/blog/1428_whats_new_2017.html
parent_post: 1428_whats_new_2017.md
part_number: '38'
part_total: '43'
post_number: '1428'
series: geometry
slug: whats_new_2017_structure_api_additions
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
title: API Changes - Structure API additions
word_count: 894
---

### Structure API additions
#### FabricSheet
The new property:
- FabricSheet.FabricNumber
returns the reinforcement numbering value for the fabric sheet element.
The new method:
- FabricSheet.GetSegmentParameterIdsAndLengths()
returns the set of parameter ID and length pairs that correspond to segments of a bent fabric sheet (like A, B, C, D etc.).
The new method:
- FabricSheet.SetSegmentLength()
sets the length of the bent fabric sheet segment (like A, B, C, D etc.)
#### Quantitative FabricSheet layout
The new enum value:
- FabricSheetLayoutPattern.QuantitativeSpacing
indicates a pattern containing multiple groups of wires with a specific spacing and diameter.
Several new methods have been added to support this layout pattern:
- FabricSheetType.SetLayoutAsCustomPattern() – Sets the major and minor layout patterns to Custom, and specifies the FabricWireItems and overhang distances to be used.
- FabricSheetType.IsCustom() – Determines whether the type is Custom Fabric Sheet.
- FabricSheettype.GetWireItem() – Gets the wire stored in the FabricSheet at the associated index.
The new class:
- Autodesk.Revit.DB.Structure.FabricWireItem
represents a single fabric wire.
It has the following methods and properties:
- FabricWireItem.Create()
- FabricWireItem.Distance – The distance to the next FabricWireItem.
- FabricWireItem.WireLength
- FabricWireItem.WireType
#### LoadCase API additions
The property:
- LoadCase.Number
can now be set.
The new method:
- LoadCase.isNumberUnique()
allows users to check if the proposed number is unique.
#### RebarContainer
The new members:
- RebarContainer.SetItemHiddenStatus()
- RebarContainer.IsItemHidden()
provide access to the option to hide an individual RebarContainerItem in the given view.
The new property:
- RebarContainer.PresentItemsAsSubelements
identifies if Items should be presented in schedules and tags as separate subelements.
#### Structural Connection API additions
Structural Connection API
The new class:
- Autodesk.Revit.DB.Structure.StructuralConnectionHandler
represents connections between structural elements. A StructuralConnectionHandler can connect structural walls, floors, foundations, framings, or columns.
Some methods and properties include:
- StructuralConnectionHandler.Create() – Creates a new instance of a StructuralConnectionHandler, which defines the connection between the given elements. The first element given is set as the primary one.
- StructuralConnectionHandler.GetConnectedElementIds()
- StructuralConnectionHandler.IsDetailed() – Checks if the StructuralConnectionHandler has the detailed connection type.
- StructuralConnectionHandler.ApprovalTypeId
- StructuralConnectionHandler.SingleElementEndIndex – The element end index for single element connections. 0 indicates the start and 1 the end.
The new class:
- Autodesk.Revit.DB.Structure.StructuralConnectionHandlerType
defines the type of a StructuralConnectionHandler.
The new class:
- Autodesk.Revit.DB.Structure.StructuralConnectionApprovalType
defines a type element which represents a connection approval type.
The new class:
- Autodesk.Revit.DB.Structure.StructuralConnectionSettings
provides access to project-wide structural connection settings. It contains the following methods and properties:
- static StructuralConnectionSettings.GetStructuralConnectionSettings()
- StructuralConnectionSettings.IncludeWarningControls – If set to true, a yellow triangle will be displayed with StructuralConnectionElements which have associated warnings.
#### Rebar Couplers
The new class:
- Autodesk.Revit.DB.Structure.RebarCoupler
represents a rebar coupler element.
RebarCoupler has the following methods and properties:
- static RebarCoupler.Create()
- RebarCoupler.CouplerLinkTwoBars() – Determines whether the coupler sits on two rebar or caps a single rebar.
- RebarCoupler.GetCoupledReinforcementData() – If the coupler connects two rebars, this method returns a list of two ReinforcementData. If it only connects one, there will be one ReinforcementData.
- RebarCoupler.GetPointsForPlacement() – Gets the point or points where the coupler is placed.
- RebarCoupler.GetCouplerPositionTransform() – Gets a transform representing the relative position of the coupler at index couplerPositionIndex in the set.
- RebarCoupler.GetCouplerQuantity() – Identifies the number of couplers in a set.
- RebarCoupler.CouplerMark
The new method:
- Rebar.GetCouplerId()
returns the id of the RebarCoupler applied to the rebar at the specified end.
The new enum value:
- Autodesk.Revit.DB.Structure.StructuralNumberingSchemas.RebarCoupler
indicates the built-in schema for numbering rebar coupler elements.
The new enum:
- Autodesk.Revit.DB.Structure.RevitCouplerError
contains various error states for a rebar coupler.
The new class:
- Autodesk.Revit.DB.Structure.RebarReinforcementData
contains information about rebar couplers.
RebarReinforcementData has the following methods:
- RebarReinforcementData.Create()
- RebarReinforcementData.GetId() – The id of the associated Rebar.
- RebarReinforcementData.SetId()
- RebarReinforcementData.GetEnd() – The end of the rebar to which a coupler is attached.
#### Rebar end treatments
The new class:
- Autodesk.Revit.DB.Structure.EndTreatmentType
represents an end treatment type for rebar.
It has the following methods:
- static EndTreatmentType.Create(Document)
- static EndTreatmentType.Create(Document, String) – Creates a new EndTreatmentType with the given name.
- static EndTreatmentType.CreateDefaultEndTreatmentType() – Creates a new EndTreatmentType with a default name.
- EndTreatmentType.GetEndTreatment() – The value of the END_TREATMENT parameter.
- EndTreatmentType.SetEndTreatment()
Several methods have been added to other classes to support end treatments:
- RebarShape.GetEndTreatmentTypeId() – The id of the end treatment type for the designated shape end.
- RebarShape.SetEndTreatmentTypeId()
- RebarShape.HasEndTreatment()
- Rebar.GetEndTreatmentTypeId()
The new enum value:
- ElementGroupType.EndTreatmentType
indicates an end treatment type.
The new property:
- ReinforcementSettings.RebarShapeDefinesEndTreatments
indicates whether end treatments are defined by the RebarShape of the Rebar element. This value can be changed if the document contains no rebars, area reinforcements, or path reinforcements.
Additionally, the following methods and properties have been modified to require that the RebarShape has no end treatments:
- RebarContainer.AppendItemFromRebar()
- RebarContainer.AppendItemFromRebarShape()
- RebarContainer.AppendItemFromCurvesAndShape()
- RebarContainerItem.SetFromRebar()
- RebarContainerItem.SetFromRebarShape()
- RebarContainerItem.SetFromCurvesAndShape()
- RebarContainerItem.RebarShapeId
#### Other Structure API additions
New properties have been added to ReinforcementSettings:
- ReinforcementSettings.NumberVaryingLengthRebarsIndividually – Modifies the way varying length bars are numbered (individually or as a whole).
- ReinforcementSettings.RebarVaryingLengthNumberSuffix – A unique identifier used for a bar within a variable length rebar set.
- RebarConstraintsManager.isRebarConstrainedPlacementEnabled – Enables/Disables the 'Rebar Constrained Placement' setting in the current Revit Application Session.
One property has been added to Rebar:
- Rebar.DistributionType – Modifies the type of a rebar set. Rebar sets can be Uniform or VaryingLength.
The new Rebar method:
- getParameterValueAtIndex(ElementId paramId, int barPositionIndex)
returns the ParameterValue at the given bar index inside a rebar set.
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
- [API Changes - Parameter API additions](./1428-32_parameter_api_additions.md)
- [API Changes - CurveElement API additions](./1428-33_curveelement_api_additions.md)
- [API Changes - Railing API additions](./1428-34_railing_api_additions.md)
- [API Changes - Schedule API additions](./1428-35_schedule_api_additions.md)
- [API Changes - Tag API additions](./1428-36_tag_api_additions.md)
- [API Changes - UI API additions](./1428-37_ui_api_additions.md)
- [API Changes - Fabrication API additions](./1428-39_fabrication_api_additions.md)
- [API Changes - Electrical API additions](./1428-40_electrical_api_additions.md)
- [API Changes - Other MEP API additions](./1428-41_other_mep_api_additions.md)
- [API Changes - Revit Link API additions](./1428-42_revit_link_api_additions.md)
- [API Changes - Category API additions](./1428-43_category_api_additions.md)
