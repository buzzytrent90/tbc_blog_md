---
post_number: "1785"
title: "Dmu Loop Project Param"
slug: "dmu_loop_project_param"
author: "Jeremy Tammik"
tags: ['elements', 'family', 'filtering', 'levels', 'parameters', 'revit-api', 'sheets', 'transactions', 'views']
source_file: "1785_dmu_loop_project_param.md"
original_url: "https://thebuildingcoder.typepad.com/blog/1785_dmu_loop_project_param.html"
---

### Get Project Parameter Id and Prevent Updater Loop
Let me once again highlight two helpful answers
by Frank [@Fair59](https://forums.autodesk.com/t5/user/viewprofilepage/user-id/2083518) Aarssen in
the [Revit API discussion forum](http://forums.autodesk.com/t5/revit-api-forum/bd-p/160)
and the newest pair of Forge Design Automation samples:
- [Get project parameter id from its name](#2)
- [Preventing an updater loop](#3)
- [New Forge and Design Automation samples](#4)
#### Get Project Parameter Id from its Name
Frank shows how to retrieve a project parameter id given its name in the thread
on [creating view filters for a project parameter](https://forums.autodesk.com/t5/revit-api-forum/create-view-filters-for-project-parameter/m-p/9051132):
\*\*Question:\*\* Occasionally, we have to make multiple view filters in a model based on a project parameter.
I am trying to put together a macro that I can modify when needed to help me create these filters.
However, I am struggling with being able to select any parameters that are not built-in to create to filters.
How can I find the parameter id when I only have the name?
\*\*Answer:\*\* Project parameters are 'document-based; parameters and are stored in a (shared) `ParameterElement`.
Given a name, this is how you find the `Id`:
```csharp
ElementId GetProjectParameterId(
Document doc,
string name )
{
ParameterElement pElem
= new FilteredElementCollector( doc )
.OfClass( typeof( ParameterElement ) )
.Cast()
.Where( e => e.Name.Equals(name) )
.FirstOrDefault();
return pElem ?.Id;
}
```
On closer inspection, I noticed that this is basically a repetition of part of an earlier answer by Frank to a previous thread
on [deleting a non-shared project parameter](https://forums.autodesk.com/t5/revit-api-forum/deleting-a-non-shared-project-parameter/td-p/5975020).
#### Preventing an Updater Loop
Frank solves an infinite looping problem in the thread
on [IUpdater get family instance and update values](https://forums.autodesk.com/t5/revit-api-forum/iupdater-get-family-instance-and-update-values/m-p/9053785):
\*\*Question:\*\* My goal: Update family parameter values from the API when shape handles are pulled.
What works: `IUpdater` class that fires off my custom code when a user manipulates the shape handles on a family.
What doesn't work: updating the family parameters.
I have an `IUpdater` class that is fired only when family instances are changed.
My problem is that it doesn't seem to update the parameter, and it loops infinitely (until Revit crashes).
My end goal here is to get the family whose shape handle has been pulled, loop through each of its parameters and set a value, in this example just setting "4", but I have more advanced match being applied that is currently disabled while I figure this out.
\*\*Answer:\*\* When you say, 'Revit is looping', it might be that the modification you apply in the updater Execute method triggers the same updater again, starting an infinite loop.
There are different levels of triggers and updaters. You may want to modify those so that the changes you make do not re-trigger your updater.
Alternatively, you might be able to add a switch to your updater Execute so that it is only active once and deactivated when it gets called a second time in the same transaction, to step out of the infinite loop.
Maybe some of
the [other DMU samples](https://thebuildingcoder.typepad.com/blog/about-the-author.html#5.31) will
help resolve the problem.
Frank shares a simpler suggestion:
To prevent the loop, I usually test to see if the new value differs from the old value:
```csharp
string newValue = "4";
if( param.AsValueString() != newValue )
{
param.SetValueString( newValue );
}
```
Ever so many thanks to Frank for his endless flow of extremely insightful answers!
#### New Forge and Design Automation Samples
I do not discuss [Forge Design Automation for Revit](https://thebuildingcoder.typepad.com/blog/about-the-author.html#5.55) or
DA4R as much as I would like, since there are still more than enough issues to be handled dealing with the pure Revit desktop API, and my other Forge Platform Development colleagues are more specialised on the web-based issues.
I would still like to highlight the continuous succession
of [new Forge samples](https://forge.autodesk.com/categories/code-samples),
recently including [communication with servers from inside Design Automation](https://forge.autodesk.com/blog/communicate-servers-inside-design-automation)
and a [Design Automation for Revit sample demonstrating parameter export and import with Excel](https://forge.autodesk.com/blog/design-automation-revit-parameters-export-import-sample-excel).
![Design Automation server communication](img/user_da_http.png)