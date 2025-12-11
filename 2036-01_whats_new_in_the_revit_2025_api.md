---
author: Jeremy Tammik
optimization_date: '2025-12-11T11:24:41.320887'
original_url: https://thebuildingcoder.typepad.com/blog/2036_whats_new_2025.html
parent_post: 2036_whats_new_2025.md
part_number: '01'
part_total: '30'
post_number: '2036'
series: geometry
slug: whats_new_2025_what's_new_in_the_revit_2025_api
source_file: 2036_whats_new_2025.md
tags:
- csharp
- elements
- family
- filtering
- geometry
- levels
- parameters
- python
- references
- revit-api
- schedules
- selection
- sheets
- views
- walls
title: 1. API Changes - What's New in the Revit 2025 API
word_count: 1978
---

### What's New in the Revit 2025 API
The Revit 2025 API contains significant changes and enhancements for add-in developers:
- [Information sources](#1)
- [What's new in previous versions](#2)
- [Detailed table of contents](#4)
Top level table of contents:

* 1. [API changes](#4.1)

+ 1.1. [CefSharp upgrade](#4.1.1)
+ 1.2. [Revit now on .NET 8](#4.1.2)
+ 1.3. [Add-ins and macro changes](#4.1.3)
+ 1.4. [Array API changes](#4.1.4)
+ 1.5. [BRepBuilder API changes](#4.1.5)
+ 1.6. [Dimension API changes](#4.1.6)
+ 1.7. [Extensible Storage(Schema) API changes](#4.1.7)
+ 1.8. [Electrical API changes](#4.1.8)
+ 1.9. [Link Visibility/Graphic Override API changes](#4.1.9)
+ 1.10.  [MEP changes](#4.1.10)
+ 1.11.  [Reinforcement API changes](#4.1.11)
+ 1.12.  [Slab API changes](#4.1.12)
+ 1.13.  [Structure API changes](#4.1.13)
+ 1.14.  [Task Dialog API changes](#4.1.14)
+ 1.15.  [Toposolid API changes](#4.1.15)
+ 1.16.  [Obsolete API removal](#4.1.16)

* 2. [API additions](#4.2)

+ 2.1. [Add-ins and macros additions](#4.2.1)
+ 2.2. [Analysis API additions](#4.2.2)
+ 2.3. [Annotations API additions](#4.2.3)
+ 2.4. [Array API additions](#4.2.4)
+ 2.5. [MEP Fabrication API additions](#4.2.5)
+ 2.6. [DirectShape API additions](#4.2.6)
+ 2.7. [Dimension API additions](#4.2.7)
+ 2.8. [Electrical API additions](#4.2.8)
+ 2.9. [Energy Analysis API additions](#4.2.9)
+ 2.10.  [IFC API additions](#4.2.10)
+ 2.11.  [Import Export API additions](#4.2.11)
+ 2.12.  [PDF Export API additions](#4.2.12)
+ 2.13.  [Reinforcement API additions](#4.2.13)
+ 2.14.  [Selection API additions](#4.2.14)
+ 2.15.  [Sketched Element API additions](#4.2.15)
+ 2.16.  [Structure API additions](#4.2.16)
+ 2.17.  [Tag/Keynotes API additions](#4.2.17)
+ 2.18.  [Toposolid API additions](#4.2.18)
+ 2.19.  [UI API additions](#4.2.19)
+ 2.20.  [View API additions](#4.2.20)
+ 2.21.  [Link Visibility/Graphic Override API additions](#4.2.21)
+ 2.22.  [RevitServer Enterprise / Revit Cloud Worksharing API additions](#4.2.22)

#### Information Sources
The information below is based on the contents of the \*Revit Platform API Changes and Additions.docx\* document included with
the Revit 2025 SDK, the software developers kit available from
the [Revit Developer Centre](https://www.autodesk.com/developer-network/platform-technologies/revit).
It is also provided in the section on \*What's New\* in the Revit 2025 API help file `RevitAPI.chm` included with the SDK:
![Revit 2025 API help on What's New](img/revit_2025_api_chm.png "Revit 2025 API help on What's New")
For convenient, easy, and effective web searching, this blog post provides a cleaned-up online HTML version of that information with numbering and table of contents added, as well as the following PDF printout of the original document included in the SDK with table of contents and page numbers added:
- [Revit_Platform_API_Changes_and_Additions_2025.pdf](doc/revit_2025_api_changes_and_additions.pdf)
The \*What's New\* section and the \*Changes and Additions\* document provide important information for discovering and exploring the newly added API functionality and for later reference.
If you encounter any issues migrating your existing add-ins between different versions, this is one of the first places to look.
For detailed information on all other aspects of the Revit API, please refer to the rest of the API documentation and samples provided in the SDK.
The most important things to install and keep at hand are:
- The Revit API help file `RevitAPI.chm`
- The Visual Studio solution containing all the SDK samples, `Samples\SDKSamples.sln`
You will regularly need both for research on how to solve specific Revit API programming tasks.
More in-depth official explanations and background information is provided by the
online [Revit API Developers Guide](http://help.autodesk.com/view/RVT/2025/ENU/?guid=Revit_API_Revit_API_Developers_Guide_html) included
in the [Revit 2025 online help](http://help.autodesk.com/view/RVT/2025/ENU).
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
- [What's New in the Revit 2020 API](https://thebuildingcoder.typepad.com/blog/2019/04/whats-new-in-the-revit-2020-api.html)
- [What's New in the Revit 2020.1 API](https://thebuildingcoder.typepad.com/blog/2019/09/whats-new-in-the-revit-20201-api.html)
- [What's New in the Revit 2021 API](https://thebuildingcoder.typepad.com/blog/2020/04/whats-new-in-the-revit-2021-api.html)
- [What's New in the Revit 2021.1 API](https://thebuildingcoder.typepad.com/blog/2020/08/revit-20211-sdk-and-whats-new.html)
- [What's New in the Revit 2022 API](https://thebuildingcoder.typepad.com/blog/2021/04/whats-new-in-the-revit-2022-api.html)
- [What's New in the Revit 2022.1 API](https://thebuildingcoder.typepad.com/blog/2021/11/whats-new-in-the-revit-20221-api.html)
- [What's New in the Revit 2023 API](https://thebuildingcoder.typepad.com/blog/2022/04/whats-new-in-the-revit-2023-api.html)
- [What's New in the Revit 2024 API](https://thebuildingcoder.typepad.com/blog/2023/04/whats-new-in-the-revit-2024-api.html)
#### Detailed Table of Contents

* 1. [API changes](#4.1)

+ 1.1. [CefSharp upgrade](#4.1.1)
+ 1.2. [Revit now on .NET 8](#4.1.2)
+ 1.3. [Add-ins and macro changes](#4.1.3)

- 1.3.1. [MacroManager API](#4.1.3.1)

+ 1.4. [Array API changes](#4.1.4)
+ 1.5. [BRepBuilder API changes](#4.1.5)
+ 1.6. [Dimension API changes](#4.1.6)
+ 1.7. [Extensible Storage(Schema) API changes](#4.1.7)
+ 1.8. [Electrical API changes](#4.1.8)

- 1.8.1. [Distribution system](#4.1.8.1)
- 1.8.2. [Parameter Naming](#4.1.8.2)

+ 1.9. [Link Visibility/Graphic Override API changes](#4.1.9)
+ 1.10.  [MEP changes](#4.1.10)

- 1.10.1.  [Duct Settings](#4.1.10.1)

+ 1.11.  [Reinforcement API changes](#4.1.11)

- 1.11.1.  [Rebar](#4.1.11.1)

+ 1.12.  [Slab API changes](#4.1.12)
+ 1.13.  [Structure API changes](#4.1.13)

- 1.13.1.  [Bending details in view](#4.1.13.1)
- 1.13.2.  [Analytical Surface](#4.1.13.2)

+ 1.14.  [Task Dialog API changes](#4.1.14)
+ 1.15.  [Toposolid API changes](#4.1.15)
+ 1.16.  [Obsolete API removal](#4.1.16)

- 1.16.1.  [Classes](#4.1.16.1)
- 1.16.2.  [Properties](#4.1.16.2)
- 1.16.3.  [Methods](#4.1.16.3)
- 1.16.4.  [Enums](#4.1.16.4)

* 2. [API additions](#4.2)

+ 2.1. [Add-ins and macros additions](#4.2.1)

- 2.1.1. [MacroManager API](#4.2.1.1)

+ 2.2. [Analysis API additions](#4.2.2)

- 2.2.1. [MEP Analytical networks](#4.2.2.1)
- 2.2.2. [MEP Duct/Pipe Pressure Loss calculation](#4.2.2.2)
- 2.2.3. [MEP Space Engineering Parameters](#4.2.2.3)

+ 2.3. [Annotations API additions](#4.2.3)
+ 2.4. [Array API additions](#4.2.4)

- 2.4.1. [Linear Array](#4.2.4.1)

+ 2.5. [MEP Fabrication API additions](#4.2.5)
+ 2.6. [DirectShape API additions](#4.2.6)
+ 2.7. [Dimension API additions](#4.2.7)
+ 2.8. [Electrical API additions](#4.2.8)
+ 2.9. [Energy Analysis API additions](#4.2.9)

- 2.9.1. [gbXML export options](#4.2.9.1)

+ 2.10.  [IFC API additions](#4.2.10)

- 2.10.1.  [IFC Hybrid Import](#4.2.10.1)
- 2.10.2.  [IFC Category Mapping](#4.2.10.2)

+ 2.11.  [Import Export API additions](#4.2.11)
+ 2.12.  [PDF Export API additions](#4.2.12)
+ 2.13.  [Reinforcement API additions](#4.2.13)

- 2.13.1.  [Rebar](#4.2.13.1)
- 2.13.2.  [Rebar splice type options and rules](#4.2.13.2)

+ 2.14.  [Selection API additions](#4.2.14)

- 2.14.1.  [UI Application](#4.2.14.1)

+ 2.15.  [Sketched Element API additions](#4.2.15)

- 2.15.1.  [Wall APIs](#4.2.15.1)

+ 2.16.  [Structure API additions](#4.2.16)

- 2.16.1.  [Analytical Elements](#4.2.16.1)
- 2.16.2.  [Analytical Surface](#4.2.16.2)
- 2.16.3.  [Bending Details on Drawings](#4.2.16.3)
- 2.16.4.  [Radial Array](#4.2.16.4)

+ 2.17.  [Tag/Keynotes API additions](#4.2.17)
+ 2.18.  [Toposolid API additions](#4.2.18)
+ 2.19.  [UI API additions](#4.2.19)

- 2.19.1.  [Context Menu](#4.2.19.1)

+ 2.20.  [View API additions](#4.2.20)

- 2.20.1.  [SheetCollection](#4.2.20.1)

+ 2.21.  [Link Visibility/Graphic Override API additions](#4.2.21)
+ 2.22.  [RevitServer Enterprise / Revit Cloud Worksharing API additions](#4.2.22)

# 1. API Changes

## 1.1. CefSharp upgrade

Revit and Autodesk add-ins use the CEFsharp library internally for several features. Some third-party add-ins do so as well. Occasionally, when different versions of the library are used, it leads to instability issues for Revit. In order to avoid version conflicts, we are clarifying what CEFsharp version is being used, and loading it prior to all add-in initializations.
- In this version, Revit uses CEFsharp version v119.4.30

## 1.2. Revit now on .NET 8

The Revit API for Revit 2025 is built on .NET 8.
This upgrade keeps Revit up to date with the latest .NET features, performance improvements, and security fixes.
Addins for Revit 2025 will need to be rebuilt on .NET 8.

## 1.3. Add-ins and macro changes
---

## Related Sections
- [1. API Changes - 1.3.1. MacroManager API](./2036-02_131_macromanager_api.md)
- [1. API Changes - 1.8.1. Distribution system](./2036-03_181_distribution_system.md)
- [1. API Changes - 1.8.2. Parameter Naming](./2036-04_182_parameter_naming.md)
- [1. API Changes - 1.10.1. Duct Settings](./2036-05_1101_duct_settings.md)
- [1. API Changes - 1.11.1. Rebar](./2036-06_1111_rebar.md)
- [1. API Changes - 1.13.1. Bending details in view](./2036-07_1131_bending_details_in_view.md)
- [1. API Changes - 1.13.2. Analytical Surface](./2036-08_1132_analytical_surface.md)
- [1. API Changes - 1.16.1. Classes](./2036-09_1161_classes.md)
- [1. API Changes - 1.16.2. Properties](./2036-10_1162_properties.md)
- [1. API Changes - 1.16.3. Methods](./2036-11_1163_methods.md)
- [1. API Changes - 1.16.4. Enums](./2036-12_1164_enums.md)
- [1. API Changes - 2.1.1. MacroManager API](./2036-13_211_macromanager_api.md)
- [1. API Changes - 2.2.1. MEP Analytical networks](./2036-14_221_mep_analytical_networks.md)
- [1. API Changes - 2.2.2. MEP Duct/Pipe Pressure Loss calculation](./2036-15_222_mep_ductpipe_pressure_loss_calculation.md)
- [1. API Changes - 2.2.3. MEP Space Engineering Parameters](./2036-16_223_mep_space_engineering_parameters.md)
- [1. API Changes - 2.4.1. Linear Array](./2036-17_241_linear_array.md)
- [1. API Changes - 2.9.1. gbXML export options](./2036-18_291_gbxml_export_options.md)
- [1. API Changes - 2.10.1. IFC Hybrid Import](./2036-19_2101_ifc_hybrid_import.md)
- [1. API Changes - 2.10.2. IFC Category Mapping](./2036-20_2102_ifc_category_mapping.md)
- [1. API Changes - 2.13.1. Rebar](./2036-21_2131_rebar.md)
- [1. API Changes - 2.13.2. Rebar splice type options and rules](./2036-22_2132_rebar_splice_type_options_and_rules.md)
- [1. API Changes - 2.14.1. UI Application](./2036-23_2141_ui_application.md)
- [1. API Changes - 2.15.1. Wall APIs](./2036-24_2151_wall_apis.md)
- [1. API Changes - 2.16.1. Analytical Elements](./2036-25_2161_analytical_elements.md)
- [1. API Changes - 2.16.2. Analytical Surface](./2036-26_2162_analytical_surface.md)
- [1. API Changes - 2.16.3. Bending Details on Drawings](./2036-27_2163_bending_details_on_drawings.md)
- [1. API Changes - 2.16.4. Radial Array](./2036-28_2164_radial_array.md)
- [1. API Changes - 2.19.1. Context Menu](./2036-29_2191_context_menu.md)
- [1. API Changes - 2.20.1. SheetCollection](./2036-30_2201_sheetcollection.md)
