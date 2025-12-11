---
ai_optimized: true
author: Jeremy Tammik
complexity_score: 3.3
content_type: qa
optimization_date: '2025-12-11T11:44:15.040257'
original_url: https://thebuildingcoder.typepad.com/blog/0980_texture_uv_coord.html
post_number: 0980
reading_time_minutes: 7
series: geometry
slug: texture_uv_coord
source_file: 0980_texture_uv_coord.htm
tags:
- elements
- family
- filtering
- geometry
- revit-api
- views
title: Texture Bitmap and UV Coordinates
word_count: 1397
---

﻿

### Texture Bitmap and UV Coordinates

Yesterday, I mentioned the new
[custom exporter framework](http://thebuildingcoder.typepad.com/blog/2013/07/graphics-pipeline-custom-exporter.html) providing
direct access to the Revit viewing and rendering graphics pipeline.

That brings new possibilities to a related area that we discussed way back in 2010: how to obtain
[texture data UV coordinates](http://thebuildingcoder.typepad.com/blog/2010/02/texture-data-uv-coordinates-and-fbx.html).
At the time, the Revit API did not provide this information, and a work-around based on the Revit FBX export helped a little.

Happily, the custom exporter framework opens that up a bit more.
Let us look at that and sundry other recent little queries:

- [Obtaining texture UV coordinates](#2)
- [Texture bitmap access](#3)
- [Calculating and marking shaded areas in the BIM](#4)
- [Nested family predicate](#5)
- [IFC viewer](#6)

#### Obtaining Texture UV Coordinates

This question was raised again in a
[forum thread on obtaining texture UV coordinates](http://forums.autodesk.com/t5/Autodesk-Revit-API/Obtaining-texture-UV-coordinates/m-p/3947006/highlight/false#M4060):

**Question:** I coded a Revit plugin that renders a scene – and calculates the UV coordinates for each vertex.
Unfortunately there are a number of situations where UV mapping is wrong – even with planar mapping.
So my question is, how can I get correct UV's?

**Answer by Arnošt Löbel:** Indeed, the regular Revit geometry API does not provide accurate UV texture mapping data.

However, correct UVs can be retrieved from the CustomExporter via an ExportContext in Revit 2014, a.k.a. Visualization API.
The OnPolymesh callback of the context receives PolymeshTopology objects that have UVs for each mesh.

The CustomExporter should be used for most (if not all) export and render external applications.

**Response:** I implemented the proposed solution and can confirm that it works extremely well.
Thank you!
My only criticism is that it still doesn't allow obtaining texture map named assigned to materials in the OnMaterial method.
I also noticed that the ArchVision RPC (cutout geometry) props do not get sent to ExportContext.
But still a huge step forward.

**Answer by Arnošt:** Thank you for making progress and reporting on it. :-)
It seems the Custom Exporter is the right tool for you to utilize.
I am sorry about the textures and RPCs.
However we are under rather strict copyright obligations and we are not allowed to export data for RPCs in any of our export formats, including custom exports.
As for the textures, although there are no such strict restrictions, the entire material component is – technically – not part of Revit code and its internal API does not allow us to acquire texture paths, at least not easily (as far as I know).
Hopefully, the material component keeps on improving going forward allowing us expanding the functionality of custom exporters in future releases of the Revit API.

#### Texture Bitmap Access

As you can see from the item above, the texture UV coordinates are available, and the bitmap is not.

As a workaround for that, Artur Brzegowy, who provided the
[custom exporter to Collada](http://thebuildingcoder.typepad.com/blog/2013/07/graphics-pipeline-custom-exporter.html#5) published
yesterday, suggested getting the texture memory data instead:

One possibility might be to create a simple plane on the fly, assign the given material on it, and make a screen snapshot of the texture using document.ExportImage or some other method.

Rudolf Honke was also interested in access to the texture bitmap, and initially suggested:

We are obviously looking for a full texture bitmap like this:

![Sample texture bitmap](img/rh_texture_01.png)

The MaterialNode provides a property named ThumbnailFile returning the path if a file that contains a thumbnail image of the material.
That is not presumably to small a preview to be useful, not a full-scale image like above.

We could also grab a texture preview image from the material browser:

![Material browser](img/rh_texture_02.png)

Unfortunately, the preview image here does not just display the texture, but other objects as well.

Based on Artur's idea, I suggested to Rudolf:

- Create a new temporary project
- Create a large cube
- Assign it the desired material
- Create a screen snapshot of that

Rudolf reports: I tried it out and am happy to say that it works.

I have one little issue with the line width, though:

![Sample bitmap](img/rh_texture_03.png)

I am requesting TL, thin lines, because the image can be cropped better that way.
The thin lines appear just fine in the standard display.

However, the exported images are adorned with a thick border.

I can obtain the desired bitmap anyway, of course.

Because of this, though, it might be better to grab the screen image directly rather than passing it through the Document.ExportImage method.
That would provide real 'what you see is what you get'.

Here is a partial cropped screen snapshot of the result – the original sample image includes 86 different textures:

![Texture bitmap results](img/rh_texture_04.png)

#### Calculating and Marking Shaded Areas in the BIM

On a completely different topic, say we have a roof with a tall chimney in a Revit building model and enable the sun settings so that shadows are displayed.

**Question:** How can we programmatically find out which area on the roof is under shadow?

**Answer:** Project both the chimney and the roof onto a plane in the direction of the sunlight and then calculate the overlap.

**Response:** Will that be accurate?
Is there a direct API to get the shaded area?

**Answer:** Yes, of course it is accurate.
No, there is no direct API support for that.

You can use the
[extrusion analyser](http://thebuildingcoder.typepad.com/blog/2013/04/extrusion-analyser-and-plan-view-boundaries.html) to
determine the shaded area projected onto a plane.
Project the roof itself onto the same plane as well.
Then use a
[2D Boolean operation on the plane](http://thebuildingcoder.typepad.com/blog/2009/02/boolean-operations-for-2d-polygons.html) to
determine the overlap.

Projecting these results from the plane back onto the roof for different hours of the day would enable you to display the outlines of the shadows as required.

#### Nested Family Predicate

Here is another unrelated topic, closer to the kernel Revit API this time:

**Question:** How can I test whether a family is nested within another family, whether it has families nested within itself, and if so which ones?

**Answer:** If a family A is nested within a family B, and the family is 'Shared', then you will also be able to see the elements and types of family A in any project containing family B.

One typical use of this is for furniture, e.g. a Table and Chairs family may contain a chair family that can be placed independently.

If the family is not shared, which is the more typical case, then you need to open the family’s document to see the members it contains.
This can be achieved in memory via the EditFamily method.

You can open family B and read the family elements in its database; A will appear as a Family element.
Furthermore, there should be one or more family symbols of A and one or more family instances of A in B's database.

Therefore, just running the appropriate filtered element collectors will answer the question quickly, easily and efficiently.

Here are some previous discussions related to this:

- [Nested instance geometry](http://thebuildingcoder.typepad.com/blog/2009/05/nested-instance-geometry.html)
- [Nested family](http://thebuildingcoder.typepad.com/blog/2009/11/nested-family.html)
- [Nested family instance](http://thebuildingcoder.typepad.com/blog/2010/02/nested-family-instance.html)
- [Nested family utility methods](http://thebuildingcoder.typepad.com/blog/2010/03/nested-family-utility-methods.html)
- [Nested lighting fixture instances](http://thebuildingcoder.typepad.com/blog/2011/04/nested-lighting-fixture-instances.html)

#### IFC Viewer

Finally, on yet another unrelated topic, here is some IFC news and pointers that might come in handy:

Navisworks supports IFC only, not ifcXML and ifcZip. The currently supported version is IFC2X3 and lower, not yet IFC4
([IFC specification](http://www.buildingsmart-tech.org/specifications/ifc-overview/ifc-overview-summary)).

IFC doesn’t support texture. Colour support is also patchy, although the Navisworks 2014 SP1 significantly improves the situation.

As regards IFC viewers, the [Solibri model viewer](http://www.solibri.com/) appears to be quite good.