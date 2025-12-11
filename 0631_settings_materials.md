---
ai_optimized: true
author: Jeremy Tammik
complexity_score: 1.7
content_type: documentation
optimization_date: '2025-12-11T11:44:14.298000'
original_url: https://thebuildingcoder.typepad.com/blog/0631_settings_materials.html
post_number: '0631'
reading_time_minutes: 1
series: materials
slug: settings_materials
source_file: 0631_settings_materials.htm
tags:
- csharp
- elements
- filtering
- revit-api
- materials
title: Retrieving Materials
word_count: 140
---

### Retrieving Materials

Some people have reported issues using the Document.Settings.Materials collection.
For instance, its Contains method may throw an exception when used with Revit Structure 2012, while it works fine with Revit Architecture 2012.

The problem actually probably has nothing to do with the flavour of Revit being used, but is caused by the template file that the project is based on.
If you retrieve materials in a project based on the architectural template in Revit Structure, it also works fine.

In any case, here is a workaround you can use instead.
Simply retrieve the materials using a filtered element collector instead of the Materials collection, for example like this:
```csharp
  FilteredElementCollector collector
    = new FilteredElementCollector( document )
      .OfClass( typeof( Material ) );

  IEnumerable<Material> materialsEnum
    = collector.ToElements().Cast<Material>();

  var materialReturn1
    = from materialElement in materialsEnum
      where materialElement.Name == "Default"
      select materialElement;
```