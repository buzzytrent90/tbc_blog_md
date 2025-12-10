---
post_number: "1630"
title: "Adn Web Labs Update"
slug: "adn_web_labs_update"
author: "Jeremy Tammik"
tags: ['csharp', 'references', 'revit-api', 'sheets', 'views']
source_file: "1630_adn_web_labs_update.md"
original_url: "https://thebuildingcoder.typepad.com/blog/1630_adn_web_labs_update.html"
---

### Updated ADN Web Site, Revit API Labs, Tag Creation
Here are a couple of Revit API related updates to take note of:
- [Autodesk Developer Network ADN web site update](#2)
- [Revit Developer Centre update](#3)
- [ADN Revit API Training Labs update](#4)
- [Revit API Training Labs Xtra update](#5)
- [New top solution author record score](#6)
#### Autodesk Developer Network ADN Web Site Update
The [Autodesk Developer Network ADN web site](https://www.autodesk.com/developer-network)
at [www.autodesk.com/developer-network](https://www.autodesk.com/developer-network) has been updated.
New ADN Open pages are live now, and all the product- and API-specific URLs are working again.
Make sure to clean your cache when using them.
#### Revit Developer Centre Update
The ADN page of greatest interest to us here, of course, is
the [Revit Developer Centre](http://www.autodesk.com/developrevit)
at [www.autodesk.com/developrevit](http://www.autodesk.com/developrevit).
Check it out in its new guise.
![Revit Developer Centre](img/revit_developer_centre.png)
#### ADN Revit API Training Labs Update
I took this opportunity to migrate
the [ADN Revit API training labs](https://github.com/ADN-DevTech/RevitTrainingMaterial) to Revit 2018.
#### ADN Revit API Training Labs Xtra Update
I also finally eliminated all deprecated API usage warnings from
the [AdnRevitApiLabsXtra](https://github.com/jeremytammik/AdnRevitApiLabsXtra).
The last one to go was the deprecated usage of the
Autodesk.Revit.Creation.Document [`NewTag` method](http://www.revitapidocs.com/2018.1/ede92095-e554-9cc7-5286-a9d053466d1b.htm),
replaced by [`IndependentTag` `Create`](http://www.revitapidocs.com/2018.1/1f622654-786a-b8fd-1f81-278698bacd5b.htm).
Here are two code snippets from the relevant commit to the GitHub repository
to [eliminate deprecated API usage of the creation document `NewTag` method](https://github.com/jeremytammik/AdnRevitApiLabsXtra/commit/ec063bca10f168a2fb5870152495de94e67ef2f0) for
C# and VB:
C# for Revit 2017 API:
```csharp
IndependentTag tag = createDoc.NewTag(
view, inst, false, TagMode.TM_ADDBY_CATEGORY,
TagOrientation.Horizontal, midpoint ); // 2017
```
C# for Revit 2018 API:
```csharp
IndependentTag tag = IndependentTag.Create(
doc, view.Id, new Reference( inst ),
false, TagMode.TM_ADDBY_CATEGORY,
TagOrientation.Horizontal, midpoint ); // 2018
```
VB for Revit 2017 API:
```vbnet
Dim tag As IndependentTag = createDoc.NewTag(
view, inst, False, TagMode.TM_ADDBY_CATEGORY,
TagOrientation.Horizontal, midpoint) ' 2017
```
VB for Revit 2018 API:
```vbnet
Dim tag As IndependentTag = IndependentTag.Create(
doc, view.Id, New Reference(inst),
False, TagMode.TM_ADDBY_CATEGORY,
TagOrientation.Horizontal, midpoint) ' 2018
```
You should get your code ready for upcoming future versions as well.
Eliminating all warning messages is a fool-proof no-brainer first step to get started with that.
#### New Top Solution Author Record Score
I reached a new record score of 41 (as far as I know) as top solution author in
the [Revit API discussion forum](http://forums.autodesk.com/t5/revit-api-forum/bd-p/160):
![Top solution author record score](img/2018-03-01_top_solution_author_41.png)
I very much enjoy watching your scores rise, too, guys!