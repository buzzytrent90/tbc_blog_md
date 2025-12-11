---
ai_optimized: true
author: Jeremy Tammik
complexity_score: 2.5
content_type: documentation
optimization_date: '2025-12-11T11:44:16.653882'
original_url: https://thebuildingcoder.typepad.com/blog/1743_snoop_enhancement.html
post_number: '1743'
reading_time_minutes: 3
series: general
slug: snoop_enhancement
source_file: 1743_snoop_enhancement.md
tags:
- csharp
- doors
- elements
- filtering
- python
- revit-api
- rooms
- sheets
- views
- walls
- windows
title: Snoop Enhancement
word_count: 641
---

### Forge DA4R IFC Support and Snoop Enhancements
Let's close this eventful week with two important enhancements added to the Forge Design Automation API for Revit and our beloved RevitLookup tool:
- [IFC Support in the Design Automation for Revit API](#2)
- [RevitLookup Snoop Enhancements](#3)
#### IFC Support in the Design Automation for Revit API
Ryan Duell, Project Manager in The Factory just announced IFC support now added to
the [Forge Design Automation API for Revit](https://forge.autodesk.com/en/docs/design-automation/v3/developers_guide/overview), stating:
We recently added IFC support to Design Automation for Revit. The supported functionality for each Revit version is as follows:
- Revit 2018
- Open IFC
- Export IFC
- Revit 2019
- Open IFC
- Export IFC
- Link IFC
- Revit 2020 (coming soon)
- Open IFC
- Export IFC
- Link IFC
Additional documentation will be added asap.
Revit 2018 does not support link IFC functionality at this time.
This support will come in handy for my [IfcSpaceZoneBoundaries project](https://github.com/jeremytammik/IfcSpaceZoneBoundaries).
It needs to link in an IFC file into a Revit project to generate certain room and space information.
That was previously not possible in the Forge DA4R environment, and now it is.
Many thanks to Ryan and The Factory for this important step forward!
Please refer to The Building Coder topic group
for [more information on DA4R](https://thebuildingcoder.typepad.com/blog/about-the-author.html#5.55).
#### RevitLookup Snoop Enhancements
Alexander [@CADBIMDeveloper](https://github.com/CADBIMDeveloper) Ignatovich, aka Александр Игнатович,
shared several useful improvements for the RevitLookup snoop commands in
his [pull request #52](https://github.com/jeremytammik/RevitLookup/pull/52); he explains:
I added some improvements that I require in
the [RevitLookup tool](https://github.com/jeremytammik/RevitLookup) during
my development process. So here they are :-)
Added to the main results data window:
- Dependent elements
- View filter overrides and visibility
- Element state in different phases
First of all, I added the results of calling the `GetDependentElements` method to the main tool window.
I faced the situation when I explored the dependent elements of the dependent element (e.g., the dependency tree).
It was also useful to explore the dependent elements of the results by collecting the database elements using the `lookup` method in the Revit Iron Python Shell.
Further, I worked a lot with view filter overrides and visibility. I wanted to see these in the lookup tool. So, I added the `View.GetFilterOverrides` and `View.GetFilterVisibility` method results to the objects window.
For the last improvement, I explored the element statuses in different phases, so I also added this functionality as well.
Here is the complete list of changes:
- allow RevitLookup to snoop dependent elements via main interface
- preserve alphabetical methods order and simplify the code
- the first step to allow create "Objects" form elements with predefined titles
- introduce snoopable object wrapper to simplify the code
- extract method
- extract methods
- extract data factory and simplify the code of ElementMethodsStream class
- snoop view filter overrides settings
- colour components are not available for invalid colour values
- allow to snoop view filters visibility values
- snoop Element.GetPhaseStatus
- rename class
Some of these enhancement are hosted in new C# snooping modules:
- Snoop/CollectorExts
- DataFactory.cs
- Snoop/Data
- ElementPhaseStatuses.cs
- SnoopableObjectWrapper.cs
- ViewFiltersOverrideGraphicSettings.cs
- ViewFiltersVisibilitySettings.cs
Here is a sample model containing a wall with two dependent elements, a door and a window:
![Sample model with door in wall](img/snoop_dependent_model.png)
Snooping the wall data lists the two dependent elements:
![List of wall dependent element ids](img/snoop_dependent_wall.png)
Clicking on the list enables us to jump straight into the dependent element data:
![Snooping the door data](img/snoop_dependent_door.png)
Many thanks to Alex for his clean implementation and kind sharing of these powerful enhancements!
The new enhancements are integrated
in [RevitLookup release 2020.0.0.1](https://github.com/jeremytammik/RevitLookup/releases/tag/2020.0.0.1).