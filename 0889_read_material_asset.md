---
post_number: "0889"
title: "Read Material Asset Parameter"
slug: "read_material_asset"
author: "Jeremy Tammik"
tags: ['csharp', 'elements', 'family', 'parameters', 'revit-api', 'selection']
source_file: "0889_read_material_asset.htm"
original_url: "https://thebuildingcoder.typepad.com/blog/0889_read_material_asset.html"
---

### Read Material Asset Parameter

Even though it is still January, the days are already getting a little bit lighter, the sun has broken through now and then, sets half an hour later, some birds are starting to tweet around, and one can imagine spring coming...

Meanwhile, in the Revit API, accessing materials tends to be tricky, so here is one little sample that hopefully helps clarify by demonstrating how to read a parameter value from a PropertySetElement attached to a physical material asset.

**Question:** I'd like to know if there's an API to access the parameter called "Tension parallel to grain" in a wood material structural asset.
The specific material I'm looking at is Softwood, Lumber and is present in a default new structural project.

There is a built-in parameter called PHY\_MATERIAL\_PARAM\_TENSION\_PARALLEL but that doesn't work.
The RevitLookup tool does not list the parameter, even though it is shown in the Revit GUI.

The "Tension Parallel to Grain" parameter contains a Strength value, which is a double.

To access this through the GUI:

- Start a new model in Revit- Go to Manage > Materials- Scroll to a material that belongs to the wood category and contains a Structural Asset. In a new model, you can scroll to Softwood, Lumber.- Select the Structural Asset (Physical Aspect).- Expand the Strength section and you'll see the parameter.

The parameter is attached to the StructuralAsset element.

Here is the Softwood, Lumber material listed and selected in the material browser:

![Material browser](img/material_browser.png)

Selecting it and navigating to the physical aspect shows the tension parallel to grain value:

![Material editor](img/material_editor.png)

How can I access that data programmatically, please?

**Answer:** You can access this parameter using the PropertySetElement.

Assuming softwood is assigned to a selected element, the following code achieves this, including extracting the single selected element from the current document selection set:

```csharp
public Result ReadMaterialParam( UIDocument uidoc )
{
  Document doc = uidoc.Document;

  Element e = null;

  Selection sel = uidoc.Selection;

  if( 1 == sel.Elements.Size )
  {
    foreach( Element e2 in sel.Elements )
    {
      e = e2;
    }
  }

  if( null == e )
  {
    TaskDialog.Show( "Error",
      "Please select one single element." );

    return Result.Failed;
  }

  Parameter paramMaterial = e.get\_Parameter(
    BuiltInParameter.STRUCTURAL\_MATERIAL\_PARAM );

  Material material = doc.GetElement(
    paramMaterial.AsElementId() ) as Material;

  PropertySetElement property = doc.GetElement(
    material.StructuralAssetId ) as PropertySetElement;

  Parameter paramTensionParallel
    = property.get\_Parameter( BuiltInParameter
      .PHY\_MATERIAL\_PARAM\_TENSION\_PARALLEL );

  TaskDialog.Show(
    "PHY\_MATERIAL\_PARAM\_TENSION\_PARALLEL",
    paramTensionParallel.AsValueString() );

  return Result.Succeeded;
}
```

This value cannot be read directly from the StructuralAsset properties.

To test run it, you can

- Create a new structural model.
- Insert a structural element, e.g. a beam, by loading the structural framing family and selecting the symbol "Dimension Lumber" 38x64.
- Assign "Softwood Lumber" to it via Properties > Structural Material > ...

Selecting the beam and launching the command calling the ReadMaterialParam method displays the following message on my system:

![Material asset parameter value](img/read_material_asset.png)

**Addendum:**

In his commment below,
[Alexander Buschmann](http://www.idat.de) makes
the observation that properties can be used instead of parameters to read some of the asset data, which I promoted that to a follow-up blog post of its own on
parameters versus properties.