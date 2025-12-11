---
ai_optimized: true
author: Jeremy Tammik
complexity_score: 1.4
content_type: qa
optimization_date: '2025-12-11T11:44:13.626345'
original_url: https://thebuildingcoder.typepad.com/blog/0253_solid_material.html
post_number: '0253'
reading_time_minutes: 1
series: materials
slug: solid_material
source_file: 0253_solid_material.htm
tags:
- doors
- elements
- geometry
- revit-api
- materials
title: Solid Material
word_count: 191
---

### Solid Material

Here is a simple question with a simple answer on materials and solids:

**Question:** Is there a way to retrieve the material of the solid geometry object of a Revit Element, like the door leaf of a door element?

**Answer:** No, I am not aware of any way that a material can be assigned to a solid.
The only representation of the solid in the graphical user interface is through its faces, and they can have materials assigned to them.
As far as I know, the solid itself has no such property.
The only way that I am aware of to
[determine the material](http://thebuildingcoder.typepad.com/blog/2008/10/element-materials.html)
of a solid is through the path Solid > Faces > FaceArray > Item > Face > MaterialElement.

Another short little note on a completely unrelated topic:

### Clean Uninstall of Revit 2010

In case you ever have problems with the installation of Revit, the Autodesk Revit Architecture Services & Support area includes a page describing a method to achieve a
[clean uninstall of Revit 2010 products](http://usa.autodesk.com/adsk/servlet/ps/dl/item?siteID=123112&id=13712572&linkID=9243099).