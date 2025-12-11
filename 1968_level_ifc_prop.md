---
ai_optimized: true
author: Jeremy Tammik
complexity_score: 5.2
content_type: qa
optimization_date: '2025-12-11T11:44:17.153754'
original_url: https://thebuildingcoder.typepad.com/blog/1968_level_ifc_prop.html
post_number: '1968'
reading_time_minutes: 5
series: general
slug: level_ifc_prop
source_file: 1968_level_ifc_prop.md
tags:
- doors
- elements
- family
- filtering
- levels
- parameters
- references
- revit-api
- rooms
- schedules
- sheets
- views
- walls
- windows
title: Level Ifc Prop
word_count: 991
---

### Element Level and IFC Properties
Today, we mention another publisher of Revit API tutorials and add-ins, some tips on handling IFC, and recent discussions on controlling the level of BIM elements:
- [TwentyTwo add-ins and tutorials](#2)
- [IFC tips for APS and Forge](#3)
- [IFC custom properties in Revit](#4)
- [Set level id of existing element](#5)
- [Set level in NewFamilyInstance](#6)
#### TwentyTwo Add-Ins and Tutorials
According to its own mission statement,
[TwentyTwo](https://twentytwo.space) is creating forever free Autodesk add-ins that help you do more with less time and effort,
delivering efficient applications, as simple as possible, to handle tedious tasks and complex operations.
Besides the [TwentyTwo blog](https://twentytwo.space),
they also share add-ins and API tutorials for both
[Navisworks](https://twentytwo.space/navisworks-add-ins-development) and
[Revit](https://twentytwo.space/revit-add-in-development).
Many thanks to [Min Naung](https://twentytwo.space/author/mgjean) for putting together and sharing this material!
#### IFC Tips for APS and Forge
Wondering about your options when translating IFC model formats using the Autodesk Platform Services (APS), previously known as Forge?
Developer advocate Eason [@yiskang](https://twitter.com/yiskang) Kang put together
a comprehensive list of [FAQ and Tips for IFC translation of Model Derivative API](https://forge.autodesk.com/blog/faq-and-tips-ifc-translation-model-derivative-api) that
might help, including and not limited to:
- Overview of different available IFC conversion methods
- Georeferencing in IFC
- Troubleshooting locally
- Testing with Navisworks
- Testing with Revit
- 3rd-party IFC viewers
- Show All Presentations
#### IFC Custom Properties in Revit
Eason also recently addressed another important IFC related question:
\*\*Question:\*\* Can the Revit API be used to add custom properties in an IFC file opened in Revit?
Can Revit export this IFC with those new properties?
\*\*Answer:\*\* There is no direct way in Revit to add custom properties to IFC.
However, it can be achieved indirectly through the following steps:
- Open the IFC model with Revit’s OpenIFC
(API: [Application.OpenIFCDocument](https://www.revitapidocs.com/2023/bb14933b-a758-2b34-b160-686a28cc48cb.htm)) to
convert IFC to RVT
- Add customer properties by adding shared parameters and specifying values for them
([sample code](https://github.com/jeremytammik/FireRatingCloud/blob/master/FireRatingCloud/Cmd_1_CreateAndBindSharedParameter.cs) from
The Building Coder)
- Define custom Property Sets for IFC (here is a [tutorial video from 3rd-party](https://youtu.be/SswHKtcM3mI) or
check this [AKN page](https://knowledge.autodesk.com/search-result/caas/simplecontent/content/export-custom-bim-standards-and-property-sets-to-ifc.html))
- Specify the custom Property Sets in IFC export setup
(See [userDefinedPSets in Revit IFC repo](https://github.com/Autodesk/revit-ifc/blob/df1485b9accd598c2912a055af205ee1b03648c7/Source/IFCExporterUIOverride/IFCExportConfiguration.cs#L425) to
know how to construct IFCExportOptions for API)
- Export the modified RVT to IFC
(API: [Document.Export](https://www.revitapidocs.com/2023/7efa4eb3-8d94-b8e7-f608-3dbae751331d.htm))
![Custom property sets in IFC export setup](img/ifc_custom_property_1_export_setup.png "Custom property sets in IFC export setup")

Custom property sets in IFC export setup

![Demo-added custom prop `FM ID` in IFC](img/ifc_custom_property_2_added_fm_id.png "Demo-added custom prop `FM ID` in IFC")

Demo-added custom prop `FM ID` in IFC

![Imported IFC in Navisworks](img/ifc_custom_property_3_nw_import.png "Imported IFC in Navisworks")

Imported IFC in Navisworks

![Content of user defined property set](img/ifc_custom_property_4_content.png "Content of user defined property set")

Content of user defined property set

![Forge Viewer](img/ifc_custom_property_5_forge.png "Forge Viewer")

Forge Viewer

Many thanks to Eason for the useful explanation!
#### Set Level Id of Existing Element
Returning to pure desktop Revit API topics, several discussions recently in
the [Revit API discussion forum](http://forums.autodesk.com/t5/revit-api-forum/bd-p/160) circled
around setting the level of an existing element, e.g.:
\*\*Question:\*\* I'm placing a new face-based family instance into my Revit model with the help of the NewFamilyInstance method taking (Face, XYZ, XYZ, FamilySymbol).
This works fine, except the instance does not have its level set to that of the host; it's set to -1 in the API and just left blank in the UI.
I tried setting the level like such using the placed instance `LevelId` property and also tried setting its `BuiltInParameter` `FAMILY_LEVEL_PARAM`.
Both throw an error saying the parameter is read-only.
\*\*Answer:\*\* On some elements, the element level can only be set during the creation of the element.
For that, I would assume that you need to use a different [overload of the `NewFamilyInstance` method](https://www.revitapidocs.com/2017/0c0d640b-7810-55e4-3c5e-cd295dede87b.htm).
Please refer to this explanation by The Building Coder and a few recent discussions of related topics in the Revit API discussion forum:
- [Change level of existing element](https://thebuildingcoder.typepad.com/blog/2020/06/creating-material-texture-and-retaining-pixels.html#4)
- [LevelId is null](https://forums.autodesk.com/t5/revit-api-forum/levelid-is-null/m-p/11392692)
- [Change level on line based family](https://forums.autodesk.com/t5/revit-api-forum/change-level-on-line-based-family/m-p/10307454)
Another potentially helpful suggestion came up on the blog:
#### Set Level in NewFamilyInstance
Xikes shared a valuable observation in
their [comment](https://thebuildingcoder.typepad.com/blog/2011/01/family-instance-missing-level-property.html#comment-5925189938)
on [family instance missing `Level` property](https://thebuildingcoder.typepad.com/blog/2011/01/family-instance-missing-level-property.html):
For those who are still stuck with this problem even when using the correct overload:

```
  public FamilyInstance NewFamilyInstance(
    XYZ location,
    FamilySymbol symbol,
    Element host,
    StructuralType structuralType )
```

It is essential to pass in the function parameter `host` as a `Level` and not as an `Element`.
Add a quick cast like `(Level) myHostElement`.
It should do the trick.
The `Level` parameter is created properly and is not read-only.
Keep in mind that this will screw up the offset values, but you can adjust those afterwards.
It would be very helpful if other developers could confirm this observation.
Thank you.