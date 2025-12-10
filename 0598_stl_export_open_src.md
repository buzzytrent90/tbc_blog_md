---
post_number: "0598"
title: "Revit STL Exporter Released as Open Source"
slug: "stl_export_open_src"
author: "Jeremy Tammik"
tags: ['family', 'parameters', 'revit-api']
source_file: "0598_stl_export_open_src.htm"
original_url: "https://thebuildingcoder.typepad.com/blog/0598_stl_export_open_src.html"
---

### Revit STL Exporter Released as Open Source

Here is one of the most surprising and promising recent news items for Revit API developers, published yesterday by Emile Kfouri on
[BIM Apps](http://bimapps.typepad.com):
The
[Revit STL exporter is now open source](http://bimapps.typepad.com/bim-apps/2011/06/stl-exporter-for-revit-2012-is-open-source.html).

Look at Emile's post for the complete story, including:

- Introduction, motivation, and explanation.- Definitions of STL, stereolithography, and open source.- STL exporter history.- User feedback.- Future.- Open source license and participating parties.- How to get started.

Exciting news!

#### Adding a Parameter to a Family

To add an unrelated technical note, here is a very quick little question that crops up from time to time:

**Question:** How can I a parameter to an existing family?

**Answer:** You can add a real family parameter directly using the FamilyManager AddParameter method taking the arguments string, BuiltInParameterGroup, ParameterType, Boolean.

Alternatively, you can create a shared parameter definition and add it to family using the FamilyManager AddParameter method taking the arguments ExternalDefinition, BuiltInParameterGroup, Boolean:

- AddParameter(String, BuiltInParameterGroup, ParameterType, Boolean) adds a new family parameter with a given name.- AddParameter(ExternalDefinition, BuiltInParameterGroup, Boolean) adds a new shared parameter to the family.