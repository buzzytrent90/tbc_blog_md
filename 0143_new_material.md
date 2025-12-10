---
post_number: "0143"
title: "New Material"
slug: "new_material"
author: "Jeremy Tammik"
tags: ['revit-api']
source_file: "0143_new_material.htm"
original_url: "https://thebuildingcoder.typepad.com/blog/0143_new_material.html"
---

### New Material

A quick post from my vacation in sunny and windy
[Avignon](http://en.wikipedia.org/wiki/Avignon).

**Question:**
How can I create a new material in Revit?
I have searched in vain for something like Document.Create.NewMaterial.

**Answer:**
Searching in the Revit API help file for NewMaterial does indeed yield no results at all.
However, searching for the two separate words "new material" returns a number of hits.
In my case, the third is the Duplicate method, and the fourth the AddOther method.
As far as I can see, those two represent the two different possible approaches to create new materials:

One is the use of the Material.Duplicate method, which is demonstrated by the Revit SDK sample Materials.

The other is to use one of the material adding methods on the Materials class, which provides access to the set of materials in a Revit project.
It provides the following methods for adding new materials:

- AddConcrete to add a concrete material.- AddGeneric to add a generic material.- AddOther to add an other material.- AddSteel to add a steel material.- AddWood to add a wood material.

Once a material has been duplicated or added, you can modify its properties to suit your needs.