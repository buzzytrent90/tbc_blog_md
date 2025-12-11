---
ai_optimized: true
author: Jeremy Tammik
complexity_score: 1.7
content_type: tutorial
optimization_date: '2025-12-11T11:44:16.586686'
original_url: https://thebuildingcoder.typepad.com/blog/1713_room_boundary_wpf.html
post_number: '1713'
reading_time_minutes: 3
series: general
slug: room_boundary_wpf
source_file: 1713_room_boundary_wpf.md
tags:
- csharp
- revit-api
- rooms
- sheets
- views
title: Room Boundary Wpf
word_count: 567
---

### Room Boundaries to CSV and WPF Template
Happy New Year to the Revit API developer community!
I spent some time during the winter break working on CSV export of room boundaries for a Forge BIM surface classification tool.
Ali Asad presented a new Visual Studio WPF MVVM Revit add-in template:
- [Export room boundaries to CSV for Forge surface classification](#2)
- [Visual Studio WPF MVVM Revit add-in template](#3)
#### Export Room Boundaries to CSV for Forge Surface Classification
A Forge BIM surface classification tool requires room boundaries to display them in the Forge viewer.
![Forge BIM surface classification](img/forge_bim_surface_classification.png)
One simple way to obtain them via the Revit API is demonstrated
by [The Building Coder samples](https://github.com/jeremytammik/the_building_coder_samples) in
the [external command `CmdListAllRooms`](https://github.com/jeremytammik/the_building_coder_samples/blob/master/BuildingCoder/BuildingCoder/CmdListAllRooms.cs).
It was originally presented in 2011, and enhanced in some further discussions:
- [Accessing room data](http://thebuildingcoder.typepad.com/blog/2011/11/accessing-room-data.html).
- [How to distinguish redundant rooms](http://thebuildingcoder.typepad.com/blog/2016/04/how-to-distinguish-redundant-rooms.html)
- [Bounding box `ExpandToContain` and lower left corner of room](http://thebuildingcoder.typepad.com/blog/2016/08/vacation-end-forge-news-and-bounding-boxes.html#6)
- [2D convex hull algorithm in C# using `XYZ`](http://thebuildingcoder.typepad.com/blog/2016/08/online-revit-api-docs-and-convex-hull.html#3)
I modified its output to generate the required data and export that to CSV in a number of release updates:
- [2019.0.144.14](https://github.com/jeremytammik/the_building_coder_samples/releases/tag/2019.0.144.14) – export room boundaries in millimetres
- [2019.0.144.13](https://github.com/jeremytammik/the_building_coder_samples/releases/tag/2019.0.144.13) – implemented IntPoint3d
- [2019.0.144.12](https://github.com/jeremytammik/the_building_coder_samples/releases/tag/2019.0.144.12) – implemented IntPoint2d
- [2019.0.144.11](https://github.com/jeremytammik/the_building_coder_samples/releases/tag/2019.0.144.11) – implemented onlySpaceSeparator argument for PointString and PointArrayString
- [2019.0.144.10](https://github.com/jeremytammik/the_building_coder_samples/releases/tag/2019.0.144.10) – remove Z component from room boundary and convex hull
- [2019.0.144.9](https://github.com/jeremytammik/the_building_coder_samples/releases/tag/2019.0.144.9) – implemented CSV export for CmdListAllRooms
- [2019.0.144.8](https://github.com/jeremytammik/the_building_coder_samples/releases/tag/2019.0.144.8) – implemented export of complete list of points of first room boundary loop
- [2019.0.144.7](https://github.com/jeremytammik/the_building_coder_samples/releases/tag/2019.0.144.7) – handle empty boundary curve in GetConvexHullOfRoomBoundary
Next, I might reimplement the external command as a DB-only add-in to be run in the DA4R
or [Forge Design Automation for Revit](https://thebuildingcoder.typepad.com/blog/2018/11/forge-design-automation-for-revit-at-au-and-in-public.html) environment.
#### Visual Studio WPF MVVM Revit Add-In Template
[Ali @imaliasad Asad](https://twitter.com/imaliasad)
[presented](https://twitter.com/imaliasad/status/1078989674172035072)
a [Visual Studio WPF Revit add-in template](https://github.com/imAliAsad/VisualStudioRevitTemplate).
It empowers you to use the Visual Studio WPF template for Revit add-in development and includes:
- Well organized MVVM architecture for Revit add-in development
- WPF user control to design beautiful Revit add-in
- Auto create ribbon tab and panel
- `Util.cs` for writing helper methods
![Visual Studio WPF MVVM Revit add-in template](img/Revit2017WPF.png)
Many thanks to Ali for sharing this useful tool!