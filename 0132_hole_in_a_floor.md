---
ai_optimized: true
author: Jeremy Tammik
complexity_score: 2.4
content_type: qa
optimization_date: '2025-12-11T11:44:13.412090'
original_url: https://thebuildingcoder.typepad.com/blog/0132_hole_in_a_floor.html
post_number: '0132'
reading_time_minutes: 1
series: general
slug: hole_in_a_floor
source_file: 0132_hole_in_a_floor.htm
tags:
- elements
- family
- levels
- parameters
- revit-api
- walls
title: Hole in a Floor
word_count: 295
---

### Hole in a Floor

**Question:**
I am creating a floor using the Revit API method Document.NewFloor. It takes the arguments CurveArray, FloorType, Level, and Boolean. However, I would also like to create floors with openings in them, since this is quite normal for floors in real life   ;-)
How can I do this via the API? The CurveArray parameter is apparently used to specify just one single simple loop.

**Answer:**
The way to solve a problem like this, as always, is to search the Revit API documentation and samples.
Just like in the user interface, an opening in a floor is added after the floor has been constructed.
Looking through the RevitAPI.chm help file, I found that the Revit API provides an overloaded method named NewOpening on the creation document class for this purpose.
In 2010, a new method with the same name has also been added to the FamilyItemFactory class for defining openings in host elements such as walls or ceilings in family documents.
In your case, you are interested in the former.
Its various overloads do the following:

- Create a new opening in a beam, brace and column.
- Create a new shaft opening between a set of levels.
- Create a new opening in a roof, floor and ceiling.

In other words, the third overload is exactly what you are looking for.

I then searched the Revit SDK samples globally for the NewOpening method and found occurrences in the NewOpenings and ShaftHolePuncher samples:

- The NewOpenings sample uses the Document.NewOpening method to create an opening on a selected wall or floor.
- The ShaftHolePuncher sample demonstrates how to create a single or shaft opening on a wall, floor or beam.

Hopefully, these samples will provide all the information you need to solve your task.