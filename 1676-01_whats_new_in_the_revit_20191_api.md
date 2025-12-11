---
author: Jeremy Tammik
optimization_date: '2025-12-11T11:24:48.987592'
original_url: https://thebuildingcoder.typepad.com/blog/1676_whats_new_2019_1.html
parent_post: 1676_whats_new_2019_1.md
part_number: '01'
part_total: '03'
post_number: '1676'
series: geometry
slug: whats_new_2019_1_what's_new_in_the_revit_2019.1_api
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
title: API Changes - What's New in the Revit 2019.1 API
word_count: 1738
---

### What's New in the Revit 2019.1 API
As I mentioned last week,
the [Revit 2019.1 update](http://thebuildingcoder.typepad.com/blog/2018/08/revit-20191-cefsharp-forge-accelerator-in-rome.html#2) has
been released.
It is equipped with an updated API that includes several exciting enhancements for add-in developers.
Many relate directly to explicit developer wishes and requests:
- [Information Sources](#1)
- [What's New in Previous Versions](#2)
- [What's New in the Revit 2019.1 API – Detailed ToC](#3)
- [1. API Changes](#3.1)
- [1.1. .NET 4.7](#3.1.1)
- [1.2. View Filters support OR operators and nesting](#3.1.2)
- [1.3. View.ViewName deprecated](#3.1.3)
- [1.4. UI API changes](#3.1.4)
- [1.5. Existing APIs now support open from cloud paths (Collaboration for Revit)](#3.1.5)
- [1.6. BasicFileInfo changes](#3.1.6)
- [1.7. Asset-related API members deprecated](#3.1.7)
- [1.8. Double Pattern Support](#3.1.8)
- [1.9. Document.Title consistency](#3.1.9)
- [1.10. View printing & exporting – activation events behavioral change](#3.1.10)
- [1.11. Structural API change](#3.1.11)
- [1.12. Rebar API changes](#3.1.12)
- [1.13. MEP API changes](#3.1.13)
- [1.14. ProjectLocation changes](#3.1.14)
- [1.15. Building Site export removed](#3.1.15)
- [1.16. Obsolete API removal](#3.1.16)
- [2. API Additions](#3.2)
- [2.17. Material API additions](#3.2.17)
- [2.18. Railing API additions](#3.2.18)
- [2.19. Find Element Dependencies](#3.2.19)
- [2.20. Dimension API additions](#3.2.20)
- [2.21. Set vertical alignment for text](#3.2.21)
- [2.22. BrowserOrganization API additions](#3.2.22)
- [2.23. Rebar API additions](#3.2.23)
- [2.24. Steel Fabrication API additions](#3.2.24)
- [2.25. MEP API additions](#3.2.25)
- [2.26. MEP Fabrication API additions](#3.2.26)
- [2.27. Analysis Visualization Framework API addition](#3.2.27)
- [2.28. IFC additions](#3.2.28)
- [2.29. Revit Link Additions](#3.2.29)
- [2.30. Export Additions](#3.2.30)
- [2.31. Access to add-in data paths](#3.2.31)
- [2.32. Site API additions](#3.2.32)
- [2.33. UI API additions](#3.2.33)
#### Information Sources
The information below is based on the contents of the \*Revit Platform API Changes and Additions.docx\* document included with
the [Revit 2019.1 SDK](http://thebuildingcoder.typepad.com/blog/2018/04/compiling-the-revit-2019-sdk-samples.html#3)
– software developers kit – available from
the [Revit Developer Centre](https://www.autodesk.com/developer-network/platform-technologies/revit) (not posted yet, but coming soon).
It is also provided in the section on \*What's New\* in the Revit 2019.1 API help file `RevitAPI.chm` included with the SDK:
![Revit 2019.1 API help on What's New](img/revit_2019_1_api_help.png)
For convenient, easy and effective web searching, this blog post provides an online version of that information.
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
#### What's New in the Revit 2019.1 API – Detailed ToC
Table of Contents:

* 1. [API changes](#3.1)

+ 1.1. [.NET 4.7](#3.1.1)
+ 1.2. [View Filters support OR operators and nesting](#3.1.2)

- 1.2.1. [ParameterFilterElement API changes](#3.1.2.1)
- 1.2.2. [ElementLogicalFilter API addition](#3.1.2.2)

+ 1.3. [View.ViewName deprecated](#3.1.3)
+ 1.4. [UI API changes](#3.1.4)

- 1.4.1. [Main window handle access](#3.1.4.1)
- 1.4.2. [Changes to PostableCommand enumeration](#3.1.4.2)

+ 1.5. [Existing APIs now support open from cloud paths (Collaboration for Revit)](#3.1.5)

- 1.5.1. [Document Open APIs support cloud paths](#3.1.5.1)
- 1.5.2. [Callback for conflict cases when opening from a cloud path](#3.1.5.2)

+ 1.6. [BasicFileInfo changes](#3.1.6)
+ 1.7. [Asset-related API members deprecated](#3.1.7)
+ 1.8. [Double Pattern Support](#3.1.8)

- 1.8.1. [Material API support for double patterns](#3.1.8.1)
- 1.8.2. [Support for overriding double patterns](#3.1.8.2)
- 1.8.3. [Support for double patterns for FilledRegion](#3.1.8.3)

+ 1.9. [Document.Title consistency](#3.1.9)
+ 1.10. [View printing & exporting – activation events behavioral change](#3.1.10)
+ 1.11. [Structural API change](#3.1.11)
+ 1.12. [Rebar API changes](#3.1.12)
+ 1.13. [MEP API changes](#3.1.13)
+ 1.14. [ProjectLocation changes](#3.1.14)
+ 1.15. [Building Site export removed](#3.1.15)
+ 1.16. [Obsolete API removal](#3.1.16)

- 1.16.1. [Classes](#3.1.16.1)
- 1.16.2. [Methods](#3.1.16.2)
- 1.16.3. [Properties](#3.1.16.3)

* 2. [API additions](#3.2)

+ 2.17. [Material API additions](#3.2.17)

- 2.17.1. [Editing the properties of an Appearance Asset](#3.2.17.1)
- 2.17.2. [Editing AssetProperty values](#3.2.17.2)
- 2.17.3. [Connected Assets](#3.2.17.3)
- 2.17.4. [Validation](#3.2.17.4)
- 2.17.5. [Schemas and Property names](#3.2.17.5)
- 2.17.6. [Utilities](#3.2.17.6)

+ 2.18. [Railing API additions](#3.2.18)

- 2.18.1. [Non-Continuous Rail Structure](#3.2.18.1)
- 2.18.2. [Baluster Placement](#3.2.18.2)

+ 2.19. [Find Element Dependencies](#3.2.19)
+ 2.20. [Dimension API additions](#3.2.20)
+ 2.21. [Set vertical alignment for text](#3.2.21)
+ 2.22. [BrowserOrganization API additions](#3.2.22)
+ 2.23. [Rebar API additions](#3.2.23)

- 2.23.1. [BarTypeDiameterOptions](#3.2.23.1)
- 2.23.2. [GetDistributionPath()](#3.2.23.2)
- 2.23.3. [RebarUpdateCurvesData](#3.2.23.3)
- 2.23.4. [RebarShapes](#3.2.23.4)
- 2.23.5. [Workshop Instructions](#3.2.23.5)
- 2.23.6. [RebarFreeFormAccessor Additions](#3.2.23.6)

+ 2.24. [Steel Fabrication API additions](#3.2.24)

- 2.24.1. [SteelElementProperties – linking between Revit elements and steel fabrication elements](#3.2.24.1)
- 2.24.2. [Custom steel connections – API additions](#3.2.24.2)

+ 2.25. [MEP API additions](#3.2.25)

- 2.25.1. [MEPCurveType Shape](#3.2.25.1)
- 2.25.2. [Mechanical Equipment Set API](#3.2.25.2)
- 2.25.3. [Hydraulic Separation API](#3.2.25.3)

+ 2.26. [MEP Fabrication API additions](#3.2.26)

- 2.26.1. [Exporting fabrication job files](#3.2.26.1)
- 2.26.2. [API for load and unload of one-off fabrication parts from loose item files](#3.2.26.2)
- 2.26.3. [API for version history of parts](#3.2.26.3)
- 2.26.4. [API for part swap out information](#3.2.26.4)
- 2.26.5. [Centerline length API](#3.2.26.5)

+ 2.27. [Analysis Visualization Framework API addition](#3.2.27)
+ 2.28. [IFC additions](#3.2.28)
+ 2.29. [Revit Link Additions](#3.2.29)
+ 2.30. [Export Additions](#3.2.30)
+ 2.31. [Access to add-in data paths](#3.2.31)
+ 2.32. [Site API additions](#3.2.32)
+ 2.33. [UI API additions](#3.2.33)

- 2.33.1. [BeforeExecutedEventArgs Cancellation](#3.2.33.1)
---

## Related Sections
- [API Changes - API Changes](./1676-02_api_changes.md)
- [API Changes - API Additions](./1676-03_api_additions.md)
