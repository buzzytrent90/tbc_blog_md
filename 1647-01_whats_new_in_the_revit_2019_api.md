---
author: Jeremy Tammik
optimization_date: '2025-12-11T11:24:48.936962'
original_url: https://thebuildingcoder.typepad.com/blog/1647_whats_new_2019.html
parent_post: 1647_whats_new_2019.md
part_number: '01'
part_total: '03'
post_number: '1647'
series: geometry
slug: whats_new_2019_what's_new_in_the_revit_2019_api
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
title: API Changes - What's New in the Revit 2019 API
word_count: 1848
---

### What's New in the Revit 2019 API
The Revit 2019 API includes numerous exciting enhancements for add-in developers. Many relate directly to explicit developer wishes and requests:
- [Information sources](#1)
- [Installation and migration from previous versions](#2)
- [What's new in previous versions](#3)
- [What's new in the Revit 2019 API – detailed ToC](#4)
- [1. API Changes](#4.1)
- [1.1. .NET 4.7](#4.1.1)
- [1.2. View Filters support OR operators and nesting](#4.1.2)
- [1.3. View.ViewName deprecated](#4.1.3)
- [1.4. UI API changes](#4.1.4)
- [1.5. Existing APIs now support open from cloud paths and C4R](#4.1.5)
- [1.6. BasicFileInfo changes](#4.1.6)
- [1.7. Asset-related API members deprecated](#4.1.7)
- [1.8. Double Pattern Support](#4.1.8)
- [1.9. Document.Title consistency](#4.1.9)
- [1.10. View printing & exporting – activation events behavioral change](#4.1.10)
- [1.11. Structural API change](#4.1.11)
- [1.12. Rebar API changes](#4.1.12)
- [1.13. MEP API changes](#4.1.13)
- [1.14. ProjectLocation changes](#4.1.14)
- [1.15. Building Site export removed](#4.1.15)
- [1.16. Obsolete API removal](#4.1.16)
- [2. API Additions](#4.2)
- [2.1. Material API additions](#4.2.1)
- [2.2. Railing API additions](#4.2.2)
- [2.3. Find Element Dependencies](#4.2.3)
- [2.4. Dimension API additions](#4.2.4)
- [2.5. Set vertical alignment for text](#4.2.5)
- [2.6. BrowserOrganization API additions](#4.2.6)
- [2.7. Rebar API additions](#4.2.7)
- [2.8. Steel Fabrication API additions](#4.2.8)
- [2.9. MEP API additions](#4.2.9)
- [2.10. MEP Fabrication API additions](#4.2.10)
- [2.11. Analysis Visualization Framework API addition](#4.2.11)
- [2.12. IFC additions](#4.2.12)
- [2.13. Revit Link Additions](#4.2.13)
- [2.14. Export Additions](#4.2.14)
- [2.15. Access to add-in data paths](#4.2.15)
- [2.16. Site API additions](#4.2.16)
- [2.17. UI API additions](#4.2.17)
#### Information Sources
The highlights were pointed out in
the [DevDays Online Revit API news video recording](http://thebuildingcoder.typepad.com/blog/2018/02/devdays-keynote-manager-and-rex-extensions.html#2).
The information below is based on the contents of the \*Revit Platform API Changes and Additions.docx\* document included with
the [Revit 2019 SDK](http://thebuildingcoder.typepad.com/blog/2018/04/compiling-the-revit-2019-sdk-samples.html#3)
– software developers kit – available from the Revit installation package and
the [Revit Developer Centre](https://www.autodesk.com/developer-network/platform-technologies/revit) (not posted yet, but coming soon).
It is also provided in the section on \*What's New\* in the Revit 2019 API help file `RevitAPI.chm` included with the SDK:
![Revit 2019 API help on What's New](img/revit_2019_api_help.png)
For convenient, easy and effective web searching, this blog post provides an online version of that information, both in pure HTML (below) as as a PDF document:
- [Revit_Platform_API_Changes_and_Additions_2019.pdf](zip/Revit_Platform_API_Changes_and_Additions_2019.pdf)
The \*What's New\* section always provides very important information, both for discovering and exploring the newly added API functionality and for later reference.
If you encounter any issues migrating your existing add-ins between different versions, this is one of the first places to look.
For detailed information on all other aspects of the Revit API, please refer to the rest of the API documentation and samples provided in the SDK.
The most important things to install and always keep at hand are:
- The Revit API help file `RevitAPI.chm`
- The Visual Studio solution containing all the SDK samples, `Samples\SDKSamples.sln`
You will need both of these constantly for research on how to solve specific Revit API programming tasks.
More in-depth official explanations and background information is provided by the
online [Revit API Developers Guide](http://help.autodesk.com/view/RVT/2019/ENU/?guid=Revit_API_Revit_API_Developers_Guide_html) included
in the [Revit 2019 Help](http://help.autodesk.com/view/RVT/2019/ENU).
#### Installation and Migration From Previous Versions
Here are the steps I performed so far setting up the Revit 2019 SDK and samples:
- The [Revit 2019 SDK samples](http://thebuildingcoder.typepad.com/blog/2018/04/compiling-the-revit-2019-sdk-samples.html)
- [RevitLookup](http://thebuildingcoder.typepad.com/blog/2018/04/revitlookup-2019-and-new-sdk-samples.html)
- [The Building Coder samples](http://thebuildingcoder.typepad.com/blog/2018/04/forge-rvt-to-ifc-adn-xtra-tbc-and-adnrme-updates.html#2)
- The [AdnRme MEP HVAC and electrical samples](http://thebuildingcoder.typepad.com/blog/2018/04/forge-rvt-to-ifc-adn-xtra-tbc-and-adnrme-updates.html#3)
- The [AdnRevitApiLabsXtra training labs](http://thebuildingcoder.typepad.com/blog/2018/04/forge-rvt-to-ifc-adn-xtra-tbc-and-adnrme-updates.html#4)
- The [RvtSamples add-in](http://thebuildingcoder.typepad.com/blog/2018/04/rvtsamples-2019.html)
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
#### What's New in the Revit 2019 API – Detailed ToC
Table of Contents:

* 1. [API changes](#4.1)

+ 1.1. [.NET 4.7](#4.1.1)
+ 1.2. [View Filters support OR operators and nesting](#4.1.2)

- 1.2.1. [ParameterFilterElement API changes](#4.1.2.1)
- 1.2.2. [ElementLogicalFilter API addition](#4.1.2.2)

+ 1.3. [View.ViewName deprecated](#4.1.3)
+ 1.4. [UI API changes](#4.1.4)

- 1.4.1. [Main window handle access](#4.1.4.1)
- 1.4.2. [Changes to PostableCommand enumeration](#4.1.4.2)

+ 1.5. [Existing APIs now support open from cloud paths (Collaboration for Revit)](#4.1.5)

- 1.5.1. [Document Open APIs support cloud paths](#4.1.5.1)
- 1.5.2. [Callback for conflict cases when opening from a cloud path](#4.1.5.2)

+ 1.6. [BasicFileInfo changes](#4.1.6)
+ 1.7. [Asset-related API members deprecated](#4.1.7)
+ 1.8. [Double Pattern Support](#4.1.8)

- 1.8.1. [Material API support for double patterns](#4.1.8.1)
- 1.8.2. [Support for overriding double patterns](#4.1.8.2)
- 1.8.3. [Support for double patterns for FilledRegion](#4.1.8.3)

+ 1.9. [Document.Title consistency](#4.1.9)
+ 1.10. [View printing & exporting – activation events behavioral change](#4.1.10)
+ 1.11. [Structural API change](#4.1.11)
+ 1.12. [Rebar API changes](#4.1.12)
+ 1.13. [MEP API changes](#4.1.13)
+ 1.14. [ProjectLocation changes](#4.1.14)
+ 1.15. [Building Site export removed](#4.1.15)
+ 1.16. [Obsolete API removal](#4.1.16)

* 2. [API additions](#4.2)

+ 2.1. [Material API additions](#4.2.1)

- 2.1.1. [Editing the properties of an Appearance Asset](#4.2.1.1)
- 2.1.2. [Editing AssetProperty values](#4.2.1.2)
- 2.1.3. [Connected Assets](#4.2.1.3)
- 2.1.4. [Validation](#4.2.1.4)
- 2.1.5. [Schemas & Property names](#4.2.1.5)
- 2.1.6. [Utilities](#4.2.1.6)

+ 2.2. [Railing API additions](#4.2.2)

- 2.2.1. [Non-Continuous Rail Structure](#4.2.2.1)
- 2.2.2. [Baluster Placement](#4.2.2.2)

+ 2.3. [Find Element Dependencies](#4.2.3)
+ 2.4. [Dimension API additions](#4.2.4)
+ 2.5. [Set vertical alignment for text](#4.2.5)
+ 2.6. [BrowserOrganization API additions](#4.2.6)
+ 2.7. [Rebar API additions](#4.2.7)

- 2.7.1. [BarTypeDiameterOptions](#4.2.7.1)
- 2.7.2. [GetDistributionPath()](#4.2.7.2)
- 2.7.3. [RebarUpdateCurvesData](#4.2.7.3)
- 2.7.4. [RebarShapes](#4.2.7.4)
- 2.7.5. [Workshop Instructions](#4.2.7.5)
- 2.7.6. [RebarFreeFormAccessor Additions](#4.2.7.6)

+ 2.8. [Steel Fabrication API additions](#4.2.8)

- 2.8.1. [SteelElementProperties – linking between Revit elements and steel fabrication elements](#4.2.8.1)
- 2.8.2. [Custom steel connections – API additions](#4.2.8.2)

+ 2.9. [MEP API additions](#4.2.9)

- 2.9.1. [MEPCurveType Shape](#4.2.9.1)
- 2.9.2. [Mechanical Equipment Set API](#4.2.9.2)
- 2.9.3. [Hydraulic Separation API](#4.2.9.3)

+ 2.10. [MEP Fabrication API additions](#4.2.10)

- 2.10.1. [Exporting fabrication job files](#4.2.10.1)
- 2.10.2. [API for load and unload of one-off fabrication parts from loose item files](#4.2.10.2)
- 2.10.3. [API for version history of parts](#4.2.10.3)
- 2.10.4. [API for part swap out information](#4.2.10.4)
- 2.10.5. [Centerline length API](#4.2.10.5)

+ 2.11. [Analysis Visualization Framework API addition](#4.2.11)
+ 2.12. [IFC additions](#4.2.12)
+ 2.13. [Revit Link Additions](#4.2.13)
+ 2.14. [Export Additions](#4.2.14)
+ 2.15. [Access to add-in data paths](#4.2.15)
+ 2.16. [Site API additions](#4.2.16)
+ 2.17. [UI API additions](#4.2.17)

- 2.17.1. [BeforeExecutedEventArgs Cancellation](#4.2.17.1)
---

## Related Sections
- [API Changes - API Changes](./1647-02_api_changes.md)
- [API Changes - API Additions](./1647-03_api_additions.md)
