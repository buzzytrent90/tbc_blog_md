---
ai_optimized: true
author: Jeremy Tammik
complexity_score: 1.8
content_type: documentation
optimization_date: '2025-12-11T11:44:16.682949'
original_url: https://thebuildingcoder.typepad.com/blog/1757_lookup_types_names.html
post_number: '1757'
reading_time_minutes: 3
series: elements
slug: lookup_types_names
source_file: 1757_lookup_types_names.md
tags:
- elements
- family
- parameters
- references
- revit-api
- sheets
- views
- walls
title: Lookup Types Names
word_count: 562
---

### SketchIt, Lookup Family Types, Definition Names
Today, we present yet another RevitLookup enhancement, a note on an undocumented built-in parameter change and a neat Forge Design Automation for Revit sample app:
- [RevitLookup family types and parameter definition names](#2)
- [Bitmap aspect ratio built-in parameter renamed](#3)
- [DA4R SketchIt demo generates walls](#4)
#### RevitLookup Family Types and Parameter Definition Names
Alexander [@aignatovich](https://forums.autodesk.com/t5/user/viewprofilepage/user-id/1257478) [@CADBIMDeveloper](https://github.com/CADBIMDeveloper) Ignatovich, aka Александр Игнатович,
submitted yet another useful [RevitLookup](https://github.com/jeremytammik/RevitLookup) enhancement
in [pull request #53 – available values for parameters (`ParameterType.FamilyType`) and `FamilyParameter` titles](https://github.com/jeremytammik/RevitLookup/pull/53).
In his own words:
> I added 2 improvements to the RevitLookup tool.
> The first is about available parameters values for parameters with `ParameterType` == `ParameterType.FamilyType`:
![RevitLookup lists family types](img/revitlookup_pull_request_53_1.png)
> We can retrieve these values using the `Family.GetFamilyTypeParameterValues` method.
The elements are either of class `ElementType` or `NestedFamilyTypeReference`:
![RevitLookup lists family types](img/revitlookup_pull_request_53_2.png)
> The second is very simple: Now the tool shows `FamilyParameter` definition names in the left pane:
![RevitLookup lists family types](img/revitlookup_pull_request_53_3.png)
Yet again many thanks to Alexander for his numerous invaluable contributions!
This enhancement is captured
in [RevitLookup release 2020.0.0.2](https://github.com/jeremytammik/RevitLookup/releases/tag/2020.0.0.2).
#### Bitmap Aspect Ratio Built-in Parameter Renamed
Rudolf [@Revitalizer](https://forums.autodesk.com/t5/user/viewprofilepage/user-id/1103138) Honke
and I completed a quick 20-km hilly run together yesterday.
This morning he pointed out an irritating change between the Revit 2019 and Revit 2020 APIs, an undocumented modification of the name of a built-in parameters defining the locked aspect ratio of a bitmap image, or \*Seitenverhältnis sperren von eingefügten Rasterbildern\* in German.
The underlying integer value remains unchanged, however, `-1007752`:
- Revit 2019: `BuiltInParameter.RASTER_MAINTAIN_ASPECT_RATIO`
- Revit 2020: `BuiltInParameter.RASTER_LOCK_PROPORTIONS`
Useful to know, just in case you happen to run into this yourself.
#### DA4R SketchIt Demo Generates Walls
I just noticed a neat
[Forge](https://forge.autodesk.com)
[Design Automation for Revit](https://forge.autodesk.com/en/docs/design-automation/v3/developers_guide/overview)
or [DA4R](https://thebuildingcoder.typepad.com/blog/about-the-author.html#5.55) sample
application created by my colleague
Jaime [@afrojme](https://twitter.com/AfroJme) Rosales,
[Forge Partner Development](http://forge.autodesk.com):
[SketchIt](https://github.com/Autodesk-Forge/design.automation-nodejs-sketchIt) is
a web application that enables the user to sketch out walls and floors in an SVG Canvas to later create and visualise them in an automatically generated RVT BIM model:
![SketchIt demo](img/jr_da4r_sketchit_demo.gif)
You can try it out live yourself in the [demo web page](https://sketchitapp.herokuapp.com).
This is a [node.js](https://nodejs.org) app demonstrating an end to end use case for external developers using Design Automation for Revit.
In addition to using Design Automation for Revit REST APIs, this app also leverages other Autodesk Forge services like Data Management API (OSS), the Viewer API and Model Derivative services.
The sketcher is built using Redux with React and makes extensive use of Flux architecture.
Main Parts
- Create a Revit Plugin to be used within AppBundle of Design Automation for Revit.
- Create your App, upload the AppBundle, define your Activity and test the workitem.
- Create the Web App to call the workitem.