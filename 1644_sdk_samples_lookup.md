---
ai_optimized: true
author: Jeremy Tammik
complexity_score: 1.7
content_type: news
optimization_date: '2025-12-11T11:44:16.451459'
original_url: https://thebuildingcoder.typepad.com/blog/1644_sdk_samples_lookup.html
post_number: '1644'
reading_time_minutes: 2
series: general
slug: sdk_samples_lookup
source_file: 1644_sdk_samples_lookup.md
tags:
- elements
- geometry
- references
- revit-api
- sheets
title: Sdk Samples Lookup
word_count: 308
---

### RevitLookup 2019 and New SDK Samples
Last week, I described how I installed Revit 2019
and [compiled the Revit 2019 SDK samples](http://thebuildingcoder.typepad.com/blog/2018/04/compiling-the-revit-2019-sdk-samples.html).
On Sunday, I migrated RevitLookup to Revit 2019, which was very easy.
Next, I compared the directory contents to discover the new SDK samples:
- [RevitLookup 2019](#2)
- [New Revit 2019 SDK samples](#3)
#### RevitLookup 2019
The migration of [RevitLookup](https://github.com/jeremytammik/RevitLookup) to Revit 2019 was trivial.
It just required updating the .NET framework target version to 4.7 and pointing the Revit API assembly references to the new DLLs.
No code changes were needed.
To add the final finishing touch, I also updated the readme file with a new Revit version badge.
The current version is [2019.0.0.1](https://github.com/jeremytammik/RevitLookup/releases/tag/2019.0.0.1).
[Builds](https://github.com/jeremytammik/RevitLookup#builds) are available
from [lookupbuilds.com](https://lookupbuilds.com).
#### New Revit 2019 SDK Samples
Comparing the Revit SDK directory contents, I discovered the following new samples:
- [AppearanceAssetEditing](http://thebuildingcoder.typepad.com/blog/2017/11/modifying-material-visual-appearance.html) – added
in [Revit 2018.1](http://thebuildingcoder.typepad.com/blog/2017/08/revit-20181-and-the-visual-materials-api.html).
- RebarFreeForm – external command to create a Rebar FreeForm element and external application to implement the custom server used to regenerate the rebar geometry based on constraints.
- SampleCommandsSteelElements – sample commands for steel elements demonstrating creation, modification and deletion of them.
The new API functionality is discussed in the document \*Revit Platform API Changes and Additions.docx\* and the \*What's New\* section of the Revit API help file RevitAPI.chm.
The SDK also sports a new undocumented structural analysis DLL:
- Structural Analysis SDK/Examples/References/CodeChecking/Engineering/rcuapiNET.dll
![Steel connection](img/steel_connection.png)