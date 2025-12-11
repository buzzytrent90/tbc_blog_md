---
ai_optimized: true
author: Jeremy Tammik
complexity_score: 5.7
content_type: qa
optimization_date: '2025-12-11T11:44:16.670733'
original_url: https://thebuildingcoder.typepad.com/blog/1752_precast_cloud_open.html
post_number: '1752'
reading_time_minutes: 4
series: general
slug: precast_cloud_open
source_file: 1752_precast_cloud_open.md
tags:
- elements
- revit-api
- sheets
- transactions
- views
title: Precast Cloud Open
word_count: 762
---

### Precast API and Cloud Open Callback
As usual, I am over-active in
the [Revit API discussion forum](http://forums.autodesk.com/t5/revit-api-forum/bd-p/160), so
you can see most of what I have been up to right there.
Here and now, I'll highlight one of those threads and clarify how to access the Revit 2019 Precast API:
- [Structural Precast API](#2)
- [`IOpenFromCloudCallback` and the `DefaultOpenFromCloudCallback` class](#3)
#### Structural Precast API
\*\*Question:\*\* I saw a slide on Precast for Revit:
![Precast API slide](img/precast_api_slide.jpg)
The slide indicates that Precast also exposes an API.
It sounds as if it has its own specific API, instead of the general Revit API.
Where can I find some information on this API, please?
\*\*Answer:\*\* Check out the blog posts discussing
the [Structural Precast Extension](https://blogs.autodesk.com/revit/tag/structural-precast-extension), especially
the [Autodesk Structural Precast Extension for Revit 2019](https://blogs.autodesk.com/revit/2018/05/09/new-structural-precast-extension-for-revit-2019).
It includes a section on the \*API for precast automation\*.
Its software installation folder \*%programfiles%/Autodesk/Structural Precast for Revit 2019\* provides a subfolder named `SDK` containing:
- A help file that listing all the exposed classes, methods and properties;
- An archive containing code samples
#### IOpenFromCloudCallback and the DefaultOpenFromCloudCallback Class
Next, let's highlight a question raised in
a [comment](https://thebuildingcoder.typepad.com/blog/2018/04/whats-new-in-the-revit-2019-api.html#comment-4464470775)
on [What's New in the Revit 2019 API](https://thebuildingcoder.typepad.com/blog/2018/04/whats-new-in-the-revit-2019-api.html) and
also in
the [Revit API discussion forum](http://forums.autodesk.com/t5/revit-api-forum/bd-p/160) thread
on [how to implement `IOpenFromCloudCallback` interface](https://forums.autodesk.com/t5/revit-api-forum/how-to-implement-iopenfromcloudcallback-interface/m-p/8794693):
\*\*Question:\*\* I am using the Revit API 2019 to detach a model from BIM360 with the following code:
```csharp
UIApplication app = commandData.Application;
string path = "BIM 360://myfile";
ModelPath modelPath = ModelPathUtils
.ConvertUserVisiblePathToModelPath( path );
OpenOptions openOptions = new OpenOptions();
openOptions.DetachFromCentralOption
= DetachFromCentralOption.DetachAndDiscardWorksets;
using( Transaction t = new Transaction( doc ) )
{
t.Start( "Open Cloud model" );
app.OpenAndActivateDocument( modelPath,
openOptions, true, IOpenFromCloudCallback );
t.Commit();
}
```
The problem is, I don't know how to implement `IOpenFromCloudCallback` correctly, so if anyone has experience with this, please help me.
Thank you so much :)
Later: I just figured out how to implement it like this:
```csharp
public Result Execute(
ExternalCommandData commandData,
ref string message,
ElementSet elements )
{
UIDocument uIDocument = commandData.Application.ActiveUIDocument;
UIApplication app = commandData.Application;
Document document = uIDocument.Document;
View currentview = document.ActiveView;
Guid projectid = Guid.Parse( "myProjectGUID" );
Guid modelid = Guid.Parse( "myModelGUID" );
ModelPath modelPath = ModelPathUtils
.ConvertCloudGUIDsToCloudPath( projectid, modelid );
OpenOptions openOptions = new OpenOptions();
IOpenFromCloudCallback openFromCloudCallback
= new cloudinterdace();
app.OpenAndActivateDocument( modelPath,
openOptions, true, openFromCloudCallback );
return Result.Succeeded;
}
public class cloudinterdace : IOpenFromCloudCallback
{
public OpenConflictResult OnOpenConflict(
OpenConflictScenario scenario )
{
throw new NotImplementedException();
}
}
```
\*\*Answer by Phil Xia:\*\* I am Phil from Revit engineering team.
I saw your sample and the exception thrown in your implementation of `IOpenFromCloudCallback`.
That is probably not a good choice.
You can use the default implementation provided by the `DefaultOpenFromCloudCallback` class that will always use the latest version for an open conflict.
Here is the introduction for this callback in the \*Revit API 2019 What's New\* sections
on [Existing APIs now support open from cloud paths (Collaboration for Revit)](https://thebuildingcoder.typepad.com/blog/2018/04/whats-new-in-the-revit-2019-api.html#4.1.5)
and [Callback for conflict cases when opening from a cloud path](https://thebuildingcoder.typepad.com/blog/2018/04/whats-new-in-the-revit-2019-api.html#4.1.5.2):
The callback method:
- Autodesk.Revit.DB.IOpenFromCloudCallback.OnOpenConflict()
can be passed to the document open methods to gain to handle conflict cases.
The method is passed an `OpenConflictScenario` value identifying the reason for the conflict (Rollback, Relinquished or OutOfDate) and should return an `OpenConflictResult` with the desired response (keep local changes, discard local changes and open the latest version or cancel).
The new class:
- DefaultOpenFromCloudCallback
provides a default way to handle conflicts: it always discards the local change and gets the latest version from the cloud.
The [`OpenConflictScenario` enumeration](https://www.revitapidocs.com/2019/7db711fa-cfb1-39da-a184-5aaf4230b660.htm) lists the values you can use to implement your handling logic to return `KeepLocalChanges`, `DiscardLocalChangesAndOpenLatestVersion` or just cancel the opening.
- Rollback – Central model is restored to an earlier version.
- Relinquished – Ownership to model elements is relinquished.
- OutOfDate – Model is out of date.
Many thanks to Phil for jumping in and clarifying!