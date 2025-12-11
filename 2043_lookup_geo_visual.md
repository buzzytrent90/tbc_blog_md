---
ai_optimized: true
author: Jeremy Tammik
complexity_score: 2.6
content_type: documentation
optimization_date: '2025-12-11T11:44:17.347513'
original_url: https://thebuildingcoder.typepad.com/blog/2043_lookup_geo_visual.html
post_number: '2043'
reading_time_minutes: 5
series: general
slug: lookup_geo_visual
source_file: 2043_lookup_geo_visual.md
tags:
- elements
- family
- geometry
- parameters
- references
- revit-api
- sheets
- views
title: Lookup Geo Visual
word_count: 927
---

### RevitLookup Geometry Visualisation
A small step for Roman, a giant leap for the Revit add-in developer community:
- [1000 stars on GitHub](#2)
- [RevitLookup Geometry Visualization](#3)
- [RevitLookup 2025.0.5](#4)
- [RevitLookup 2025.0.6](#5)
- [RevitLookup 2025.0.7](#6)
- [Versions and Visualisation Wiki](#7)
RevitLookup has been rewarded 1000 well-earned stars on GitHub.
To celebrate,
Roman [@Nice3point](https://t.me/nice3point) Karpovich, aka Роман Карпович,
presents a huge new chunk of RevitLookup functionality enabling Revit BIM element geometry visualization.
#### 1000 Stars and Geometry Visualisation
We are proud to share that RevitLookup has achieved 1000 stars on GitHub!
This milestone is a testament to its value and the dedication of our community.
Thank you for helping us reach this landmark!
[![Star History Chart](https://api.star-history.com/svg?repos=jeremytammik/RevitLookup&type=Date)](https://star-history.com/#jeremytammik/RevitLookup&Date)
To celebrate it, we are excited to introduce a major new feature in this release that will transform your interaction with models, offering a deeper understanding of the geometric objects that constitute your models:
#### RevitLookup Geometry Visualization
> Introducing the new Geometry Visualization feature in RevitLookup!
Now you can visualize various geometry objects directly within the interface.
Enhance your BIM workflow with this powerful tool!
![RevitLookup geometry visualisation](img/geovis01.jpg "RevitLookup geometry visualisation")
It is built using the Revit API `DirectContext3D` functionality and described in
the [wiki documentation of RevitLookup Geometry Visualization](https://github.com/jeremytammik/RevitLookup/wiki/Visualization).
It was mainly implemented in release 2025.0.5, with further enhancements following in 2025.0.6 and 2025.0.7.
Please feel free to submit your feedback, wishes and suggestions regarding visualization in
the [comments on pull request 245](https://github.com/jeremytammik/RevitLookup/pull/245).
#### RevitLookup 2025.0.5
[RevitLookup 2025.0.5](https://github.com/jeremytammik/RevitLookup/releases/edit/2025.0.6) includes
comprehensive geometry visualization capabilities, enabling users to visualize various geometry objects directly within the RevitLookup interface.
In Revit, geometry is at the core of every model.
Whether you are dealing with simple shapes or intricate structures, having the ability to visualize geometric elements can significantly improve your workflow, analysis and understanding of the BIM.
To illustrate the power of these visualization capabilities, here are samples of the geometric objects you can now explore directly within RevitLookup:
Mesh:
![Mesh](img/geovis02mesh.png "Mesh")
Face:
![Face](img/geovis03face.png "Face")
Solid:
![Solid](img/geovis04solid.png "Solid")
Curve:
![Curve](img/geovis05curve.png "Curve")
Edge:
![Edge](img/geovis06edge.png "Edge")
BoundingBox:
![BoundingBox](img/geovis07boundingbox.png "BoundingBox")
XYZ:
![XYZ](img/geovis08xyz.png "XYZ")
For detailed documentation, check
the [wiki documentation of RevitLookup Geometry Visualization](https://github.com/jeremytammik/RevitLookup/wiki/Visualization).
Feel free to leave comments and suggestions regarding visualization in
the [pill request 245](https://github.com/jeremytammik/RevitLookup/pull/245).
Your input help improve this tool for everyone in the Revit community.
Other improvements include:
- \*\*BoundingBoxXYZ\*\* class support
- Added `Bounds` method support
- Added `MinEnabled` method support
- Added `MaxEnabled` method support
- Added `BoundEnabled` method support
- Added \*\*Edit parameter\*\* icon
- Added \*\*Select\*\* context menu action for Reference type
- Added \*\*Export family size table\*\* for FamilySizeTableManager type by @SergeyNefyodov in https://github.com/jeremytammik/RevitLookup/pull/244
Added new extensions:
- Application: GetFormulaFunctions – Gets list of function names supported by formula engine
- Application: GetFormulaOperators – Gets list of operator names supported by formula engine
- BoundingBoxXYZ: Centroid – Gets the bounding box center point
- BoundingBoxXYZ: Vertices – Gets list of bounding box vertices
- BoundingBoxXYZ: Volume – Evaluate bounding box volume
- BoundingBoxXYZ: SurfaceArea – Evaluate bounding box surface area
- Document: GetAllGlobalParameters – Returns all global parameters available in the given document
- Document: GetLightGroupManager – Gets a light group manager object from the given document
- Document: GetTemporaryGraphicsManager – Gets a TemporaryGraphicsManager reference of the document
- Document: GetAnalyticalToPhysicalAssociationManager – Gets a AnalyticalToPhysicalAssociationManager for this document
- Document: GetFamilySizeTableManager – Gets a FamilySizeTableManager from a Family
- UIApplication: CurrentTheme – Gets a current theme
- UIApplication: CurrentCanvasTheme – Gets a current canvas theme
- UIApplication: FollowSystemColorTheme – Indicate if the overall theme follows operating system color theme
- View: GetSpatialFieldManager – Retrieves manager object for the given view
Hope everyone enjoys the new release.
Thanks!
Made with love by [@Nice3point](https://t.me/nice3point).
#### RevitLookup 2025.0.6
[RevitLookup 2025.0.6](https://github.com/jeremytammik/RevitLookup/releases/edit/2025.0.6) implements:
- [Visualization dark theme support](https://github.com/jeremytammik/RevitLookup/issues/250)
- [Full changelog](https://github.com/jeremytammik/RevitLookup/compare/2025.0.5...2025.0.6)
#### RevitLookup 2025.0.7
[RevitLookup 2025.0.7](https://github.com/jeremytammik/RevitLookup/releases/edit/2025.0.7) implements
solid scaling, theme synchronisation with Revit and other improvements:
Solid scaling:
Visualisation now supports scaling a solid, relative to its centre.
Exploring small objects is now even easier, cf.,
[issue 251](https://github.com/jeremytammik/RevitLookup/issues/251):
![Solid scaling](img/geovis09solid2.png "Solid scaling")
Theme synchronisation with Revit:
Starting with Revit 2024, you can choose to automatically change the RevitLookup theme.
Fans of darker colors will no longer have to dig through the settings every time:
![Theme synchronisation](img/geovis10theme.png "Theme synchronisation")
Other improvements:
- Improved arrow position for vertical edges on visualization
- Multithreading visualization support. Changing settings now does not affect rendering. Previously there were artifacts due to fast settings changes
New `Element` extensions:
- GetCuttingSolids – Gets all the solids which cut the input element
- GetSolidsBeingCut – Get all the solids which are cut by the input element
- IsAllowedForSolidCut – Validates that the element is eligible for a solid-solid cut
- IsElementFromAppropriateContext – Validates that the element is from an appropriate document
#### Versions and Visualisation Wiki
- [RevitLookup versioning](https://github.com/jeremytammik/RevitLookup/wiki/Versions)
- [RevitLookup visualization](https://github.com/jeremytammik/RevitLookup/wiki/Visualization)