---
post_number: "1249"
title: "Traditional, 3D Printed, Vertical Compound Structures"
slug: "vertical_compound"
author: "Jeremy Tammik"
tags: ['elements', 'references', 'revit-api', 'views', 'walls']
source_file: "1249_vertical_compound.htm"
original_url: "https://thebuildingcoder.typepad.com/blog/1249_vertical_compound.html"
---

### Traditional, 3D Printed, Vertical Compound Structures

Let's look at various kinds of structures and compound structures today:

- [A traditional American Indian sweat lodge](#1)
- [The biggest co-created 3D-printed object on earth](#2)
- [Creating and assigning vertical versus simple compound structures](#3)

#### A Traditional American Indian Sweat Lodge

First, here is a quick look at the nice
[sweat lodge](https://en.wikipedia.org/wiki/Sweat_lodge) we built last weekend for an
[Inipi](https://en.wikipedia.org/wiki/Inipi) purification ceremony:

![Sweat Lodge November 2014](file:////j/photo/jeremy/2014/2014-11-24_sweat_lodge/20141124_104435.jpg)

I traditionally try to do that before embarking on my yearly worldwide travels to Autodesk University and the DevDay conferences.

Here are
[some more pictures](https://www.facebook.com/media/set/?set=a.10204348146756673.1073741832.1019863650) of it.

By the way, this traditional hand-made hut has a certain interesting similarity with something slightly more modern:

#### The Biggest Co-created 3D-Printed Object on Earth

[Michiel van der Kley](http://www.michielvanderkley.nl) created
[Project EGG](http://projectegg.org) and
with it one of the biggest co-created 3D printed objects on Earth consisting of 4760 unique 'stones', all individually 3D-printed and shipped them to the Netherlands for assembly:

![Project EGG](img/projectegg_4s.jpg)

For more information, please refer to the
[GrabCAD interview with Michiel](http://blog.grabcad.com/blog/2014/11/21/project-spotlight-project-egg).

#### Creating and Assigning Vertical Versus Simple Compound Structures

Back to the Revit API and a more mundane technical issue raised by Scott Wilson in the Revit API discussion forum on how to
[assign a new CompoundStructure to a WallType vs modify and existing one](http://forums.autodesk.com/t5/revit-api/walltype-assigning-new-compoundstructure-vs-modify-existing/m-p/5202427),
and possibly related to GeomGym's issue on
[setting a new floor type](http://forums.autodesk.com/t5/revit-api/set-a-new-floortype-from-the-api/td-p/5098648) as well:

**Question:** I'm using Revit 2015 API to duplicate a wall type so that I can create custom types with new compound structures. I found some odd behaviour and wanted to know if there is something I'm missing or whether this is a bug.

My first approach was to duplicate a wall type of WallKind.Basic, create a list of layers with width, function and material id all set then create a new compound structure using CompoundStructure.CreateSimpleCompoundStructure passing in my list of layers.

After assigning the new compound structure to the duplicated wall type, the new type was fine except that none of the materials were displayed correctly (all dark grey like the default material). Btw, the materials used are simple coloured materials without any appearance assets assigned. When I checked the properties and layer structure of the new wall type everything looked as it should, the correct layers were assigned and the layer structure preview even showed the layers in their correct colours.

I tried a few things like working with duplicates of different wall types and re-assigning the materials to each layer again using SetMaterialId but nothing changed the behaviour.

Finally I tried using the existing compound structure of the duplicated wall type and used SetLayers on it with the exact same list of layers I was using previously with CreateSimpleCompoundStructure and everything worked fine, all layers displayed in correct colours. This confirms to me that the problem does not lie with the materials I'm using or the way I have constructed the layers, but is caused by assigning a compound structure created using CreateSimpleCompoundStructure.

So, in summary my problem is solved, but I'm interested if anyone knows why using CreateSimpleCompoundStructure doesn't work properly, is there a property I need to set somewhere or is it possibly a bug in the API?

If I take an existing WallType and assign a new CompoundStructure that was created using CompoundStructure.CreateSimpleCompoundStructure the materials are not displayed correctly (they are all grey like the 'Default' material with no layer separation visible even in a section view).

If I instead get the WallType's existing CompoundStructure and just replace its layers with exactly the same list of layers as used before, then assign the modified CompoundStructure back to the WallType everything is fine. Because of this I'm certain that the problem is not with the layers I've created or the materials being used. It is something caused by CreateSimpleCompoundStructure.

I suspect that it is a bug in the API, but it could also be that there is something I am unaware of that needs to be done to a new CompoundStructure before the material colours will display correctly.

**Answer:** I logged the issue REVIT-47356 [CreateSimpleCompoundStructure materials display wrong] with our development team for this on your behalf as it requires further exploration and possibly a modification to our software. Please make a note of this number for future reference.

After some careful analysis by one of the few people who know the internals of everything, we reached the following conclusion on this:

The whole issue is moot, almost.

The only thing required is a documentation enhancement:

You cannot apply the result of CreateSimpleCompoundStructure to a wall type; you must create a vertically compound structure instead.

The CompoundStructure class provides the following static creation members:

- CreateSimpleCompoundStructure – Creates a non-vertically compound structure comprised of parallel layers.
- CreateSingleLayerCompoundStructure(MaterialFunctionAssignment, Double, ElementId) – Creates a CompoundStructure containing a single layer.
- CreateSingleLayerCompoundStructure(Double, MaterialFunctionAssignment, Double, ElementId) – Creates a vertically compound CompoundStructure with one layer.

Two things remain to be cleaned up:

1. This does not appear to be documented. Remarks should be added to the documentation on the CreateSimpleCompoundStructure and CreateVerticallyCompoundStructure methods.
2. The internal HostObjAttr::setCompoundStructure only throws an exception if its input is vertically compound and the derived type does not support it (everything except walls). It does not throw when the opposite situation is encountered, i.e., a simple compound structure is passed to a wall type.

If you replace CreateSimpleCompoundStructure with the overload to create a vertically compound structure, the problem should be resolved.

The issue will remain open until the documentation is updated and extra validation added to SetCompoundStructure.