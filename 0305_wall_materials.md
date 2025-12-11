---
ai_optimized: true
author: Jeremy Tammik
complexity_score: 1.2
content_type: qa
optimization_date: '2025-12-11T11:44:13.715300'
original_url: https://thebuildingcoder.typepad.com/blog/0305_wall_materials.html
post_number: '0305'
reading_time_minutes: 2
series: materials
slug: wall_materials
source_file: 0305_wall_materials.htm
tags:
- elements
- family
- revit-api
- walls
- materials
title: Wall Solid versus Face Materials
word_count: 403
---

### Wall Solid versus Face Materials

Still away on holiday in Andalusia, here is another little item that I am posting ahead of time.

We looked at various aspects of materials in the past, such as the materials assigned to an
[element and its solid faces](http://thebuildingcoder.typepad.com/blog/2008/10/element-materials.html) and to a
[family instance](http://thebuildingcoder.typepad.com/blog/2008/10/family-instance-materials.html),
[material quantity extraction](http://thebuildingcoder.typepad.com/blog/2010/02/material-quantity-extraction.html), creating a
[new material](http://thebuildingcoder.typepad.com/blog/2009/05/new-material.html),
and the fact that in the general case, there is no
[solid material](http://thebuildingcoder.typepad.com/blog/2009/11/solid-material.html).
There are exceptions to the last statement, though, as we see from the answer to the following question:

**Question:** I created a single wall and assigned different materials to several of its faces.
I have no problem determining the different materials based on the face, but I can't get the material that is initially assigned to the wall itself.

If I do nothing to change the material of the wall, it appears as the first material item at the object.
If I do change it, though, it appears at the end of the list.
So I cannot rely on the position in the list.

Is there some way to get the material from the entire wall, based on the solid?
We want to assign the material to the solid, and create additional surfaces for materials which differ from the material of the wall itself.
To do this, we need to determine the material of the wall itself.

**Answer:** You can obtain the material of the entire wall from its
[compound structure](http://thebuildingcoder.typepad.com/blog/2008/11/wall-compound-layers.html),
which includes material information for each of its layers.
We also looked at the issue of calculating the
[compound wall layer volumes](http://thebuildingcoder.typepad.com/blog/2009/02/compound-wall-layer-volumes.html).

By the way, it might be interesting to compare the results of the
[GetMaterialArea and GetMaterialVolume](http://thebuildingcoder.typepad.com/blog/2010/02/material-quantity-extraction.html) methods
with the face-based area calculation mentioned above and the volume calculation based on the compound layer structure volumes.

Many thanks to Steffen Rabe of
[RIB Software AG](http://www.rib-software.com)
for this solution!