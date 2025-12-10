---
post_number: "2041"
title: "Hide Linked"
slug: "hide_linked"
author: "Jeremy Tammik"
tags: ['elements', 'filtering', 'references', 'revit-api', 'selection', 'sheets']
source_file: "2041_hide_linked.md"
original_url: "https://thebuildingcoder.typepad.com/blog/2041_hide_linked.html"
---

### Revit 2025 API Video and Hiding Linked Elements
The video recording on what's new in the Revit 2025 API has been released, and we discuss a nice example combining element pre-selection and `PostCommand`:
- [Revit 2025 API video](#2)
- [Hiding linked elements](#3)
#### Revit 2025 API Video
Boris Shafiro and Michael Morris present the video recording of the presentations
on [What's new in Autodesk Revit 2025 API](https://www.youtube.com/playlist?list=PLuFh5NgXkweMoOwwM2NlYmQ7FdMKPEBS_).
It consists of three parts:
- [Introduction and .NET 8 Migration](https://youtu.be/ONLf4BuGBU8) (7 minutes)
- [Breaking changes and removed API](https://youtu.be/huj3ynWwejA) (15 minutes)
- [New APIs and Capabilities](https://youtu.be/jExac5Kv-Qs) (40 minutes)
![Revit 2025 API video](img/rvt2025apivideo.png "Revit 2025 API video")
Please also refer to the following previous articles related to Revit 2025 API:
- [Revit 2025 and RevitLookup 2025](https://thebuildingcoder.typepad.com/blog/2024/04/revit-2025-and-revitlookup-2025.html)
- [The Building Coder Samples 2025](https://thebuildingcoder.typepad.com/blog/2024/04/the-building-coder-samples-2025.html)
- [RevitLookup Hotfix and the Revit 2025 SDK](https://thebuildingcoder.typepad.com/blog/2024/04/revitlookup-hotfix-and-the-revit-2025-sdk.html)
- [What's New in the Revit 2025 API](https://thebuildingcoder.typepad.com/blog/2024/04/whats-new-in-the-revit-2025-api.html)
- [Migrating VB to .NET Core 8](https://thebuildingcoder.typepad.com/blog/2024/05/migrating-vb-to-net-core-8-and-ai-news.html)
- [Revit 2025.1](https://thebuildingcoder.typepad.com/blog/2024/05/revit-20251-and-handling-lack-of-ui-in-da.html)
#### Hiding Linked Elements
Turning to a topic that has not yet been addressed by the new release, the open Revit Ideas wish list item
to [hide elements in linked model](https://forums.autodesk.com/t5/revit-ideas/hide-elements-in-linked-model/idc-p/12786934),
Lorenzo Virone shared a solution using `PostCommand` and element pre-selection via `Selection.SetReferences`.
He explains the detailed approach in
the [Revit API discussion forum](http://forums.autodesk.com/t5/revit-api-forum/bd-p/160) thread on how
to [hide elements in linked file](https://forums.autodesk.com/t5/revit-api-forum/hide-elements-in-linked-file/td-p/5777305):
This solution also demonstrates how to preselect elements for `PostCommand` processing:

```
// Select elements using UIDocument
// then use PostCommand "HideElements"
// elemsFromRevitLinkInstance is "List"
// these are the elements you want to hide in the link

var refs = elemsFromRevitLinkInstance.Select( x
  => new Reference(x).CreateLinkReference(revitLinkInstance))
    .ToList();

uidoc.Selection.SetReferences(refs);

uidoc.Application.PostCommand(
  RevitCommandId.LookupPostableCommandId(
    PostableCommand.HideElements));
```

\*\*Response:\*\* Can you provide a description of the process for using your code to hide elements only in a linked model?
I am not familiar with the API and deploying a script like this.
\*\*Answer:\*\*
Here is an sample that selects the first RevitLinkInstance, retrieve its floors and hides them:

```
// Get a link
var filter = new ElementClassFilter(typeof(RevitLinkInstance));

var firstInstanceLink
  = (RevitLinkInstance) new FilteredElementCollector(doc)
    .WherePasses(filter)
    .FirstElement();

// Get its floors
filter = new ElementClassFilter(typeof(Floor));
var elemsFromRevitLinkInstance
  = new FilteredElementCollector(
    firstInstanceLink.GetLinkDocument())
      .WherePasses(filter)
      .ToElements();

// Isolate them
var refs = elemsFromRevitLinkInstance.Select( x
  => new Reference(x).CreateLinkReference(firstInstanceLink))
    .ToList();

uidoc.Selection.SetReferences(refs);

uidoc.Application.PostCommand(
  RevitCommandId.LookupPostableCommandId(
    PostableCommand.HideElements));
```

Many thanks to Lorenzo for addressing this need and sharing his helpful solution!