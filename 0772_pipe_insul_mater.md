---
ai_optimized: true
author: Jeremy Tammik
complexity_score: 5.0
content_type: code_example
optimization_date: '2025-12-11T11:44:14.561627'
original_url: https://thebuildingcoder.typepad.com/blog/0772_pipe_insul_mater.html
post_number: '0772'
reading_time_minutes: 6
series: mep
slug: pipe_insul_mater
source_file: 0772_pipe_insul_mater.htm
tags:
- csharp
- elements
- filtering
- parameters
- python
- references
- revit-api
- selection
- walls
- mep
title: Pipe Insulation and Insulation Material
word_count: 1262
---

### Pipe Insulation and Insulation Material

I recently looked at the
[Revit MEP 2013 API](http://thebuildingcoder.typepad.com/blog/2012/05/the-revit-2013-mep-api-and-external-services.html) and
[migrated the AdnRme sample to it](http://thebuildingcoder.typepad.com/blog/2012/05/the-adn-mep-sample-adnrme-for-revit-mep-2013.html).

To follow up on that, here is an unexpectedly hard Revit MEP 2013 question by Victor Chekalin, who recently created the nice
[DataStorage sample](http://thebuildingcoder.typepad.com/blog/2012/05/devblog-devcamp-element-and-project-wide-data.html).
The new question leads to some tricky and interesting aspects on how to access data or database elements through the Revit API that are not immediately accessible via the expected route or "normal" channels.
Where there's a will, there's a way, though :-)

**Question:** I always get Element materials using an Element.Materials property.
But now I've found the issue that I cannot retrieve PipeInsulation and pipe material using Materials property, although these elements have a material.

Furthermore, even if I can retrieve the pipe material via Pipe.PipeType.Material, I still cannot find any way to get PipeInsulation.

So I have two questions

1. Why there is no possibility to get PipeInsulation and pipe materials via Element.Materials Property?- Is there a way to get the PipeInsulation material?

I understand that pipe material is defined by the pipe type, but, for example, wall materials are defined by wall type as well and I can retrieve them from a wall instance without resorting to the wall type using the Wall.Materials property. Furthermore, the pipe material has a PipeMaterialType type not a Material type.
Why?

I looked at the API help, API Reference Guide and read the post on the
[Revit MEP 2012 API](http://thebuildingcoder.typepad.com/blog/2011/06/the-revit-mep-2012-api.html) describing
the new pipe and duct insulations, but I didn't find the answer how to retrieve PipeInsulation material via API.

PipeInsulation and PipeInsulationType don't have a Material property or GetMaterial method.
The Materials property has zero size.

By the way, why are so many differences accessing materials from pipe and pipe insulation compared to other elements?
It seems for me as if the MEP version has been created by other developers :-)

![Pipe insulation](img/pipe_insulation2.png)

**Answer:** As you are aware and point out, the pipe material is defined by the type and not the instance, so you would obviously query the type and not the instance for it.

Pipe insulation is a completely separate element from the pipe, so you have to access the insulation material from the insulation element, not from the pipe. Here is the relevant section from the Revit API 2012 help file What's New section:

#### Duct and pipe insulation and lining

The new classes

- DuctInsulation- PipeInsulation- DuctLining

and related types support read/write and create access to duct and pipe insulation and lining.
In Revit 2012, these objects are now accessible as standalone elements related to their parent duct, pipe, or fitting.

Yes, you are correct in that the access to materials for pipes is different and more difficult compared to other elements such as walls.

There is actually on-going work in this area, and it is not yet complete; prior to 2013, pipes didn't have 'normal' materials, but rather just a string property.
We are aware that materials for Walls and other elements in general are directly API accessible from the Element, instead of having to dig into the type, and would like pipe and pipe insulation material access to be just as easy.

Again, as you saw, RevitLookup displays the pipe insulation with a MaterialSet which is not displayed in bold like it is for other elements, because it is empty:

![Pipe insulation type in RevitLookup](img/snoop_pipe_insulation_type.png)

Still, we found a solution to your issue by using alternative access routes.

You can retrieve the parameter named 'Material' off the PipeInsulationType object from the PipeInsulation object.
The parameter is an ElementId.

This little difficulty is compounded by the fact that we did not immediately find any easy way to get the PipeInsulationType from a pipe.

One way that we found to achieve this is to use a filter to retrieve all PipeInsulation elements in the document and compare the HostElementId of each one to that of the original pipe.
When the matching one is found, you have the PipeInsulation element corresponding to that pipe, you can get the PipeInsulationType from that, and then get the material from the parameter.

We implemented the following helper classes and methods to demonstrate this:

- PipeFilter: Restrict user selection to pipe element only.- GetSelectedElement: Return first pre-selected element or prompt user to select a pipe.- GetPipeInslationFromPipe: Return pipe insulation from given pipe.- GetMaterialFromPipeInsulation: Return material from given pipe insulation.- GetInsulationFromSelection: Prompt user to select a pipe, retrieve its insulation and insulation material, and report the results.

Here is the entire sample code exercising these:
```python
const string \_caption = "Pipe Insulation Info";

/// <summary>
/// Restrict user selection to pipe element only
/// </summary>
class PipeFilter : ISelectionFilter
{
  public bool AllowElement( Element e )
  {
    return e is Pipe;
  }

  public bool AllowReference( Reference r, XYZ p )
  {
    return true;
  }
}

/// <summary>
/// Return first pre-selected element or
/// prompt user to select a pipe.
/// </summary>
Element GetSelectedElement( UIDocument uidoc )
{
  Element e = null;

  SelElementSet elements = uidoc.Selection.Elements;

  if( null == elements || elements.IsEmpty )
  {
    try
    {
      Reference r = uidoc.Selection.PickObject(
        ObjectType.Element,
        new PipeFilter(),
        "Please pick a pipe" );

      e = uidoc.Document.GetElement( r.ElementId );
    }
    catch( RvtOperationCanceledException )
    {
    }
  }
  else if( null != elements && !elements.IsEmpty )
  {
    IEnumerator it = elements.GetEnumerator();

    if( it.MoveNext() )
    {
      e = it.Current as Element;
    }
  }
  return e;
}

/// <summary>
/// Return pipe insulation from given pipe.
/// </summary>
PipeInsulation GetPipeInslationFromPipe(
  Pipe pipe )
{
  if( pipe == null )
  {
    throw new ArgumentNullException( "pipe" );
  }

  Document doc = pipe.Document;

  FilteredElementCollector fec
    = new FilteredElementCollector( doc )
      .OfClass( typeof( PipeInsulation ) );

  PipeInsulation pipeInsulation = null;

  foreach( PipeInsulation pi in fec )
  {
    // Find the pipe that has this inulation

    if( pi.HostElementId == pipe.Id )
      pipeInsulation = pi;
  }
  return pipeInsulation;
}

/// <summary>
/// Return material from given pipe insulation.
/// </summary>
Material GetMaterialFromPipeInsulation(
  PipeInsulation pipeInsulation )
{
  if( pipeInsulation == null )
  {
    throw new ArgumentNullException( "pipeInsulation" );
  }

  Document doc = pipeInsulation.Document;

  PipeInsulationType pipeInsulationType
    = doc.GetElement( pipeInsulation.GetTypeId() )
      as PipeInsulationType;

  Parameter p = pipeInsulationType.get\_Parameter(
    "Material" );

  return null == p
    ? null
    : doc.GetElement( p.AsElementId() ) as Material;
}

/// <summary>
/// Prompt user to select a pipe, retrieve its
/// insulation and insulation material,
/// and report the results.
/// </summary>
void GetInsulationFromSelection(
  UIDocument uidoc )
{
  Element e = GetSelectedElement( uidoc );

  if( null != e )
  {
    if( e is Pipe )
    {
      PipeInsulation pi
        = GetPipeInslationFromPipe( e as Pipe );

      if( null == pi )
      {
        TaskDialog.Show( \_caption,
          "Insulation not found" );
      }
      else
      {
        Material material
          = GetMaterialFromPipeInsulation( pi );

        if( null == material )
        {
          TaskDialog.Show( \_caption,
            "Material not found" );
        }
        else
        {
          TaskDialog.Show( \_caption,
            string.Format( "Material '{0}' <{1}>",
            material.Name, material.Id ) );
        }
      }
    }
    else
    {
      TaskDialog.Show( \_caption, "Not a pipe" );
    }
  }
}
```

Here is
[PipeInsulationMaterial.zip](zip/PipeInsulationMaterial.zip) containing
the full Visual Studio solution, source code, and add-in manifest for this external command.

Many thanks to Steven Mycynek for researching and creating this solution!

Later, Martin Schmid pointed out that the GetPipeInslationFromPipe method using a filtered element collector and comparing all pipe insulation element HostElementId properties with the target pipe can be simplified by calling the InsulationLiningBase GetInsulationIds method instead.
It takes the pipe element id and returns all pipe insulation element ids associated with it directly like this:
```csharp
  ICollection<ElementId> pipeInsulationIds
    = InsulationLiningBase.GetInsulationIds(
      doc, pipe.Id );
```

Not so easy to find, though :-)

I updated the code in the zip file above to take this approach into account as well, simply adding an assertion to verify that the result is the same.