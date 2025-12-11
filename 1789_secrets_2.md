---
ai_optimized: true
author: Jeremy Tammik
complexity_score: 4.4
content_type: qa
optimization_date: '2025-12-11T11:44:16.749212'
original_url: https://thebuildingcoder.typepad.com/blog/1789_secrets_2.html
post_number: '1789'
reading_time_minutes: 8
series: general
slug: secrets_2
source_file: 1789_secrets_2.md
tags:
- csharp
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
title: Secrets 2
word_count: 1586
---

### Secret Series 2, CAD Link Status and DWG Blocks
Joshua Lumley shares video #2 in his secret series, access to the CAD link status string and block in DWG export:
- [Secrets of Revit API coding part 2](#2)
- [Video 2 in the secret series](#3)
- [Getting CAD link status](#4)
- [Making blocks in Revit DWG export](#5)
#### Secrets of Revit API Coding Part 2
Last year, Joshua Lumley shared the recording he made for his BILT submission
on [five secrets of Revit API C# coding](https://thebuildingcoder.typepad.com/blog/2018/09/five-secrets-of-revit-api-coding.html).
In his [comment](https://thebuildingcoder.typepad.com/blog/2019/08/zero-touch-node-element-wrapper-and-load-from-stream.html#comment-4646680624)
on [Loading a .NET assembly from a memory stream](https://thebuildingcoder.typepad.com/blog/2019/08/zero-touch-node-element-wrapper-and-load-from-stream.html#3),
he points out part two, for this year's event:
> What really gets really tricky is when your add-in references another DLL not provided by Microsoft's .net framework.
> I made a 43-minute video on how to do it with the Xceed Extended.Wpf.Toolkit, [cf. below](#3).
> It also avoids 'double' loading.
#### Video 2 in the Secret Series
Joshua Lumley's video #2 in the Secret Series,
[Revit API C# make installer MSI file and avoid my blunders](https://youtu.be/S0MPxBRL7c0):

> In Support of BILT ANZ 2019 150-minute Lab by Joshua Lumley from Christchurch, New Zealand.
> Turn your Revit macro that was initiated from macro manager into a proper plugin installer (an MSI file) you can distribute to other computers with no fuss.
> Includes use of Nuget packages: Extended WPF Toolkit & Ookii Dialogues.
> In the hope you will learn from my mistakes – throughout this lab I share a few embarrassing moments, where I spent many hours solving issues that turned out to have very simple resolutions.
Many thanks to Joshua for his useful work and kind sharing!
#### Getting CAD Link Status
Two different threads in
the [Revit API discussion forum](http://forums.autodesk.com/t5/revit-api-forum/bd-p/160) ask
how to determine the loaded versus unloaded status of a CAD link, so let's summarise the answer here:
- [CAD link status](https://forums.autodesk.com/t5/revit-api-forum/cad-link-status/m-p/9075576)
- [How to get the status of the Revit link?](https://forums.autodesk.com/t5/revit-api-forum/how-to-get-the-status-of-the-revit-link/td-p/9072787)
\*\*Question:\*\* I'm trying to get the status of my linked CAD files through Revit API; more specifically, I want to list all the CAD links that were not found.
I know that I'm dealing with `ImportInstance`, I know that I can use `.IsLinked` to determine if it was linked or imported.
I can't find any way to get the status info, and even finding a path to the original file seems impossible.
Has any of you dealt with this before? Is it even possible?
By using the method `RevitLinkType.IsLoaded`, I can only tell whether or not the link is loaded (True or False).
Is there a way to get the Status string?
![CAD links not loaded](img/cad_links_not_loaded.png)
\*\*Answer:\*\* I believe that the solution can be found in the discussion
on [automatically reloading links after migration](https://thebuildingcoder.typepad.com/blog/2016/08/automatically-reload-links-after-migration.html).
It includes statements like, \*It took a bit of time to find the right classes as there are no less than four levels of indirection to get from RevitLinkType to the 'Saved Path' and then use that in the call to LoadFrom.\*
Maybe
he [`ExternalFileReference.GetLinkedFileStatus` method](https://www.revitapidocs.com/2020/cd21f80a-f8be-535a-0793-7c113f27c487.htm) provides
what you need.
Try using this code:
```csharp
FilteredElementCollector linktypes
= new FilteredElementCollector( doc )
.OfClass( typeof( CADLinkType ) )
.WhereElementIsElementType();
foreach( Element e in linktypes )
{
ExternalFileReference efr
= e.GetExternalFileReference();
string linkStatus = efr
.GetLinkedFileStatus().ToString();
}
```
\*\*Response:\*\* Fantastic! Thank you both for constructive replies!
#### Making Blocks in Revit DWG Export
A recent discussion on how to ensure that family instances in Revit are exported to DWG blocks, or, alternatively, how to reassemble together block from them after the export:
\*\*Question:\*\* I have a Revit plugin that exports to DWG and an AutoCAD plugin that imports the newly created DWG.
For Revit exports, some Revit family instances are exported as 1-1 Fam Instance->Blocks, while other Revit family instances with similar geo complexity are exported as a collection of individual lines, arcs, squares, etc. – not a block.
I’m trying to find a way to tell Revit to export family instances as blocks always but unsure if this is possible.
Also unsure if I can add a property to a Revit fam instance inside Revit in a way that when it exports to the DWG as multiple primitives, I could cleverly reassemble the primitives as a block.
Could anyone help me identify if either API workflow is possible?
\*\*Answer:\*\* I assume that if there options, they are not just exposed to the API, but also in the UI.
\*\*Response:\*\* I'm attempting everything as a user first.
The settings for export don't have a block setting (only solids vs. mesh setting, have used both).
I'm not able to add properties that carry over to let me identify objects in AutoCAD.
For example, element id's don't come in as attributes, so I don't have any logic that can sweep up primitives and assemble as a block.
Also, I have fam instances like electrical fixtures that come in as several arcs, lines and annotation layered vs. single block with geo layer, so I'm not able to identify a programmatic way to assemble the anno, primitive, elec fixtures and turn them into a single block as geo layer since there are no mapping id's.
I've used Revit Snoop and Acad MgdDbg to look under the hood for any matching id's between outgoing element and incoming objects, but they don't exist.
I found id's in acad object `XData` but they are not related to element id's either.
Has anyone created an alternative export tool that gives more control to modify objects programmatically after import?
Maybe I just export all geometry myself in Revit with unique id's and metadata in json/xml and construct things on the import side?
Is that dumb?
\*\*Answer:\*\* The DWG export algorithm uses the geometry that the instance object is producing for drawing on screen and based on the types of primitives will export it as a block or not.
Most families produce a `GInstance` nodes, which behaves like a block in AutoCAD.
It is a collection of geometric primitives (lines, arc etc.) and a `Transformation` that defines the coordinates.
This type of node is exported as a block.
As you observed, there are families that produce only geometric primitives and the exporter doesn't consider them as a block.
Unfortunately, I don't have any advice to you. It seems that needs to be implemented on Revit side.
The families that are exported as lines and arcs should construct their representation as a `GInstance`.
I'm not sure this is doable.
I’m also thinking about the solutions with ids.
I remember that each AutoCAD entity's xdata contains the `ElementId` from the Revit element.
You can see this using the `xdlist` command in AutoCAD.
For families exported as blocks, you should run this command on the block reference, not on the entities that compose the block – these don’t have xdata.
Can you give me an example where this rule doesn’t apply? I remember that there were some issues for elements that comes from linked files (they have ids but from a different document).
Why do you group AutoCAD entities same as the Revit entities? What are you trying to achieve?
I looked at your sample files and I understood the issue.
It’s about nested families in certain conditions – the nested Family is 'shared' and the nested family is an annotation family while the main one isn't.
In your example the symbol (shared family) has a mark that is different from the mark of family instance containing it.
It looks like the intent is to schedule the two components differently, even though in Revit they are placed using the same family.
If this is the case, you can add a logic to group all the primitives from the shared family into its own block.
You can look at each entity’s xData where you will find ID of element that produced that entity.
I don’t know of a way to get the ID of the family instance and tie it to all the shared families contained, in case you want a block containing all the entities from the family and shared families.
\*\*Response:\*\* This is great, thank you!
One other easy'ish question: What do the other xdata index values indicate?
![Revit DWG export xdata](img/rvt_dwg_export_xdata.png)
\*\*Answer:\*\* Here are the codes that we use, cf.
the [DWG and DXF export `Xdata` specification](https://thebuildingcoder.typepad.com/blog/2010/08/dwg-and-dxf-export-xdata-specification.html):
```csharp
// XData Identifiers
#define XDATA_ELEMENT_ID 1
#define XDATA_CATEGORY_ID 2
#define XDATA_SUB_CATEGORY_ID 3
#define XDATA_MATERIAL_ID 4
#define XDATA_TYPE_ID 5
#define XDATA_IS_MATERIAL_OVERRIDDEN_BY_FACE 6
// Room boundary and area XDATA
#define XDATA_ROOM_NAME_ID 101
#define XDATA_ROOM_NUMBER_ID 102
#define XDATA_ROOM_OCCUPANCY_ID 103
#define XDATA_ROOM_DEPARTMENT_ID 104
#define XDATA_ROOM_COMMENTS_ID 105
```