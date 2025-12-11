---
ai_optimized: true
author: Jeremy Tammik
complexity_score: 5.0
content_type: qa
optimization_date: '2025-12-11T11:44:13.531358'
original_url: https://thebuildingcoder.typepad.com/blog/0198_roof_eave_cut.html
post_number: 0198
reading_time_minutes: 4
series: general
slug: roof_eave_cut
source_file: 0198_roof_eave_cut.htm
tags:
- csharp
- elements
- parameters
- revit-api
- selection
- walls
title: Roof Eave Cut in Revit and ACA
word_count: 816
---

### Roof Eave Cut in Revit and ACA

Here is a case that is interesting in its own right, or rather two separate cases and a comparison between the two.
The issue is the programmatic setting of the roof eave cut property in AutoCAD Architecture and Revit, respectively.
One interesting aspect is the similarity and differences between the two solutions.

First of all, here is the original question, addressed at ACA:

**Question:**
I am having difficulty accessing the edges of a RoofSlab entity.
I am programming in C#.
I need to change the EdgeCut to a Plumb Cut and I don't know how to access the edges so that I can change the property from Square Cut to Plumb Cut.

#### Setting the SlabEdge.IsPlumbCut Property in AutoCAD Architecture

In ACA, this issue can also be formulated like this:

**Question:** How can I define the roof slab edge plumb cut property and set it to plumb or square cut in AutoCAD Architecture?
The ACA.NET API SlabEdge.IsPlumbCut property is read-only, so some other method is needed.

**Answer:** As said, the required property is represented by the SlabEdge.IsPlumbCut property, which is read-only in the API.
So you are searching for write access to this property?

This property can be controlled by the roof slab style.
One way to change it is to set up an appropriate edge style.
You can define an edge style that uses the plumb orientation by default and then define a slab style using that edge style.
When you convert to slab, use that slab style, and the plumb edge will come with it.

You can also directly access and write to this property on each edge individually, independently of the edge style, by using the SlabEdge.SetCutOption method.
The setting of this property is displayed when you select the 'Edges' entry in the roof slab property palette:
![ACA Slab Edges](img/aca_slab_edges.png)

I implemented a sample ACA.NET application PlumbCut which toggles the state of this setting on each edge of a selected roof slab every time it is called.
Here is the relevant code snippet from the command method implementation:
```csharp
  RoofSlab roofSlab = t.GetObject(
    entId, OpenMode.ForWrite ) as RoofSlab;

  SlabFace face = roofSlab.Face;

  foreach( SlabLoop loop in face.Loops )
  {
    foreach( SlabEdge edge in loop.Edges )
    {
      bool isPlumb = edge.IsPlumbCut;

      // SlabEdge.IsPlumbCut cannot be assigned
      // to -- it is read only:
      //edge.IsPlumbCut = !edge.IsPlumbCut;

      edge.SetCutOption( isPlumb );
    }
  }
```

Here is the complete
[PlumbCut](zip/PlumbCut.zip)
ACA.NET command source code and the complete Visual Studio solution to build it.

So much for AutoCAD Architecture.
Here is an answer to the analogue question in the context of Revit:

#### Setting the Roof Eave Cut Property in Revit

**Question:** How can I define the roof eave cut property and set it to plumb or square cut in Revit?

**Answer:** I created a very simple sample model consisting of four walls supporting a footprint roof to explore your query and see the property you are looking for, the roof element Rafter Cut property, which can take the following values:

- Plumb Cut- Two Cut - Plumb- Two Cut - Square

![Roof Rafter Cut Property](img/roof_rafter_cut.png)

These values correspond to the following EaveCutterType enumeration values in the API:

- PlumbCut- TwoCutPlumb- TwoCutSquare

Exploring the roof element parameters using RvtMgdDbg, I can see that these values are stored in the integer-valued built-in parameter ROOF\_EAVE\_CUT\_PARAM.

To ensure that this parameter can be accessed and its integer value read and written using the EaveCutterType enumeration values, I implemented an external command and a sample application RoofEaveCut.
To run it, you need to pre-select a foot print roof in the model.
Then, each time you call the external command, the value of the Rafter Cut property is cycled through the three values listed above.
This demonstrates that you can both read and write the built-in ROOF\_EAVE\_CUT\_PARAM parameter value as an integer and cast it to the EaveCutterType enumeration values.
Here is the relevant code snippet:

```csharp
const string \_prompt
  = "Please select a single foot print roof element.";

const BuiltInParameter \_bip
  = BuiltInParameter.ROOF\_EAVE\_CUT\_PARAM;

public IExternalCommand.Result Execute(
  ExternalCommandData commandData,
  ref string message,
  ElementSet elements )
{
  Application app = commandData.Application;
  Document doc = app.ActiveDocument;
  ElementSet els = doc.Selection.Elements;
  if( 1 != els.Size )
  {
    message = \_prompt;
  }
  else
  {
    FootPrintRoof roof = null;
    foreach( Element e in els )
    {
      roof = e as FootPrintRoof;
    }
    if( null == roof )
    {
      message = \_prompt;
    }
    else
    {
      Parameter p = roof.get\_Parameter( \_bip );
      int i = p.AsInteger();
      switch( i )
      {
        case (int) EaveCutterType.PlumbCut:
          i =(int) EaveCutterType.TwoCutPlumb;
          break;

        case (int) EaveCutterType.TwoCutPlumb:
          i =(int) EaveCutterType.TwoCutSquare;
          break;

        case (int) EaveCutterType.TwoCutSquare:
          i =(int) EaveCutterType.PlumbCut;
          break;
      }
      p.Set( i );
    }
  }
  return 0 == message.Length
    ? IExternalCommand.Result.Succeeded
    : IExternalCommand.Result.Failed;
}
```

Here is the complete
[RoofEaveCut](zip/RoofEaveCut.zip)
Revit external command source code and the complete Visual Studio solution to build it.