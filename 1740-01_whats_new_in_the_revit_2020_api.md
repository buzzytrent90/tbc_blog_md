---
author: Jeremy Tammik
optimization_date: '2025-12-11T11:24:49.034849'
original_url: https://thebuildingcoder.typepad.com/blog/1740_whats_new_2020.html
parent_post: 1740_whats_new_2020.md
part_number: '01'
part_total: '37'
post_number: '1740'
series: geometry
slug: whats_new_2020_what's_new_in_the_revit_2020_api
source_file: 1740_whats_new_2020.md
tags:
- elements
- filtering
- geometry
- levels
- parameters
- references
- revit-api
- schedules
- selection
- sheets
- views
- walls
title: API Changes - What's New in the Revit 2020 API
word_count: 2254
---

### What's New in the Revit 2020 API
The Revit 2020 API includes exciting enhancements for add-in developers, including numerous developer wishes and requests that have now been explicitly addressed:
- [Information sources](#1)
- [Installation and migration from previous versions](#2)
- [What's new in previous versions](#3)
- [Detailed TOC of major changes and renovations](#4)
Short overview – abbreviated table of contents:

* 1. [API Changes](#4.1)

+ 1.1. [Revit automatically initializes CEFsharp](#4.1.1)
+ 1.2. [MEP Fabrication API deprecation](#4.1.2)
+ 1.3. [Structural API deprecation](#4.1.3)
+ 1.4. [Export API changes](#4.1.4)
+ 1.5. [Image API changes](#4.1.5)
+ 1.6. [Obsolete API removal](#4.1.6)

* 2. [API Additions](#4.2)

+ 2.1. [Attached detail group API additions](#4.2.1)
+ 2.2. [PDF and Image API additions](#4.2.2)
+ 2.3. [ElementLogicalFilter API additions](#4.2.3)
+ 2.4. [Application API additions](#4.2.4)
+ 2.5. [UI API additions](#4.2.5)
+ 2.6. [Document additions](#4.2.6)
+ 2.7. [Category API additions](#4.2.7)
+ 2.8. [Parameter API additions](#4.2.8)
+ 2.9. [Geometry API additions](#4.2.9)
+ 2.10. [View API additions](#4.2.10)
+ 2.11. [Schedule API additions](#4.2.11)
+ 2.12. [Site API additions](#4.2.12)
+ 2.13. [Shared Coordinates API additions](#4.2.13)
+ 2.14. [Part API additions](#4.2.14)
+ 2.15. [Railing API additions](#4.2.15)
+ 2.16. [Export API additions](#4.2.16)
+ 2.17. [Multi-Reference Annotation additions](#4.2.17)
+ 2.18. [Material API additions](#4.2.18)
+ 2.19. [Analysis API additions](#4.2.19)
+ 2.20. [Structural API additions](#4.2.20)
+ 2.21. [MEP API additions](#4.2.21)

#### Information Sources
The highlights were pointed out in
the [DevDays Online Revit API news video recording](https://thebuildingcoder.typepad.com/blog/2019/04/the-revit-2020-fcs-api-and-sdk.html#3).
The information below is based on the contents of the \*Revit Platform API Changes and Additions.docx\* document included with
the [Revit 2020 SDK](https://thebuildingcoder.typepad.com/blog/2019/04/revitlookup-and-sdk-for-revit-2020.html#3),
the software developers kit available from the Revit installation package and
the [Revit Developer Centre](https://www.autodesk.com/developer-network/platform-technologies/revit).
It is also provided in the section on \*What's New\* in the Revit 2020 API help file `RevitAPI.chm` included with the SDK:
![Revit 2020 API help on What's New](img/whats_new_2020.png "Revit 2020 API help on What's New")
For convenient, easy and effective web searching, this blog post provides an online version of that information, both in pure HTML (below) and as a PDF document:
- [Revit_Platform_API_Changes_and_Additions_2020.pdf](zip/Revit_Platform_API_Changes_and_Additions_2020.pdf)
The \*What's New\* section always provides very important information, both for discovering and exploring the newly added API functionality and for later reference.
If you encounter any issues migrating your existing add-ins between different versions, this is one of the first places to look.
For detailed information on all other aspects of the Revit API, please refer to the rest of the API documentation and samples provided in the SDK.
The most important things to install and always keep at hand are:
- The Revit API help file `RevitAPI.chm`
- The Visual Studio solution containing all the SDK samples, `Samples\SDKSamples.sln`
You will need both of these constantly for research on how to solve specific Revit API programming tasks.
More in-depth official explanations and background information is provided by the
online [Revit API Developers Guide](http://help.autodesk.com/view/RVT/2020/ENU/?guid=Revit_API_Revit_API_Developers_Guide_html) included
in the [Revit 2020 Help](http://help.autodesk.com/view/RVT/2020/ENU).
#### Installation and Migration From Previous Versions
Here are the steps I performed so far setting up the Revit 2020 SDK and samples:
- [The Revit 2020 FCS, API and SDK](https://thebuildingcoder.typepad.com/blog/2019/04/the-revit-2020-fcs-api-and-sdk.html)
- [RevitLookup and SDK for Revit 2020](https://thebuildingcoder.typepad.com/blog/2019/04/revitlookup-and-sdk-for-revit-2020.html)
#### What's New in Previous Versions
Here are links to previous discussions of \*What's New in the Revit API\*:
- [What's New in the Revit 2010 API](http://thebuildingcoder.typepad.com/blog/2013/02/whats-new-in-the-revit-2010-api.html)
- [What's New in the Revit 2011 API](http://thebuildingcoder.typepad.com/blog/2013/02/whats-new-in-the-revit-2011-api.html)
- [What's New in the Revit 2012 API](http://thebuildingcoder.typepad.com/blog/2013/02/whats-new-in-the-revit-2012-api.html)
- [What's New in the Revit 2013 API](http://thebuildingcoder.typepad.com/blog/2013/03/whats-new-in-the-revit-2013-api.html)
- [What's New in the Revit 2014 API](http://thebuildingcoder.typepad.com/blog/2013/04/whats-new-in-the-revit-2014-api.html)
- [What's New in the Revit 2015 API](http://thebuildingcoder.typepad.com/blog/2014/04/whats-new-in-the-revit-2015-api.html)
- [What's New in the Revit 2016 API](http://thebuildingcoder.typepad.com/blog/2015/04/whats-new-in-the-revit-2016-api.html)
- [What's New in the Revit 2017 API](http://thebuildingcoder.typepad.com/blog/2016/04/whats-new-in-the-revit-2017-api.html)
- [What's New in the Revit 2017.1 API](http://thebuildingcoder.typepad.com/blog/2016/11/whats-new-in-the-revit-20171-api.html)
- [What's New in the Revit 2018 API](http://thebuildingcoder.typepad.com/blog/2017/04/whats-new-in-the-revit-2018-api.html)
- [Revit 2018.1 and the Visual Materials API](http://thebuildingcoder.typepad.com/blog/2017/08/revit-20181-and-the-visual-materials-api.html)
- [Revit 2018.1.1 and 2018.1 API Docs Online](http://thebuildingcoder.typepad.com/blog/2017/09/revit-201811-fixes-cropbox-setting.html)
- [Revit 2018.1 Visual Materials API](http://thebuildingcoder.typepad.com/blog/2017/11/modifying-material-visual-appearance.html)
- [What's New in the Revit 2018.2 API](http://thebuildingcoder.typepad.com/blog/2017/12/whats-new-in-the-revit-20182-api.html)
- [What's New in the Revit 2019 API](http://thebuildingcoder.typepad.com/blog/2018/04/whats-new-in-the-revit-2019-api.html)
#### Detailed TOC of Major Changes and Renovations
Detailed Table of Contents:

* 1. [API Changes](#4.1)

+ 1.1. [Revit automatically initializes CEFsharp](#4.1.1)
+ 1.2. [MEP Fabrication API deprecation](#4.1.2)
+ 1.3. [Structural API deprecation](#4.1.3)
+ 1.4. [Export API changes](#4.1.4)
+ 1.5. [Image API changes](#4.1.5)
+ 1.6. [Obsolete API removal](#4.1.6)

- 1.6.1. [Classes](#4.1.6.1)
- 1.6.2. [Methods](#4.1.6.2)
- 1.6.3. [Properties](#4.1.6.3)

* 2. [API Additions](#4.2)

+ 2.1. [Attached detail group API additions](#4.2.1)
+ 2.2. [PDF and Image API additions](#4.2.2)

- 2.2.1. [ImageType API](#4.2.2.1)
- 2.2.2. [ImageInstance API](#4.2.2.2)

+ 2.3. [ElementLogicalFilter API additions](#4.2.3)

- 2.3.1. [Setting filters](#4.2.3.1)

+ 2.4. [Application API additions](#4.2.4)

- 2.4.1. [Journal playback](#4.2.4.1)

+ 2.5. [UI API additions](#4.2.5)

- 2.5.1. [Add Animated Progress Bar to a TaskDialog](#4.2.5.1)
- 2.5.2. [Specify minimum width and height for a DockablePane](#4.2.5.2)

+ 2.6. [Document additions](#4.2.6)

- 2.6.1. [Returning cloud model information for cloud workshared models](#4.2.6.1)
- 2.6.2. [Open/Save/Modify cloud models on BIM 360](#4.2.6.2)
- 2.6.3. [Identifying if background calculations are underway](#4.2.6.3)

+ 2.7. [Category API additions](#4.2.7)

- 2.7.1. [Category visibility](#4.2.7.1)
- 2.7.2. [Get string names of categories](#4.2.7.2)

+ 2.8. [Parameter API additions](#4.2.8)

- 2.8.1. [Getting localized parameter names](#4.2.8.1)
- 2.8.2. [Hiding empty parameters](#4.2.8.2)
- 2.8.3. [Filtering by presence or absence of parameter value](#4.2.8.3)

+ 2.9. [Geometry API additions](#4.2.9)

- 2.9.1. [CurveLoop API](#4.2.9.1)

+ 2.10. [View API additions](#4.2.10)

- 2.10.1. [View Template API additions](#4.2.10.1)
- 2.10.2. [View Filter API additions](#4.2.10.2)

+ 2.11. [Schedule API additions](#4.2.11)
+ 2.12. [Site API additions](#4.2.12)

- 2.12.1. [TopographySurface additions](#4.2.12.1)
- 2.12.2. [TopographyLinkType additions](#4.2.12.2)

+ 2.13. [Shared Coordinates API additions](#4.2.13)

- 2.13.1. [SiteLocation addition](#4.2.13.1)
- 2.13.2. [BasePoint additions](#4.2.13.2)

+ 2.14. [Part API additions](#4.2.14)

- 2.14.1. [Part creation from DirectShapes](#4.2.14.1)
- 2.14.2. [Returning the entities that create a divided Part](#4.2.14.2)

+ 2.15. [Railing API additions](#4.2.15)

- 2.15.1. [BalusterInfo reference name](#4.2.15.1)

+ 2.16. [Export API additions](#4.2.16)

- 2.16.1. [CustomExporter now supports some 2D views](#4.2.16.1)
- 2.16.2. [New Navisworks export options](#4.2.16.2)

+ 2.17. [Multi-Reference Annotation additions](#4.2.17)

- 2.17.1. [Additional References](#4.2.17.1)
- 2.17.2. [3D view placement restrictions](#4.2.17.2)

+ 2.18. [Material API additions](#4.2.18)

- 2.18.1. [Glazing schema API](#4.2.18.1)

+ 2.19. [Analysis API additions](#4.2.19)

- 2.19.1. [Path of Travel API](#4.2.19.1)

+ 2.20. [Structural API additions](#4.2.20)

- 2.20.1. [RebarConstraint additions](#4.2.20.1)
- 2.20.2. [Updating free-form rebar based on shared parameters](#4.2.20.2)
- 2.20.3. [StructuralConnectionHandler additions](#4.2.20.3)

+ 2.21. [MEP API additions](#4.2.21)

- 2.21.1. [FabricationPart additions](#4.2.21.1)
- 2.21.2. [FabricationNetworkChangeService](#4.2.21.2)
- 2.21.3. [ElectricalSystem API additions](#4.2.21.3)

# API Changes

## 1.1. Revit automatically initializes CEFsharp

Revit and Autodesk add-ins use the CEFsharp library internally for several features. Some third-party add-ins do so as well. Occasionally, when different versions of the library are used, it leads to instability issues for Revit.
In order to avoid version conflicts, we are clarifying what CEFsharp version is being used, and loading it prior to all add-in initialization.
- In this version, Revit uses CEFsharp version 65.0.1.
- In the initialization, legacy JavaScript binding is enabled:

`CefSharpSettings.LegacyJavascriptBindingEnabled = true`

## 1.2. MEP Fabrication API deprecation

The following member has been deprecated and replaced:
- FabricationConfiguration.GetMaterialThickness() → FabricationPart.MaterialThickness

## 1.3. Structural API deprecation

The following member has been deprecated:
- Autodesk.Revit.DB.Structure.StructuralConnectionHandler.GetSubPartIds()
The subpart functionality was previously removed, so there is no replacement method.

## 1.4. Export API changes

The following member has been deprecated and replaced:
- CustomExporter.Export(View3d) → CustomExporter.Export(View)
See the API additions section for more details on the new 2D view support for the custom exporter.

## 1.5. Image API changes

With changes made to support PDF import and extensions to the image API, the following members and classes have been deprecated and replaced:
- ImageType.Create(Document, string) → ImageType.Create(Document, ImageTypeOptions)
- ImageType.ReloadFrom(string) → ImageType.ReloadFrom(ImageTypeOptions)
- Document.Import(string, ImageImportOptions, View, out Element) → ImageType.Create() followed by ImageInstance.Create()
- class ImageImportOptions → class ImageTypeOptions

## 1.6. Obsolete API removal

The following API members and classes which had previously been marked Obsolete have been removed in this release. Consult the API documentation from prior releases for information on the replacements to use:

#### 1.6.1. Classes

#### 1.6.2. Methods

- ProjectLocation.IsProjectLocationNameUnique(Document, String)
- FabricationPart.IsStraightSegment(Document, ElementId)
- FabricationPart.CanSplitStraight(Document, ElementId, XYZ)
- FabricationUtils.ExportToMAJ(Document, IList,String, bool, out IList)
- ParameterFilterElement.Create(Document, String, ICollection, IList)
- ParameterFilterElement.GetRules()
- ParameterFilterElement.SetRules()
- ParameterFilterElement.AllRuleParametersApplicable (IList)
- ParameterFilterElement.AllRuleParametersApplicable (Document, ICollection, IList)
- ParameterFilterElement.GetRuleParameter(FilterRule)
- ParameterFilterElement.GetRuleParameters()

#### 1.6.3. Properties

- BasicFileInfo.SavedInVersion
- AssetProperty.Item[String]
- Application.Assets[AssetType]

# API Additions

## 2.1. Attached detail group API additions

The Group and GroupType API now includes API related to attached detail groups.
The new methods:
- Group.ShowAttachedDetailGroups()
- Group.ShowAllAttachedDetailGroups()
- Group.HideAttachedDetailGroups()
- Group.HideAllAttachedDetailGroups()
control visibility for an element group's attached detail groups as seen in a given view.
The new methods:
- Group.GetAvailableAttachedDetailGroupTypeIds()
- GroupType.getAvailableAttachedDetailGroupTypeIds()
return the attached detail groups available for this group or group type.
The new method:
- Group.IsCompatibleAttachedDetailGroupType()
checks if the orientation of the input attached detail group matches the input view's orientation. Note that detail groups in perpendicular elevation views (for example, North and East views) are considered compatible. When showing these detail groups, an error (FailureMessage) based on id can be generated if the orientation of the annotations do not match the orientation of the target view (for example, the failure definition DimensionPerpendicularToView). To prevent displaying detail groups in the wrong view, you can check the OwnerViewId of a detail group to make sure it matches the view in which you are trying to display it.
The new properties:
- Group.IsAttached
- Group.AttachedParentId
provide information about whether a group is attached and if so, to what group it is associated.

## 2.2. PDF and Image API additions
---

## Related Sections
- [API Changes - 2.2.1. ImageType API](./1740-02_221_imagetype_api.md)
- [API Changes - 2.2.2. ImageInstance API](./1740-03_222_imageinstance_api.md)
- [API Changes - 2.3.1. Setting filters](./1740-04_231_setting_filters.md)
- [API Changes - 2.4.1. Journal playback](./1740-05_241_journal_playback.md)
- [API Changes - 2.5.1. Add Animated Progress Bar to a TaskDialog](./1740-06_251_add_animated_progress_bar_to_a_taskdialog.md)
- [API Changes - 2.5.2. Specify minimum width and height for a DockablePane](./1740-07_252_specify_minimum_width_and_height_for_a_dockabl.md)
- [API Changes - 2.6.1. Returning cloud model information for cloud workshared models](./1740-08_261_returning_cloud_model_information_for_cloud_wo.md)
- [API Changes - 2.6.2. Open/Save/Modify cloud models on BIM 360](./1740-09_262_opensavemodify_cloud_models_on_bim_360.md)
- [API Changes - 2.6.3. Identifying if background calculations are underway](./1740-10_263_identifying_if_background_calculations_are_und.md)
- [API Changes - 2.7.1. Category visibility](./1740-11_271_category_visibility.md)
- [API Changes - 2.7.2. Get string names of categories](./1740-12_272_get_string_names_of_categories.md)
- [API Changes - 2.8.1. Getting localized parameter names](./1740-13_281_getting_localized_parameter_names.md)
- [API Changes - 2.8.2. Hiding empty parameters](./1740-14_282_hiding_empty_parameters.md)
- [API Changes - 2.8.3. Filtering by presence or absence of parameter value](./1740-15_283_filtering_by_presence_or_absence_of_parameter_.md)
- [API Changes - 2.9.1. CurveLoop API](./1740-16_291_curveloop_api.md)
- [API Changes - 2.10.1. View Template API additions](./1740-17_2101_view_template_api_additions.md)
- [API Changes - 2.10.2. View Filter API additions](./1740-18_2102_view_filter_api_additions.md)
- [API Changes - 2.12.1. TopographySurface additions](./1740-19_2121_topographysurface_additions.md)
- [API Changes - 2.12.2. TopographyLinkType additions](./1740-20_2122_topographylinktype_additions.md)
- [API Changes - 2.13.1. SiteLocation addition](./1740-21_2131_sitelocation_addition.md)
- [API Changes - 2.13.2. BasePoint additions](./1740-22_2132_basepoint_additions.md)
- [API Changes - 2.14.1. Part creation from DirectShapes](./1740-23_2141_part_creation_from_directshapes.md)
- [API Changes - 2.14.2. Returning the entities that create a divided Part](./1740-24_2142_returning_the_entities_that_create_a_divided_.md)
- [API Changes - 2.15.1. BalusterInfo reference name](./1740-25_2151_balusterinfo_reference_name.md)
- [API Changes - 2.16.1. CustomExporter now supports some 2D views](./1740-26_2161_customexporter_now_supports_some_2d_views.md)
- [API Changes - 2.16.2. New Navisworks export options](./1740-27_2162_new_navisworks_export_options.md)
- [API Changes - 2.17.1. Additional References](./1740-28_2171_additional_references.md)
- [API Changes - 2.17.2. 3D view placement restrictions](./1740-29_2172_3d_view_placement_restrictions.md)
- [API Changes - 2.18.1. Glazing schema API](./1740-30_2181_glazing_schema_api.md)
- [API Changes - 2.19.1. Path of Travel API](./1740-31_2191_path_of_travel_api.md)
- [API Changes - 2.20.1. RebarConstraint additions](./1740-32_2201_rebarconstraint_additions.md)
- [API Changes - 2.20.2. Updating free-form rebar based on shared parameters](./1740-33_2202_updating_free_form_rebar_based_on_shared_para.md)
- [API Changes - 2.20.3. StructuralConnectionHandler additions](./1740-34_2203_structuralconnectionhandler_additions.md)
- [API Changes - 2.21.1. FabricationPart additions](./1740-35_2211_fabricationpart_additions.md)
- [API Changes - 2.21.2. FabricationNetworkChangeService](./1740-36_2212_fabricationnetworkchangeservice.md)
- [API Changes - 2.21.3. ElectricalSystem API additions](./1740-37_2213_electricalsystem_api_additions.md)
