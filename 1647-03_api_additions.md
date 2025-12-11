---
author: Jeremy Tammik
optimization_date: '2025-12-11T11:24:48.940973'
original_url: https://thebuildingcoder.typepad.com/blog/1647_whats_new_2019.html
parent_post: 1647_whats_new_2019.md
part_number: '03'
part_total: '03'
post_number: '1647'
series: geometry
slug: whats_new_2019_api_additions
source_file: 1647_whats_new_2019.md
tags:
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
- sheets
- transactions
- views
- walls
- windows
title: API Changes - API Additions
word_count: 2878
---

### API Additions

#### 2.1. Material API additions
#### 2.1.1. Editing the properties of an Appearance Asset
New Revit API capabilities have been introduced to edit the properties contained in an appearance asset of a material. These properties appear in the Appearance tab of the Materials dialog and govern the appearance of the material in realistic views and rendering.
Editing the properties in an appearance Asset requires establishment of an edit scope. The new class
- Autodesk.Revit.DB.Visual.AppearanceAssetEditScope
allows an application to create and maintain an editing session for an appearance asset. The scope provides access to an editable Asset object whose properties may be changed. Once all of the desired changes have been made to the asset's properties, the edit scope should be committed, which causes the changes to be sent back into the document. (This is the only part of the process when a transaction must be opened).
The new class has the following methods:
- AppearanceAssetEditScope.Start() – Starts the edit scope for the asset contained in a particular AppearanceAssetElement. The editable Asset is returned from this method.
- AppearanceAssetEditScope.Commit() – Finishes the edit scope: all changes made during the edit scope will be committed. Provides an option to forces the update of all open views.
- AppearanceAssetEditScope.Cancel() – Cancels the edit scope and discards any changes.
#### 2.1.2. Editing AssetProperty values
The following properties are now writeable from within an AppearanceAssetEditScope, to support modification of an asset property's stored value:
- AssetPropertyString.Value
- AssetPropertyBoolean.Value
- AssetPropertyInteger.Value
- AssetPropertyDouble.Value
- AssetPropertyFloat.Value
- AssetPropertyEnum.Value
- AssetPropertyDistance.Value
In addition, the following new methods have been added to support modification of property values:
- AssetPropertyDoubleArray3d.SetValueAsXYZ()
- AssetPropertyDoubleArray4d.SetValueAsDoubles()
- AssetPropertyDoubleArray4d.SetValueAsColor()
AssetPropertyList now has new methods to allow changes to the members of the list:
- AssetPropertyList.AddNewAssetPropertyDouble()
- AssetPropertyList.InsertNewAssetPropertyDouble()
- AssetPropertyList.AddNewAssetAsColor()
- AssetPropertyList.InsertNewAssetAsColor()
- AssetPropertyList.AddNewAssetPropertyInteger()
- AssetPropertyList.InsertNewAssetPropertyInteger()
- AssetPropertyList.RemoveAssetProperty()
#### 2.1.3. Connected Assets
Connected assets are associated to properties in appearance assets, and represent subordinate objects encapsulating a collection of related properties. One example of a connected asset is the "Unified Bitmap" representing an image and its mapping parameters and values. AssetProperty offers new methods to provide the ability to modify, add or delete the asset connected to a property:
- AssetProperty.GetSingleConnectedAsset() – Gets the single connected asset of this property.
- AssetProperty.RemoveConnectedAsset() – Removes the single connected asset of this property.
- AssetProperty.AddConnectedAsset (String schemaId) – Create a new default asset of schema type and connects it to this property.
- AssetProperty.AddCopyAsConnectedAsset() – Connects the property to a copy of the asset.
#### 2.1.4. Validation
Inputs to change the value of asset properties are validated against the requirements of the associated schema.
The new methods:
- AssetPropertyString.IsValidValue(String)
- AssetPropertyInteger.IsValidValue (int)
- AssetPropertyEnum.IsValidValue (int)
- AssetPropertyDouble.IsValidValue (double)
- AssetPropertyFloat.IsValidValue (float)
- AssetPropertyDistance.IsValidValue (double)
- AssetPropertyDoubleArray3d.IsValidValue (XYZ)
- AssetPropertyDoubleArray4d.IsValidValue (IList)
- AssetPropertyDoubleArray4d.IsValidValue (Color)
identify if the input value is a valid value that can be set to the given asset property.
The new method:
- AssetProperty.IsEditable()
identifies the AssetProperty can currently be edited.
#### 2.1.5. Schemas & Property names
Appearance asset properties are aligned with specific schemas. Each schema contains necessary properties which define how the appearance of the material will be generated. There are 14 standard material schemas:
- Ceramic
- Concrete
- Generic
- Glazing
- Hardwood
- MasonryCMU
- Metal
- MetallicPaint
- Mirror
- PlasticVinyl
- SolidGlass
- Stone
- WallPaint
- Water
In addition, there are 5 schemas representing "advanced" materials – these may be encountered as a result of import from other Autodesk products:
- AdvancedLayered
- AdvancedMetal
- AdvancedOpaque
- AdvancedTransparent
- AdvancedWood
Finally, there are 10 schemas used for the aspects of the connected assets:
- BumpMap
- Checker
- Gradient
- Marble
- Noise
- Speckle
- Tile
- UnifiedBitmap
- Wave
- Wood
The new method:
- AssetProperty.IsValidSchemaIdentifier(String schemaName)
identifies if the input name is a valid name for a supported schema.
To assist in creating code accessing and manipulating the properties of a given schema, predefined properties have been introduced to allow a compile-time reference to a property name without requiring you to transcribe it as a string in your code. These predefined property names are available in static classes named similar to the schema names, above, e.g. Autodesk.Revit.DB.Visual.Ceramic.
#### 2.1.6. Utilities
The new method:
- Application.GetAssets(AssetType)
returns a list of assets available to the session.
The new method:
- AppearanceAssetElement.Duplicate()
creates a copy of an appearance asset element and the asset contained by it.
The new operator:
- Asset.operator[]
accesses a particular AssetProperty associated to the given asset.
#### 2.2. Railing API additions
#### 2.2.1. Non-Continuous Rail Structure
The new property:
- RailingType.RailStructure
provides access to a collection of the information about non-continuous rails that are part of a RailingType. The Autodesk.Revit.DB.Architecture.NonContinuousRailStructure object returns contains a collection of Autodesk.Revit.DB.Architecture.NonContinuousRailInfo objects, each of which represents the properties needed to define a single non-continuous rail.
#### 2.2.2. Baluster Placement
The new property:
- RailingType.BalusterPlacement
provides access to the baluster and post placement information for a given railing type. The Autodesk.Revit.DB.Architecture.BalusterPlacement object returned may contain instances of:
- Autodesk.Revit.DB.Architecture.BalusterPattern – represents the baluster pattern properties, containing 1 or more objects of the type BalusterInfo.
- Autodesk.Revit.DB.Architecture.PostPattern – represents the post pattern properties, containing up to 3 objects of the type BalusterInfo
- Autodesk.Revit.DB.Architecture.BalusterInfo – represents the properties governing an instance of a single railing baluster or post.
#### 2.3. Find Element Dependencies
The new method:
- Element.GetDependentElements()
returns a list of ids of elements which are "children" of this element; that is, those elements which will be deleted along with this element. The method optionally takes an ElementFilter which can be used to reduce the output list to the collection of elements matching specific criteria.
#### 2.4. Dimension API additions
The dimension API now supports adding linear dimensions to cut geometry which is view-specific.
The new property:
- Dimension.AreReferencesAvailable
always returns true for dimensions which are not view-specific. It can return false for view-specific dimensions that can lose their references in certain situations. For example, the host element references may not be available when the view containing the dimension is closed. In general, if the host element view-specific geometry is not available, dimensions that reference that geometry will not be able to resolve their references.
The new property:
- Dimension.IsValid
always returns true for dimensions which are not view-specific. It can return false for view-specific dimensions that are hidden because they are in an invalid state. An example of an invalid state is having misaligned references for generation of an aligned dimension.
#### 2.5. Set vertical alignment for text
The new property:
- Autodesk.Revit.DB.TextNoteOptions.VerticalAlignment
allows you to specify the vertical alignment for a new text note created by TextNote.Create().
#### 2.6. BrowserOrganization API additions
The new enumerated value:
- BrowserOrganizationType.Schedules
corresponds to the type of the browser organization definition for schedules.
The new method:
- BrowserOrganization.GetCurrentBrowserOrganizationForSchedules()
gets the BrowserOrganization that applies to the Schedules section of the project browser.
#### 2.7. Rebar API additions
#### 2.7.1. BarTypeDiameterOptions
The new options class:
- BarTypeDiameterOptions
allows creation of a new set of diameter values for a RebarBarType. It can be used when copying the diameter information as a bulk of data from one RebarBarType to another.
The diameter options can be set for a RebarBarType with the new method:
- Autodesk.Revit.DB.Structure.RebarBarType.SetBarTypeDiameters()
which sets all input diameters from the input BarTypeDiameterOptions in the current RebarBarType.
#### 2.7.2. GetDistributionPath()
The new method:
- Autodesk.Revit.DB.Structure.RebarHandlePositionData.GetDistributionPath()
gets the distribution path currently stored in the rebar.
For a free form rebar set the distance between two consecutive bars may be different if it is calculated between different points on bars. The distribution path is an array of curves with the property that based on these curves the set was calculated to respect the layout rule and number of bars or spacing.
#### 2.7.3. RebarUpdateCurvesData
Several new properties have been added to RebarUpdateCurvesData:
- Autodesk.Revit.DB.Structure.RebarUpdateCurvesData.HostMirrored – If true, then host of the rebar was mirrored (along with the rebar) before the most recent regeneration.
- Autodesk.Revit.DB.Structure.RebarUpdateCurvesData.IsReversed – Used to store the state of the bar referring to the direction of the bars. This is useful when using face intersection to calculate bars. After mirroring, curves created from intersecting faces may be reversed, so we use this to store the state and keep the rebar pointing in the correct direction.
- Autodesk.Revit.DB.Structure.RebarUpdateCurvesData.ErrorMessage – The reason for calculation failure. If the calculation fails, this message will be shown in an error, or warning if we are editing the constraints.
#### 2.7.4. RebarShapes
Two new methods in the Rebar class check or get associated RebarShape objects:
- Autodesk.Revit.DB.Structure.Rebar.GetAllRebarShapeIds() – Gets the ids of the RebarShape elements which define the shape of this Rebar.
- Autodesk.Revit.DB.Structure.Rebar.CanBeMatchedWithMultipleShapes() – Determines whether this Rebar can be matched with more than one RebarShape.
#### 2.7.5. Workshop Instructions
Several new properties identify the workshop instructions of a Rebar:
- Autodesk.Revit.DB.Structure.RebarFreeFormAccessor.WorkshopInstructions – Gets or sets the workshop instructions for the current Rebar element (in other words, is it bent or straight.)
- Autodesk.Revit.DB.Structure.RebarUpdateCurvesData.WorkshopInstructions – Gets the workshop instructions for this Rebar. This property is read-only.
- Autodesk.Revit.DB.Structure.RebarUpdateCurvesData.AreWorkshopInstructionsChanged – Determines if the workshop instructions have changed since the last regeneration.
#### 2.7.6. RebarFreeFormAccessor Additions
Several new methods have been added to determine information about the bar at a specific index of the set:
- Autodesk.Revit.DB.Structure.RebarFreeFormAccessor.GetShapeIdAtIndex() – Returns the shape id for the bar at the given position index.
- Autodesk.Revit.DB.Structure.RebarFreeFormAccessor.IsBarMatchedWithShapeInReverseOrder() – Checks if the bar at the given index is matched in reverse order with its shape.
- Autodesk.Revit.DB.Structure.RebarFreeFormAccessor.GetHookTypeIdAtIndex()
- Autodesk.Revit.DB.Structure.RebarFreeFormAccessor.GetHookOrientationAngleAtIndex()
- Autodesk.Revit.DB.Structure.RebarFreeFormAccessor.GetCouplerIdAtIndex()
- Autodesk.Revit.DB.Structure.RebarFreeFormAccessor.GetEndTreatmentTypeIdAtIndex()
#### 2.8. Steel Fabrication API additions
#### 2.8.1. SteelElementProperties – linking between Revit elements and steel fabrication elements
The new class:
- Autodesk.Revit.DB.Steel.SteelElementProperties
is used to attach steel fabrication information to various Revit elements. Elements which can have fabrication information include:
- FamilyInstance elements (structural beams and columns)
- StructuralConnectionHandler elements associated to the connection
- Specific steel connection elements (bolts, anchors, plates, etc). These connection elements will be of type Element but with categories related to structural connections, for example:
- OST_StructConnectionWelds
- OST_StructConnectionHoles
- OST_StructConnectionShearStuds
- OST_StructConnectionBolts
- OST_StructConnectionAnchors
- OST_StructConnectionPlates
- Some concrete elements (walls, floors, concrete beams) when they are used as input elements to creation of steel connections.
Use the static method:
- SteelElementProperties.GetSteelElementProperties()
to obtain the properties if they exist.
The properties contain:
- SteelElementProperties.UniqueID
which is the id of the object in fabrication terms, and can be used to determine the Steel Core element corresponding to this Revit element, for use with the Advance Steel API.
You can also look up the id about a Revit element using the static method:
- SteelElementProperties.GetFabricationUniqueID()
For Revit elements which do not currently have a fabrication link, it can be added using:
- SteelElementProperties.AddFabricationInformationForRevitElements()
If you have a fabrication id, you can look up the corresponding Revit element using:
- SteelElementProperties.GetReference()
This may return a reference to an element or a subelement.
#### 2.8.2. Custom steel connections – API additions
The new method:
- StructuralConnectionHandler.Create(Document, IList, String)
creates a custom StructuralConnectionHandler along with its associated StructuralConnectionHandlerType. Input elements should include structural members and steel connection members, and at least one StructuralConnectionHandler representing the generic connection to replace with the new detailed custom connection.
The methods:
- StructuralConnectionHandlerType.AddElementsToCustomConnection()
- StructuralConnectionHandlerType.RemoveMainSubelementsFromCustomConnection()
provide support for adding or removing steel connection elements in a custom connection.
The new properties:
- StructuralConnectionHandler.IsCustom
- StructuralConnectionHandlerType.IsCustom
- StructuralConnectionHandlerType.IsDetailed
provide read access to information about the structural connection handler elements.
#### 2.9. MEP API additions
#### 2.9.1. MEPCurveType Shape
The new property:
- MEPCurveType.Shape
returns the shape of the profile for the MEP curve type.
#### 2.9.2. Mechanical Equipment Set API
Two new classes allow creation and manipulation of a mechanical equipment set, which is a set of interrelated mechanical equipment in an MEP system that may operate in parallel. This release is limited to supporting one or more pump sets that are part of a piping system. The mechanical equipment set settings will be used for the MEP flow and pressure drop calculations.
The Element subclass Autodesk.Revit.DB.Mechanical.MechanicalEquipmentSet represents an instance of a set associated to an MEP system.
It includes the new members:
- MechanicalEquipmentSet.Create()
- MechanicalEquipmentSet.GetMembers()
- MechanicalEquipmentSet.Add()
- MechanicalEquipmentSet.Remove()
- MechanicalEquipmentSet.AreValidMembers()
- MechanicalEquipmentSet.AreElementsNotConnectedInSeries()
- MechanicalEquipmentSet.Classification
- MechanicalEquipmentSet.OnDuty
- MechanicalEquipmentSet.OnStandby
The ElementType subclass Autodesk.Revit.DB.Mechanical.MechanicalEquipmentSetType represents a type for a set of mechanical equipment in a MEP system. It includes:
- MechanicalEquipmentSetType.Create()
used to create a uniquely named type of mechanical equipment set.
For pump sets, the new method:
- PipingSystem.GetPumpSets()
gets the set of element ids of pump sets associated with this system.
#### 2.9.3. Hydraulic Separation API
New methods implemented in the PipingSystem class support hydraulic separation. Hydraulically separated systems allow independent flow and pressure analysis for each hydraulic loop. For example, each hydraulic loop has its own critical path. The calculated pressure drop on the primary pump consists of all pressure drop on the primary critical path. Any pressure drop on the secondary loop would only contribute to the calculated pressure drop of the secondary pump.
The new methods:
- PipingSystem.CreateHydraulicSeparation()
- PipingSystem.DeleteHydraulicSeparation()
- PipingSystem.IsHydraulicLoopBoundary()
- PipingSystem.CanBeHydraulicLoopBoundary()
support creation, deletion, and validation of members of a hydraulically separated system. Separated systems will have their own associated system elements from which you can read and modify the members after creation.
#### 2.10. MEP Fabrication API additions
#### 2.10.1. Exporting fabrication job files
The new method:
- FabricationPart.SaveAsFabricationJob()
writes a fabrication job to disk in the MAJ file format. The exported file will contain the fabrication parts included in the input. It takes an options class, FabricationSaveJobOptions, allowing for the possibility of including holes branches meet the main straight.
#### 2.10.2. API for load and unload of one-off fabrication parts from loose item files
The new class:
- Autodesk.Revit.DB.FabricationItemFile
contains information about one-off items that can be loaded into a fabrication configuration.
The new class:
- Autodesk.Revit.DB.FabricationItemFolder
may contain nested FabricationItemFolders and a list of FabricationItemFiles.
The new members:
- FabricationConfiguration.LoadItemFiles()
- FabricationConfiguration.UnloadItemFiles()
- FabricationConfiguration.GetAllLoadedItemFiles()
- FabricationConfiguration.GetAllUsedItemFiles()
- FabricationConfiguration.CanUnloadItemFiles()
- FabricationConfiguration.AreItemFilesLoaded()
- FabricationConfiguration.GetItemFolders()
allow control over the loading and unloading of item files into the configuration.
The new method:
- FabricationPart.Create(Document, FabricationItemFile, ElementId)
creates a FabricationPart from an item file.
#### 2.10.3. API for version history of parts
The new class:
- Autodesk.Revit.DB.FabricationVersionInfo
gives the information about different versions of fabrication data, including the reason why the data was changed.
The new method:
- FabricationPart.GetVersionHistory()
returns a list of FabricationVersionInfo classes that describe the history of the changes made to the part. The most recent changes are first in the list.
#### 2.10.4. API for part swap out information
The new class:
- Autodesk.Revit.DB.ReloadSwapOutInfo
gives information about a part that was swapped out during reload.
The new members:
- ConfigurationReloadInfo.OutOfDatePartCount
- ConfigurationReloadInfo.GetOutOfDatePartStatus()
identify the parts that had newer versions found during a reload and which Revit attempted to swap out.
#### 2.10.5. Centerline length API
The new property:
- FabricationPart.CenterlineLength
returns the length of the fabrication part's centerline.
#### 2.11. Analysis Visualization Framework API addition
The new property:
- AnalysisDisplayColoredSurfaceSettings.Transparency
allows for the use of transparency in the display of the colored surface.
#### 2.12. IFC additions
The new functions:
- IFCImportOptions.GetExtraOptions()
- IFCImportOptions.SetExtraOptions()
allow for passing in arbitrary options for custom IFC importers. Users can pass in a string to string map specifying extra data they wish to pass for IFC import.
#### 2.13. Revit Link Additions
The new function:
- RevitLinkType.GetPhaseMap()
allows users to read the Phase Map parameter for the link. The phase map is a correspondence between phases in the host document and phases in the linked document, and is used by room calculations. Note that the link must be loaded to read the phase map.
#### 2.14. Export Additions
The new property:
- DWFExportOptions.ExportOnlyViewId
indicates that only the specified view should be exported when creating a DWF export.
#### 2.15. Access to add-in data paths
The new properties:
- Autodesk.Revit.ApplicationServices.Application.CurrentUsersDataFolderPath
- Autodesk.Revit.ApplicationServices.Application.CurrentUsersAddinsDataFolderPath
provide access to the data folder and add-in data folder for the current Revit version and current user.
#### 2.16. Site API additions
The new property:
- TopographySurface.ArePointsEditable
identifies whether the points of this TopographySurface can be edited independently. This property is used to check validity for several TopographySurface editing methods.
#### 2.17. UI API additions
#### 2.17.1. BeforeExecutedEventArgs Cancellation
The property:
- BeforeExecutedEventArgs.Cancel
is now permitted to be set to true. This allows the call-back to cancel the execution of the impending command. This can be useful if an add-in wishes to selectively allow the command to proceed based on interactions which take place in this call-back. The property should also report the cancellation status if it has been cancelled by a different subscriber.
---

## Related Sections
- [API Changes - What's New in the Revit 2019 API](./1647-01_whats_new_in_the_revit_2019_api.md)
- [API Changes - API Changes](./1647-02_api_changes.md)
