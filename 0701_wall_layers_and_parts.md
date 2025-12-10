---
post_number: "0701"
title: "Identifying Wall Compound Layers and Parts"
slug: "wall_layers_and_parts"
author: "Jeremy Tammik"
tags: ['geometry', 'levels', 'parameters', 'references', 'revit-api', 'walls']
source_file: "0701_wall_layers_and_parts.htm"
original_url: "https://thebuildingcoder.typepad.com/blog/0701_wall_layers_and_parts.html"
---

### Identifying Wall Compound Layers and Parts

We recently looked at how to determine the exact geometry and wall
[join points of compound layer geometry](http://thebuildingcoder.typepad.com/blog/2011/10/retrieving-detailed-wall-layer-geometry.html) subcomponents by temporarily creating separate parts from the wall.
Here is another interesting question the correlation between the wall compound layers and its ensuing parts:

**Question:** I have split my wall into parts, and now I would like to determine which of the resulting parts correspond to the component structure layers with certain functions such as Structure or Membrane.

Once the parts have been created, the CompoundStructureLayer information is apparently lost. As a workaround, I was thinking of creating a list of the compound structure layers before converting the wall to parts and storing their materials, width etc.
After the parts are created, I hoped to access each part, search for their material and width, and map these to the list I saved in order to know which part originally had which function.

I am hoping that there is a better way to achieve this, since this approach has multiple limitations.

As a simple example, let's assume we have a wall with two layers, layer 1 and layer 2.
After creating parts from it, we have parts named 1 and 2.
From part 1, I can access the wall that it was created from, and the wall contains all the original layers.
So far, so good.
Now the task here is to find which part corresponds to the layer whose function parameter is say Membrane before it was converted into parts.
How do I know which part was created from which layer?
I was thinking of using material, width etc. to do the check, but if we multiple layers have the same material and width, and possibly even volume, how can I make this mapping foolproof?
Furthermore, some parts may be further subdivided into say horizontal parts, which will add another level of complexity to the problem.

**Answer:** It should be possible to find the location of any layer.

1. Remember that the wall location line is always the centre of the wall, regardless of the layer structure,- Use CompoundStructure.GetOffsetForLocationLine to find other locations such as CoreBoundary, FinishBoundary, etc., and/or- Use CompoundStructure.GetLayerWidth to walk the layers in order, and/or- Use the FindReferencesByDirectionWithContext ray tracing method to find the wall parts in order.

For vertically compound structures, there is more work to be done, as you pointed out.

It is important to remember that zero width membrane layers do not generate parts.