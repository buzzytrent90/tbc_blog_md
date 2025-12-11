---
ai_optimized: true
author: Jeremy Tammik
complexity_score: 5.9
content_type: qa
optimization_date: '2025-12-11T11:44:17.381329'
original_url: https://thebuildingcoder.typepad.com/blog/2057_filter_type.html
post_number: '2057'
reading_time_minutes: 8
series: filtering
slug: filter_type
source_file: 2057_filter_type.md
tags:
- elements
- family
- filtering
- parameters
- references
- revit-api
- schedules
- sheets
- transactions
- views
title: Filter Type
word_count: 1594
---

### Unload Links Offline and Filter for Types
Let's look at some standard filtering questions and a typical use of transmission data:
- [Parameter filter to delete viewport type](#2)
- [Cannot delete type in use](#3)
- [Determine coordination model filepath](#4)
- [Unload links with transmission data](#5)
#### Parameter Filter to Delete Viewport Type
Sometimes, the BIM element that you need to access is hard to identify and filter out.
Some elements are not simply identifiable by category or class.
[Sean Page](https://forums.autodesk.com/t5/user/viewprofilepage/user-id/3064449) shares
an effective approach to filter for such an element in the thread
on [deleting unpurgeable viewport types through API](https://forums.autodesk.com/t5/revit-api-forum/deleting-unpurgeable-viewport-types-through-api/m-p/12741221):
\*\*Question:\*\* When you copy views (including schedules) from one project to another, it brings in the default viewport type of that project.
That viewport type becomes unpurgeable.
So if you do this with enough files, you get a whole bunch of unpurgeable viewport types in your project.
We have a few of these unpurgeable viewport types in our template and I'm looking for a way to clean this up without being forced to recreate the template, which would take me weeks.
Anyone any ideas on how I could achieve this through the API (if possible at all)?
\*\*Answer:\*\*
Here is what I use to get Viewport Types.
It uses a parameter filter to return all element types whose SYMBOL_FAMILY_NAME_PARAM equals "Viewport".
It may not be super language stable, but it works great for me:

```
  FilterRule rule
    = ParameterFilterRuleFactory.CreateEqualsRule(
      new ElementId( (Int64)
        BuiltInParameter.SYMBOL_FAMILY_NAME_PARAM),
      "Viewport");

  ElementParameterFilter filter = new ElementParameterFilter(rule);
  IList viewportTypes;

  using( FilteredElementCollector fec
    = new FilteredElementCollector(doc)
      .OfClass( typeof(ElementType) )
      .WherePasses(filter) )
  {
    viewportTypes = fec.Cast().ToList();
  }
  return viewportTypes;
```

Many thanks to Sean for the nice and effective solution.
#### Cannot Delete Type in Use
A similar filter comes up in the discussion
[is it possible to delete arrowhead types?](https://forums.autodesk.com/t5/revit-api-forum/is-it-possible-to-delete-arrowhead-types/td-p/13025122)
It uses post-processing on the `FamilyName` property instead of the more efficient parameter filter, so Sean's solution above could be used to further improve it:
\*\*Question:\*\* This code throws and exception and crashes:

```
var arrowHeads = new FilteredElementCollector(doc)
  .WhereElementIsElementType()
  .Cast()
    .Where(t => t.FamilyName == "Arrowhead")
    .ToList();

using (Transaction trans = new Transaction(doc,
  "Remove Arrowhead"))
{
  trans.Start();
  foreach (var arrowHead in arrowHeads)
  {
    doc.Delete(arrowHead.Id);
  }
  trans.Commit();
}
```

It throws an `Autodesk.Revit.Exceptions.InternalException` saying \*An internal error has occurred\* and then terminates Revit.
\*\*Answer:\*\*
Maybe the type is in use.
You can't delete system types, only purge them, cf.:
- [Purge unused via API](https://forums.autodesk.com/t5/revit-api-forum/purge-unused-via-the-api/td-p/6431564)
- [Purge and `PostCommand`](https://thebuildingcoder.typepad.com/blog/2017/11/purge-and-detecting-an-empty-view.html#2)
- [Purge unused with `eTransmit`](https://thebuildingcoder.typepad.com/blog/2022/03/purge-unused-and-the-autodesk-camel.html)
![Purge unused](img/arrowhead_purge.png "Purge unused")
You can determine whether the Arrowhead type is deletable or not by using the [`CanBeDeleted` property](https://www.revitapidocs.com/2015/5efe8253-d555-00c2-8db6-9114e328fcc7.htm).
This will help prevent Revit from crashing.
\*\*Response:\*\*
`CanBeDeleted` returns true for all elements.
\*\*Answer:\*\*
Try to use `IFailurePreProcessor` to handle Revit based errors.
If a serious error occurs, you can skip the process without crashing Revit.
I tested deleting all arrowhead types and was able to remove all except "Arrow Open 90 Degree 1.25mm".
Each attempt to delete this specific arrowhead type causes an exception and crash.
Could you please verify if the same arrowhead type is causing the issue on your end?
\*\*Response:\*\*
Yes. It is.
\*\*Answer:\*\*
The "Arrow Open 90 Degree 1.25mm" arrowhead type is being used by a component.
Upon reviewing the journal file, I discovered that that it is internally utilised by `StairsPathType`, `setArrowheadTypeId`.
This explains why Revit throws an exception when attempting to delete it.
Many thanks to [Mohamed Arshad](https://forums.autodesk.com/t5/user/viewprofilepage/user-id/8461394)
and [Naveen Kumar Thalaivirichan](https://forums.autodesk.com/t5/user/viewprofilepage/user-id/5661631)
for this clarification.
#### Determine Coordination Model Filepath
Adressing another filtering and data access task,
[Leandro Arns Gonzales](https://stackoverflow.com/users/13294294/leandro-arns-gonzales) shares a solution
for [getting coordination model filename and path in active document](https://stackoverflow.com/questions/79055736/getting-coordination-models-filename-and-path-in-active-document):
\*\*Question:\*\*
I am trying to get the coordination model filenames and paths in my active document.
I tried using `RevitLinkInstance` elements, but it is not a Revit link, so returns an empty list.
I tried using `OfCategory(OST_Coordination_Model)`, but it still gives me zero elements.
I finally found it using `OfClass(typeof(DirectShapeType))`.
I am able to retrieve its name, but I am still struggling to get the file location.
\*\*Answer:\*\*
I managed to find a way.
The path is stored in the built-in parameter `DIRECTCONTEXT3D_SOURCE_ID`.
This is how I ended up getting my coordination model file name and location:

```
  FilteredElementCollector collectorTwo
    = new FilteredElementCollector(doc);

  ICollection revitLinksShape
    = collectorTwo.OfClass(typeof(DirectShapeType))
      .ToElements();

  // Itera por todas as referências externas
  // para buscar modelos de coordenação

  foreach (var fileRef in revitLinksShape)
  {
    var directShapeType = fileRef as DirectShapeType;

    if(directShapeType != null)
    {
      Parameter coordModelPathParam = directShapeType.get_Parameter(BuiltInParameter.DIRECTCONTEXT3D_SOURCE_ID);
      if (coordModelPathParam != null)
      {
        // Get the file path from the parameter
        string filePath = coordModelPathParam.AsString();

        // Display the file path
        TaskDialog.Show("Coordination Model Path", $"File Path: {filePath}");
      }
    }
  }
```

Many thanks to Leandro for raising this and sharing their solution.
#### Unload Links with Transmission Data
Let's wrap up with a solution
showing [how to open Revit model with unload Revit links option](https://forums.autodesk.com/t5/revit-api-forum/how-to-open-revit-model-with-unload-revit-links-option/m-p/13009038):
\*\*Question:\*\*
How can I open a Revit model with Revit links unloaded using
the [`OpenDocumentFile` method](https://www.revitapidocs.com/2024/5716f206-98ee-0490-4c6c-f0cdd6644190.htm)?
\*\*Answer:\*\*
You can achieve this using the transmission data:
- [List Linked Files and TransmissionData](http://thebuildingcoder.typepad.com/blog/2011/05/list-linked-files-and-transmissiondata.html)
- [Using the WriteTransmissionData Method](http://thebuildingcoder.typepad.com/blog/2011/10/using-the-writetransmissiondata-method.html)
- [Standalone BasicFileInfo and ExtractPartAtom](http://thebuildingcoder.typepad.com/blog/2018/04/standalone-basicfileinfo-and-extractpartatom-method.html)
Here are some old notes from another case on closing worksets and unloading links when opening model:
Q: Can worksets be closed or links unloaded programmatically without actually opening the RVT model?
A developer asks: System memory is hitting the limit.
My add-in does not need the Revit links to be loaded to be able to function.
Can I somehow open the RVT file with
(1) all worksets closed or
(2) all RVT links unloaded
to save system memory?
Is there a way to achieve this in the APS design automation activity command line?
A: I have two samples demonstrating how to open a RVT file with worksets closed / RVT links unloaded, but I haven’t migrated them to DA4R.
Q: Just to confirm – if you change this in the transmission data, you do not need to save the RVT file in Revit in order to not load links next time it is opened?
A: The command to open the model with worksets closed just sets a property in the `OpenOptions`, `SetOpenWorksetsConfiguration`.
It then calls `OpenAndActivateDocument` to open the model with worksets closed.
So, this is only valid for the lifetime of the `OpenOptions` and the open call needs to be executed immediately.
The command to not load links is different: for that, an option is set and stored in the model transmission data using `TransmissionData.WriteTransmissionData`.
In that case, the option will be remembered and applied next time an open is performed.
In the sample command, an open follows immediately.
However, it could also be executed later, and the setting would be retained.
So, the two methods can both be used to disable loading on the fly, provided the caller does the opening.
I hardly think the workset config option can be controlled in this manner by the default design automation environment.
However, in DA, the file to open could be passed in as payload, and the DA add-in could implement the call to open the file itself, couldn't it?
... I spent some time to verify if the changes remain in the file after closing Revit.
As said, `WriteTransmissionData` will store the link unload state in the RVT file.
But, if a customer wants to keep workset state, they must call `Document.SaveAs` to save the changes made.
\*\*Response:\*\*
Here is my code showing how you can unload Revit links as well as CAD links:

```
public static Result UnloadRevitLinks(string filePath)
{
  FilePath location = new FilePath(filePath);
  TransmissionData transData = TransmissionData
    .ReadTransmissionData(location);
  try
  {
    if (transData != null)
    {
      ICollection externalReferences
        = transData.GetAllExternalFileReferenceIds();

      MessageBox.Show("externalReferences" + externalReferences);
      foreach (ElementId refId in externalReferences)
      {
        MessageBox.Show("refId" + refId);
        ExternalFileReference extRef = transData
          .GetLastSavedReferenceData(refId);

        MessageBox.Show("extRef" + extRef);
        if (extRef.ExternalFileReferenceType
            == ExternalFileReferenceType.RevitLink
          || extRef.ExternalFileReferenceType
            == ExternalFileReferenceType.CADLink)
        {
          MessageBox.Show("Deleteting revit links");
          transData.SetDesiredReferenceData(refId,
            extRef.GetPath(), extRef.PathType, false);
          MessageBox.Show("All revit links are deleted");
        }
      }
      transData.IsTransmitted = true;
      TransmissionData.WriteTransmissionData(
        location, transData);
      return Result.Succeeded;
    }
    else
    {
      TaskDialog.Show("Unload Revit Links",
        "The document does not have any transmission data.");
      return Result.Failed;
    }
  }
  catch (Exception ex)
  {
    MessageBox.Show("ex" + ex);
    throw;
  }
}
```

Many thanks to [Archana Sapkal](https://forums.autodesk.com/t5/user/viewprofilepage/user-id/11948904) for raising the issue and sharing their solution.