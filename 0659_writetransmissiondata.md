---
ai_optimized: true
author: Jeremy Tammik
complexity_score: 3.0
content_type: code_example
optimization_date: '2025-12-11T11:44:14.346435'
original_url: https://thebuildingcoder.typepad.com/blog/0659_writetransmissiondata.html
post_number: 0659
reading_time_minutes: 2
series: general
slug: writetransmissiondata
source_file: 0659_writetransmissiondata.htm
tags:
- csharp
- elements
- parameters
- python
- references
- revit-api
- transactions
title: Using the WriteTransmissionData Method
word_count: 435
---

### Using the WriteTransmissionData Method

I presented a sample to
[using the TransmissionData class to list linked files](http://thebuildingcoder.typepad.com/blog/2011/05/list-linked-files-and-transmissiondata.html),
and Mike Caruso submitted a
[comment](http://thebuildingcoder.typepad.com/blog/2011/05/list-linked-files-and-transmissiondata.html?cid=6a00e553e168978833014e8890c564970d#comment-6a00e553e16897883301) asking
how to use it to set a new path on a RVT file.
Furthermore, Guming recently asked
[how to make use of the ExternalFileReference class](http://thebuildingcoder.typepad.com/blog/2011/09/analytical-model-isenabled-method-and-parameter.html?cid=6a00e553e168978833015435b29a1e970c#comment-6a00e553e168978833015435b29a1e970c).
Here is a sample answering both of these questions:

**Question:** Can you demonstrate how I would load, unload and make changes to the path of linked Revit files?

**Answer:** Loading, unloading and editing path to referenced RVT files is possible using methods on the TransmissionData class.
Please note that the host RVT file must be closed in order to be editable by these methods.

The Revit API help file RevitAPI.chm description of the TransmissionData class includes a sample code snippet defining the aptly named method UnloadRevitLinks to unload all Revit links.

Here is a sample external command based on that to change the Revit linked model path:
```python
/// <summary>
///  This command will change the path of all linked
///  Revit files the next time the document at the
///  given location is opened.
///  Please refer to the TransmissionData reference
///  for more details.
/// </summary>
[Transaction( TransactionMode.ReadOnly )]
public class CmdChangeLinkedFilePath : IExternalCommand
{
  public Result Execute(
    ExternalCommandData commandData,
    ref string message,
    ElementSet elements )
  {
    UIApplication uiapp = commandData.Application;
    UIDocument uidoc = uiapp.ActiveUIDocument;
    Application app = uiapp.Application;
    Document doc = uidoc.Document;

    FilePath location = new FilePath( "C:/file.rvt" );

    TransmissionData transData
      = TransmissionData.ReadTransmissionData(
        location );

    if( null != transData )
    {
      // Collect all (immediate) external
      // references in the model

      ICollection<ElementId> externalReferences
        = transData.GetAllExternalFileReferenceIds();

      // Find every reference that is a link

      foreach( ElementId refId in externalReferences )
      {
        ExternalFileReference extRef
          = transData.GetLastSavedReferenceData(
            refId );

        if( extRef.ExternalFileReferenceType
          == ExternalFileReferenceType.RevitLink )
        {
          // Change the path of the linked file,
          // leaving everything else unchanged:

          transData.SetDesiredReferenceData( refId,
            new FilePath( "C:/MyNewPath/cut.rvt" ),
            extRef.PathType, true );
        }
      }

      // Make sure the IsTransmitted property is set

      transData.IsTransmitted = true;

      // Modified transmission data must be saved
      // back to the model

      TransmissionData.WriteTransmissionData(
        location, transData );
    }
    else
    {
      TaskDialog.Show( "Unload Links",
        "The document does not have"
        + " any transmission data" );
    }
    return Result.Succeeded;
  }
}
```

By setting the four parameters of SetDesiredReferenceData method, you can also cause a loaded linked file to be unloaded, and vice versa.

Here is
[version 2012.0.93.0](zip/bc_12_93.zip) of
The Building Coder samples including the new CmdChangeLinkedFilePath command.

**Response:** Thank you!
That worked like a charm.