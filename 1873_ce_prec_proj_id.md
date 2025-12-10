---
post_number: "1873"
title: "Ce Prec Proj Id"
slug: "ce_prec_proj_id"
author: "Jeremy Tammik"
tags: ['elements', 'filtering', 'geometry', 'levels', 'parameters', 'references', 'revit-api', 'sheets', 'views', 'windows']
source_file: "1873_ce_prec_proj_id.md"
original_url: "https://thebuildingcoder.typepad.com/blog/1873_ce_prec_proj_id.html"
---

### Custom Export Precision, Sheet Metadata, Project Id
I lit upon many interesting topics in the past few days, on pure Revit API, Forge, BIM360 and AI:
- [Custom export precision](#2)
- [Dismissing a Windows dialogue with JtClicker](#3)
- [AU classes for construction customers](#4)
- [Retrieve sheet metadata in Forge viewer](#5)
- [Determining the BIM 360 project id](#6)
- [AI solves partial differential equations](#7)
- [AI-enhanced video editing](#8)
#### Custom Export Precision
[Sunsflower](https://forums.autodesk.com/t5/user/viewprofilepage/user-id/3074901) took
another look at improving the precision of a custom exporter in
the [Revit API discussion forum](http://forums.autodesk.com/t5/revit-api-forum/bd-p/160) thread
on [CustomExporter Export Very Jagged Mesh for Curved Surfaces](https://forums.autodesk.com/t5/revit-api-forum/customexporter-export-very-jagged-mesh-for-curved-surfaces/td-p/9820131):
\*\*Question:\*\* As shown in the screenshots below, when I tried to export a curved surface, the `OnPolyMesh` method in `IExportContext` produces very jagged edges:
![Jagged edges](img/custom_export_precision_1.png "Jagged edges")
![Smooth edges](img/custom_export_precision_2.png "Smooth edges")
Is there a way to improve this?
\*\*Answer:\*\* Check out The Building Coder topic group on
the [custom exporter](https://thebuildingcoder.typepad.com/blog/about-the-author.html#5.1).
Especially, please read these two posts:
- [Revit export precision and tolerance](https://thebuildingcoder.typepad.com/blog/2015/06/angelhack-athens-sustainability-and-export-precision.html#4)
- [Controlling the quality of the geometry on custom export](https://thebuildingcoder.typepad.com/blog/2016/02/reorg-fomt-devcon-ted-qr-custom-exporter-quality.html#8)
I all else fails, you will have to access the real element geometry instead of using the custom exporter.
That is normally a lot more work, though.
\*\*Response:\*\* My solution is to triangulate the face in the `OnFace` method.
This way, I can input a `LevelOfDetail` parameter.
However, I lose all `UV` data at the same time.
Currently, I use `Face.Project` to approximate a set of `UV`, which is quite unstable.
I also tried to set the `LevelOfDetail` property on `ViewNode`, and it also works.
#### Dismissing a Windows Dialogue with JtClicker
Another topic group is dedicated
to [detecting and handling dialogues and failures](https://thebuildingcoder.typepad.com/blog/about-the-author.html#5.32).
It started out before the [DialogBoxShowing event](https://www.revitapidocs.com/2020/cb46ea4c-2b80-0ec2-063f-dda6f662948a.htm) and
[Failure handling APIs](https://www.revitapidocs.com/2020/c03bb2e5-f679-bf24-4e87-08b3c3a08385.htm) were implemented, using a Windows hook to determine that a dialogue was being shown:
\*\*Question:\*\* You explained how to use the native Windows API hook
to [dismiss a dialogue](https://thebuildingcoder.typepad.com/blog/2009/10/dismiss-dialogue-using-windows-api.html).
Is there a complete sample project and solution available to understand how to use it to dismiss the dialogue box in Revit?
\*\*Answer:\*\* Whenever searching for such information, one of the first places to go
are [The Building Coder topic groups](https://thebuildingcoder.typepad.com/blog/about-the-author.html#5).
In this case, you can look at [detecting and handling dialogues and failures](https://thebuildingcoder.typepad.com/blog/about-the-author.html#5.32).
The Windows hook functionality is not really used 'in Revit', as far as I can remember.
It is independent functionality that can interact with a Revit add-in, if you like.
The complete project is available in
the [JtClicker repository on GitHub](https://github.com/jeremytammik/JtClicker).
#### AU Classes for Construction Customers
Are you specifically interested in construction?
Check out the overview
of [AU classes for construction customers](https://forge.autodesk.com/blog/forge-au-classes-construction).
#### Retrieve Sheet Metadata in Forge Viewer
Now, let's turn to Forge.
Here is a pretty illuminating exploration on accessing Revit sheet metadata in that environment:
\*\*Question:\*\* I am trying to retrieve the 'Identity Data' of a sheet in a Revit model using the Model Derivative API:
![Sheet identity data](img/sheet_identity_data.png "Sheet identity data")
Unfortunately, I was unable to find this info anywhere in the properties.
Is it possible, or do I have to use Design Automation for this?
Thanks!
\*\*Answer:\*\* It should be possible to get sheet properties by navigating the hierarchy of the object tree.
The root node (`id` = `1`) is the document, and the sheets will be listed as children of that root.
One would need to iterate through the children of the root to get the properties of the sheets and views.
\*\*Response:\*\* I do see the model as `id` = `1`, but I can't find sheets as children of that root.
I also do not see `Sheet` in the model browser in the Forge viewer in any example (which certainly contain Revit models having sheets).
Am I overlooking something?
\*\*Answer:\*\* The model browser will not show sheets, because they do not have physical geometry associated with them.
However, there will be sheet objects as children of the root.
Maybe, in some RVTs, sheets do not appear in the property database, though.
It probably depends on which API you use to get properties.
I'm referring to the full property database available to the Forge viewer.
\*\*Response:\*\* This is an export of the tree of `/metadata` for my Revit model:

```
Untitled
{
"data": {
  "type": "objects",
  "objects": [
      {
      // ... (2,059 lines)
```

Here are the `/properties`:

```
Untitled
{
"data": {
  "type": "properties",
  "collection": [
      {
      // ... (20,859 lines)
```

These come from a sheet called "A102 - Plans".
\*\*Answer:\*\* I don't know what subset of element properties are returned by the Forge properties API.
I do know that the Forge viewer will show sheet properties in some cases, e.g., in BIM360 Docs when you open the sheet.
\*\*Response:\*\* Yes indeed, BIM 360 Docs does show properties of sheets.
I checked and confirmed.
Now I wonder how it gets those properties.
\*\*Answer:\*\* Just like I said – it loops through the children of the root node and finds the sheet element with the matching name.
However, it's not using the Forge properties API.
It uses the raw property data, available to the Forge Viewer
\*\*Response:\*\* I kind of understand what you say.
I understand that the properties are being retrieved by the raw property data.
However, to first select the element id, the hierarchy (from the `/metadata` endpoint) should retrieve sheets, right?
I don't see sheets in that response; or is it that there's also other raw data which is different from the Forge metadata API?
\*\*Answer:\*\* The Forge metadata endpoint is not raw, it's processed data.
From the above, it looks like it's missing the child properties that will let you easily find the sheets from the root element.
\*\*Response:\*\* Thanks, this is very helpful.
Final question: can this raw data be accessed by a customer?
\*\*Answer:\*\* Yes, using Forge Viewer.
It may be possible to get this information via metadata somehow that I am not aware of.
\*\*Response:\*\* Hmm... so, if I want to query and fetch attribute values, that won't be possible using Forge viewer, right?
\*\*Answer:\*\* The MD service does let you perform queries to get the metadata you want, with two choices of data format.
If you run into data that MD does not collect, and Revit Design Automation would be your fallback.
Here is an example accessing additional metadata,
to [extract compound structure layer from RVT files using Design Automation for Revit](https://github.com/augustogoncalves/forge-customproperty-revit).
The resources listed for the [Forge at AU 2020 pre-event online bootcamp](https://forge.autodesk.com/blog/forge-au-2020-pre-event-online-bootcamp) will probably also be useful for you.
#### Determining the BIM 360 Project Id
Kevin Augustino very kindly shared his current approach
to [retrieve the BIM 360 Document Management Project Id of the active Revit cloud model](https://forums.autodesk.com/t5/revit-api-forum/bim-360-document-management-project-id-of-revit-cloud-model/m-p/9830419):
\*\*Question:\*\* How can I retrieve the BIM 360 Document Management Project Id of the active Revit model?
I'm aware of \*Document.GetCloudModelPath().GetProjectGUID()\*, but this seems to be a C4R Project Id.
I need the Document Management Id to interface with
the [Forge BIM 360 and Data Management APIs](https://forge.autodesk.com/en/docs/bim360/v1/reference/http/).
So far, I've found that the Document Management file has an attribute that matches the C4R Project Guid: \*attributes.extension.data.projectGuid\*.
So, I need to find the Docs project that contains a file such that:

```
  attributes.extension.data.projectGuid
    = .GetCloudModelPath().GetProjectGUID().
```

But surely there's a better approach than doing a [folder search](https://forge.autodesk.com/en/docs/data/v2/reference/http/projects-project_id-folders-folder_id-search-GET/) using a filter matching \*filter[attributes.extension.data.projectGuid]\* with `ValueFromCloudModelPath` on every Docs Project that my Forge App has access to?
\*\*Answer:\*\* I asked the development team for you whether they can suggest a better way.
They are currently discussing the implementation of a direct method to retrieve the BIM 360 project id of the document via a property such as `Document.ProjectId`, now as we speak. It will hopefully be available in a future release of Revit.
Meanwhile, the convoluted approach you describe sounds significantly better than nothing at all to me, so well done finding a way through the maze.
\*\*Response:\*\* For anyone else who runs into this same need, here are some of my other findings:
`Document.PathName` seems to be a string in this form when opening a cloud model:

```
  BIM 360:///.rvt
```

So, another option is to try parsing `Document.PathName` to get the Document Management Project name:

```
  string regexPattern =
    @"^BIM 360:\/\/(?.*)\/(?.*)$";

  if (Regex.IsMatch(doc.PathName, regexPattern))
  {
    Match match = Regex.Match( doc.PathName, regexPattern );
    string projectName = match.Groups["ProjectName"].Value;
  }
```

Then look for a project with that name by iterating each hub returned
from \*https://forge.autodesk.com/en/docs/data/v2/reference/http/hubs-GET/\*,
and, on each one, try
to [get a project using a name filter](https://forge.autodesk.com/en/docs/data/v2/reference/http/hubs-hub_id-projects-GET/),
using a filter such as

```
  string.format( "?filter[attributes.name]={0}",
    HttpUtility.UrlEncode(projectName))
```

If this project name isn't unique, then this approach might not get the correct one.
But additional processing can be applied to use a folder search looking for

```
  attributes.extension.data.projectGuid
    = .GetCloudModelPath().GetProjectGUID()
```

So at least this way, the folder search is only done on potential matches, rather than every single project.
If the Document Management project name changes, then `Document.PathName` won't refresh to the new project name until you re-save the model.
So, as a fallback, if I still haven't found the project Id, I resort to the folder search on every project regardless of name.
Not ideal, but hopefully a direct method will be added to the Revit API in the future!
Many thanks to Kevin for all his research and documentation work on this!
#### AI Solves Partial Differential Equations
[AI has cracked a key mathematical puzzle for understanding our world](https://www.technologyreview.com/2020/10/30/1011435/ai-fourier-neural-network-cracks-navier-stokes-and-partial-differential-equations):
> Partial differential equations can describe everything from planetary motion to plate tectonics, but they’re notoriously hard to solve...
> They can be used to model everything from planetary orbits to plate tectonics to the air turbulence that disturbs a flight, which in turn allows us to do practical things like predict seismic activity and design safe planes...
> PDEs are notoriously hard to solve...
> Researchers at Caltech have introduced a new deep-learning technique for solving PDEs,
a [Fourier Neural Operator for Parametric
Partial Differential Equations](https://arxiv.org/pdf/2010.08895.pdf)
... dramatically more accurate... much more generalizable ... 1'000 times faster ...
#### AI-Enhanced Video Editing
Here is another example of AI usage that may come in handier to you right away than solving differential equations:
AU is coming up. Are you possibly thinking about recording a video?
Check out [Descript](https://www.descript.com) before you do.
It is a collaborative audio and video editor that includes transcription, a screen recorder, publishing, full multitrack editing, and some mind-bendingly useful AI tools:
- [Blog post](https://medium.com/descript/introducing-descript-fa37eb193819)
- [Video](https://youtu.be/Bl9wqNe5J8U):