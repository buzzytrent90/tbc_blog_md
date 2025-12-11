---
author: Jeremy Tammik
optimization_date: '2025-12-11T11:24:48.989288'
original_url: https://thebuildingcoder.typepad.com/blog/1676_whats_new_2019_1.html
parent_post: 1676_whats_new_2019_1.md
part_number: '02'
part_total: '03'
post_number: '1676'
series: geometry
slug: whats_new_2019_1_api_changes
source_file: 1676_whats_new_2019_1.md
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
title: API Changes - API Changes
word_count: 1721
---

### API Changes

#### .NET 4.7
Revit's API assemblies are built using .NET 4.7. At a minimum, add-ons will need to target .NET 4.7 for Revit 2019.
#### View Filters support OR operators and nesting
#### ParameterFilterElement API changes
View filters now support multiple nested levels of combinations of criteria joined with either "AND" or "OR". In the API, an ElementFilter hierarchy is replacing the use of lists of FilterRules previously obtained from ParameterFilterElement. Several methods are deprecated and replaced. Note that the deprecated methods will still function as before when accessing the criteria contained within a filter joined by a single AND, but cannot function as expected when accessing more complicated sets of criteria. The new method:
- ParameterFilterElement.UsesConjunctionOfFilterRules()
can be used to check if a ParameterFilterElement uses a conjunction of filter rules (without any logical OR operations), as was always the case before Revit release 2019. This function is also deprecated, as are all functions related to the use of FilterRules in ParameterFilterElement.
- Deprecated function → Replacement function – Notes
- ParameterFilterElement.Create(Document, String, ICollection, IList) → ParameterFilterElement.Create(Document, String, ICollection, ElementFilter) – The ElementFilter input now specifies the filtering rules. This allows combinations of filter rules using logical AND and OR operations. The ElementFilter must be either an ElementParameterFilter or an ElementLogicalFilter containing only ElementParameterFilters and other ElementLogicalFilters.
- ParameterFilterElement.GetRules() → ParameterFilterElement.GetElementFilter() – The new function returns an ElementFilter representing the combination of filter rules used by the ParameterFilterElement. Note that logical combinations using both AND and OR operations are possible. GetRules() is applicable only to filters with rules conjoined with a single AND operation.
- ParameterFilterElement.SetRules() → ParameterFilterElement.SetElementFilter() – The new function sets the ParameterFilterElement to contain an ElementFilter representing the combination of filter rules.
- ParameterFilterElement.AllRuleParametersApplicable (IList) → ParameterFilterElement.AllRuleParametersApplicable (ElementFilter) – The new function checks that the parameters of the rules used by the given ElementFilter are valid for this filter's categories.
- ParameterFilterElement.AllRuleParametersApplicable (Document, ICollection, IList) → ParameterFilterElement.AllRuleParametersApplicable (Document, ICollection, ElementFilter) – The new function checks that the parameters of the given ElementFilter (representing a combination of rules) are valid for the given set of categories.
- ParameterFilterElement.GetRuleParameter(FilterRule) → FilterRule.GetRuleParameter()
- ParameterFilterElement.GetRuleParameters() → ParameterFilterElement.GetElementFilterParameters()
The new function:
- ParameterFilterElement.ElementFilterIsAcceptableForParameterFilterElement()
can be used to check if an ElementFilter is suitable to be used for a view filter (it consists only of logical filters joining ElementParameterFilters in combination).
#### ElementLogicalFilter API addition
The new function:
- ElementLogicalFilter.GetFilters()
returns the set of ElementFilters that are combined by the given ElementLogicalFilter. This is useful to read the contents of an ElementFilter obtained from a ParameterFilterElement.
#### View.ViewName deprecated
The property
- DB.View.ViewName
has been deprecated. It was a duplication of Element.Name and this property should be used in its place.
#### UI API changes
#### Main window handle access
Two new properties allow access to the handle of the Revit main window:
- Autodesk.Revit.UI.UIApplication.MainWindowHandle
- Autodesk.Revit.UI.UIControlledApplication.MainWindowHandle
This handle should be used when displaying modal dialogs and message windows to ensure that they are properly parented. Use these properties instead of System.Diagnostics.Process.GetCurrentProcess().MainWindowHandle, which is no longer a reliable method for retrieving the main window handle starting with Revit 2019.
#### Changes to PostableCommand enumeration
Some commands were renamed or removed as a result of changes to Revit's windowing system:
- Deprecated Name → Replacement/New Name – Notes
- PostableCommand.ReplicateWindow → N/A – This command is no longer available
- PostableCommand.CascadeWindows → PostableCommand.TabViews
- PostableCommand.TileWindows → PostableCommand.TileViews
- PostableCommand.CloseHiddenWindows → PostableCommand.CloseInactiveViews
#### Existing APIs now support open from cloud paths (Collaboration for Revit)
#### Document Open APIs support cloud paths
The existing methods:
- Application.OpenDocumentFile(ModelPath modelPath, OpenOptions openOptions)
- UIApplication.OpenAndActivateDocument(ModelPath modelPath, OpenOptions openOptions, bool detachAndPrompt)
and the new methods:
- Application.OpenDocumentFile(ModelPath modelPath,OpenOptions openOptions, IOpenFromCloudCallback openFromCloudCallback)
- UIApplication.OpenAndActivateDocument(ModelPath modelPath, OpenOptions openOptions, bool detachAndPrompt, IOpenFromCloudCallback openFromCloudCallback)
now offer the ability to open a C4R model from its location on the cloud. Obtain a relevant ModelPath representing the document from:
- ModelPathUtils.ConvertCloudGUIDsToCloudPath(Guid, Guid)
by inputting the project Guid and model Guid (which could be obtained from various Forge APIs).
#### Callback for conflict cases when opening from a cloud path
The callback method:
- Autodesk.Revit.DB.IOpenFromCloudCallback.OnOpenConflict()
can be passed to the document open methods to gain to handle conflict cases. The method is passed an OpenConflictScenario value identifying the reason for the conflict ( Rollback, Relinquished or OutOfDate) and should return an OpenConflictResult with the desired response (keep local changes, discard local changes and open the latest version or cancel).
The new class:
- DefaultOpenFromCloudCallback
provides a default way to handle conflicts: it always discards the local change and gets the latest version from the cloud.
#### BasicFileInfo changes
The structure of BasicFileInfo storage has been changed. As a result, calling:
- BasicFileInfo.Extract()
from a version of Revit prior to Revit 2019 will result in an exception if it is called on a Revit 2019 or later file. The class and method documentation has been clarified to reflect this possibility that may occur again in the future.
The following BasicFileInfo property was deprecated and replaced:
- Deprecated property → Replacement/new property – Notes
- SavedInVersion → Format – The new property contains only the file format indicator (the major release version such as "2019") used for saving the file. No build information or product information is included in this string, making it easier to use for comparisons or messaging.
Calling BasicFileInfo.Extract() from Revit 2019 on any Revit file version should work as expected. Both SavedInVersion and Format should be populated for old and new files.
#### Asset-related API members deprecated
As a part of the introduction of new Appearance Asset editing API's, some members were deprecated and replaced:
- Deprecated member → Replacement method
- Autodesk.Revit.DB.Visual.AssetProperties[String] → Autodesk.Revit.DB.Visual.AssetProperties.FindByName(String)
- Autodesk.Revit.DB.Visual.AssetProperties[int] → Autodesk.Revit.DB.Visual.AssetProperties.Get(int)
- Autodesk.Revit.DB.Visual.Asset.GetConnectedProperty(String) → Autodesk.Revit.DB.Visual.Asset.GetSingleConnectedAsset()
- Autodesk.Revit.DB.Visual.Asset.GetConnectedPropertiesNames() → Autodesk.Revit.DB.Visual.Asset.GetSingleConnectedAsset()
#### Double Pattern Support
#### Material API support for double patterns
With the introduction of double patterns in materials Revit now distinguishes between a background and a foreground pattern. The existing properties for the patterns and colors of the surface and cut faces have been deprecated. New properties have been introduced for the background and foreground patterns and colors for both surface and cut faces. (The deprecated properties will return the same values as the new foreground versions.)
- Deprecated property → Replacement/new properties
- Material.SurfacePatternColor → Material.SurfaceForegroundPatternColor
- Material.SurfaceBackgroundPatternColor → Material.SurfacePatternId
- Material.SurfaceForegroundPatternId → Material.SurfaceBackgroundPatternId
- Material.CutPatternColor → Material.CutForegroundPatternColor
- Material.CutBackgroundPatternColor → Material.CutPatternId
- Material.CutForegroundPatternId → Material.CutBackgroundPatternId
#### Support for overriding double patterns
Several methods and properties in the OverrideGraphicSettings class have been replaced with versions specific to the foreground or background pattern:
- Deprecated method/property → Replacement/new property/method
- OverrideGraphicSettings.SetProjectionFillColor → OverrideGraphicSettings.SetSurfaceForegroundPatternColor + OverrideGraphicSettings.SetSurfaceBackgroundPatternColor
- OverrideGraphicSettings.ProjectionFillColor → OverrideGraphicSettings.SurfaceForegroundPatternColor + OverrideGraphicSettings.SurfaceBackgroundPatternColor
- OverrideGraphicSettings.SetProjectionFillPatternId → OverrideGraphicSettings.SetSurfaceForegroundPatternId + OverrideGraphicSettings.SetSurfaceBackgroundPatternId
- OverrideGraphicSettings.ProjectionFillPatternId → OverrideGraphicSettings.SurfaceForegroundPatternId + OverrideGraphicSettings.SurfaceBackgroundPatternId
- OverrideGraphicSettings.SetProjectionFillPatternVisible → OverrideGraphicSettings.SetSurfaceForegroundPatternVisible + OverrideGraphicSettings.SetSurfaceBackgroundPatternVisible
- OverrideGraphicSettings.IsProjectionFillPatternVisible → OverrideGraphicSettings.IsSurfaceForegroundPatternVisible + OverrideGraphicSettings.IsSurfaceBackgroundPatternVisible
- OverrideGraphicSettings.SetCutFillColor → OverrideGraphicSettings.SetCutForegroundPatternColor + OverrideGraphicSettings.SetCutBackgroundPatternColor
- OverrideGraphicSettings.CutFillColor → OverrideGraphicSettings.CutForegroundPatternColor + OverrideGraphicSettings.CutBackgroundPatternColor
- OverrideGraphicSettings.SetCutFillPatternId → OverrideGraphicSettings.SetCutForegroundPatternId + OverrideGraphicSettings.SetCutBackgroundPatternId
- OverrideGraphicSettings.CutFillPatternId → OverrideGraphicSettings.CutForegroundPatternId + OverrideGraphicSettings.CutBackgroundPatternId
- OverrideGraphicSettings.SetCutFillPatternVisible → OverrideGraphicSettings.SetCutForegroundPatternVisible + OverrideGraphicSettings.SetCutBackgroundPatternVisible
- OverrideGraphicSettings.IsCutFillPatternVisible → OverrideGraphicSettings.IsCutForegroundPatternVisible + OverrideGraphicSettings.IsCutBackgroundPatternVisible
#### Support for double patterns for FilledRegion
Three properties in the FilledRegionType class have been deprecated and replaced to support double patterns:
- Deprecated property → Replacement/new properties
- FilledRegionType.Background → FilledRegionType.IsMasking
- FilledRegionType.Color → FilledRegionType.ForegroundPatternColor + FilledRegionType.BackgroundPatternColor
- FilledRegionType.FillPatternId → FilledRegionType.ForegroundPatternId + FilledRegionType.BackgroundPatternId
#### Document.Title consistency
In prior releases, the property:
- Document.Title
honoured the Windows user setting requiring dlsplay of file extensions. The property has been changed to consistently return a title with no extension.
#### View printing & exporting – activation events behavioral change
Because of an improvement to Revit processing, views will no longer be activated prior to being printed or exported. This means that the ViewActivating and ViewActivated events are no longer triggered as each view is printed or exported from a set.
#### Structural API change
The following method has been deprecated:
- Deprecated property → Replacement
- StructuralConnectionHandler.SetDefaultPrimaryElement() → StructuralConnectionHandler.SetDefaultElementOrder()
#### Rebar API changes
The method:
- Autodesk.Revit.DB.Structure.Rebar.GetShapeId()
now throws an exception if the rebar is a free-form which has multiple shapes (i.e., its WorkshopInstructions type is Bent).
#### MEP API changes
The following members have been deprecated:
- Deprecated property → Replacement – Notes
- MEPAnalyticalConnectionType.GetAllTypes() → none – Use FilteredElementCollector to find elements that match this class.
- Duct.IsLevelId() → none – Use GetElement() to check the element type
- FlexDuct.IsLevelId() → none – Use GetElement() to check the element type
- Pipe.IsLevelId() → none – Use GetElement() to check the element type
- FlexPipe.IsLevelId() → none – Use GetElement() to check the element type
#### ProjectLocation changes
The original version of ProjectLocation.IsProjectLocationNameUnique checked for name uniqueness across the entire document. The replacement version only enforces uniqueness among the ProjectLocations belonging to a given SiteLocation.
- Deprecated function → Replacement function
- ProjectLocation.IsProjectLocationNameUnique(Document doc, String name) → ProjectLocation.IsProjectLocationNameUnique(Document doc, String name, ElementId siteLocationId)
#### Building Site export removed
The following method and class were removed:
- BuildingSiteExportOptions
- Document.Export(String,String,View3D, ViewPlan, BuildingSiteExportOptions)
corresponding to the removal of this capability from Revit itself.
#### Obsolete API removal
The following API members and classes which had previously been marked Obsolete have been removed in this release. Consult the API documentation from prior releases for information on the replacements to use:
#### Classes
#### Methods
- Autodesk.Revit.Creation.Document.NewElectricalSystem(Autodesk.Revit.DB.Connector, Autodesk.Revit.DB.Electrical.ElectricalSystemType)
- Autodesk.Revit.Creation.Document.NewElectricalSystem(System.Collections.Generic.ICollection, Autodesk.Revit.DB.Electrical.ElectricalSystemType)
- Autodesk.Revit.DB.Structure.Rebar.ComputeDrivingCurves()
- Autodesk.Revit.DB.Structure.Rebar.GetBarPositionTransform()
- Autodesk.Revit.DB.Structure.Rebar.GetDistributionPath()
- Autodesk.Revit.DB.Structure.Rebar.ScaleToBox()
- Autodesk.Revit.DB.Structure.Rebar.ScaleToBoxFor3D()
- Autodesk.Revit.DB.Structure.Rebar.SetLayoutAsFixedNumber()
- Autodesk.Revit.DB.Structure.Rebar.SetLayoutAsMaximumSpacing()
- Autodesk.Revit.DB.Structure.Rebar.SetLayoutAsMinimumClearSpacing()
- Autodesk.Revit.DB.Structure.Rebar.SetLayoutAsNumberWithSpacing()
- Autodesk.Revit.DB.Structure.Rebar.SetLayoutAsSingle()
- Autodesk.Revit.DB.Plumbing.PipeSettings.GetPressLossCalculationServerInfo()
- Autodesk.Revit.DB.Plumbing.PipeSettings.SetPressLossCalculationServerInfo()
- Autodesk.Revit.DB.ExternalResourceBrowserData.IsValidResouceName()
#### Properties
- Autodesk.Revit.DB.Structure.Rebar.ArrayLength
- Autodesk.Revit.DB.Structure.Rebar.BarsOnNormalSide
- Autodesk.Revit.DB.Structure.Rebar.BaseFinishingTurns
- Autodesk.Revit.DB.Structure.Rebar.Height
- Autodesk.Revit.DB.Structure.Rebar.MultiplanarDepth
- Autodesk.Revit.DB.Structure.Rebar.Normal
- Autodesk.Revit.DB.Structure.Rebar.Pitch
- Autodesk.Revit.DB.Structure.Rebar.RebarShapeId
- Autodesk.Revit.DB.Structure.Rebar.TopFinishingTurns
- Autodesk.Revit.DB.ProjectLocation.ProjectPosition
- Autodesk.Revit.DB.ProjectLocation.SitePosition
---

## Related Sections
- [API Changes - What's New in the Revit 2019.1 API](./1676-01_whats_new_in_the_revit_20191_api.md)
- [API Changes - API Additions](./1676-03_api_additions.md)
