---
ai_optimized: true
author: Jeremy Tammik
complexity_score: 1.2
content_type: qa
optimization_date: '2025-12-11T11:44:13.404456'
original_url: https://thebuildingcoder.typepad.com/blog/0126_access_linked_file_geometry.html
post_number: '0126'
reading_time_minutes: 1
series: geometry
slug: access_linked_file_geometry
source_file: 0126_access_linked_file_geometry.htm
tags:
- elements
- geometry
- revit-api
- walls
title: Access to Linked File Geometry
word_count: 299
---

### Access to Linked File Geometry

Here are some notes on accessing the geometry of imported or linked files from a case handled by my colleague Joe Ye:

**Question:**
How can I access the geometry of imported or linked Revit RVT, AutoCAD DWG, and SketchUp SKP files?

For instance, I have created a file Wall.rvt containing a wall and use Import/Link to insert it into LinkWall.rvt.
Now, when traversing the entities of LinkWall.rvt, I see an Autodesk.Revit.Element.Instance using a Symbol which seems to be the imported or linked Wall.rvt file.
How can I access the elements from the Symbol so that I can recursively traverse the imported/linked file?
Furthermore, how can I do the same for an imported or linked SketchUp or DWG file?

**Answer:**
Each imported or linked Revit project generates a document instance in the Revit Application object's document collection Application.Documents.
Each imported RVT file maps to one document.
You can determine the document name from the symbol, and then traverse all documents in the collection to find the appropriate one.
From the corresponding document instance, you can access its elements in the usual way, for instance using Document.get\_Element or Document.get\_Elements.
Unfortunately, we currently have no solution for resolving the issue of two identically named linked files.

With regard to imported DWG or SKP files, there is no way to access elements in the imported or linked file.

For more details on these steps, please refer to the preceding discussions on
[linked files](http://thebuildingcoder.typepad.com/blog/2008/12/linked-files.html),
[how to hide them](http://thebuildingcoder.typepad.com/blog/2009/01/hiding-linked-files.html),
and
[listing their elements](http://thebuildingcoder.typepad.com/blog/2009/02/list-linked-elements.html).