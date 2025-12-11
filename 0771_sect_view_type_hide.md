---
ai_optimized: true
author: Jeremy Tammik
complexity_score: 2.8
content_type: qa
optimization_date: '2025-12-11T11:44:14.559446'
original_url: https://thebuildingcoder.typepad.com/blog/0771_sect_view_type_hide.html
post_number: '0771'
reading_time_minutes: 1
series: elements
slug: sect_view_type_hide
source_file: 0771_sect_view_type_hide.htm
tags:
- csharp
- elements
- family
- parameters
- revit-api
- views
title: Change Section View Type and Hide Cut Line
word_count: 222
---

### Change Section View Type and Hide Cut Line

Leo raised a couple of questions on
[section view creation](http://thebuildingcoder.typepad.com/blog/2011/07/section-view-creation.html)
in his
[comment](http://thebuildingcoder.typepad.com/blog/2011/07/section-view-creation.html?cid=6a00e553e168978833016766a448ba970b#comment-6a00e553e168978833016766a448ba970b) which
Rudolf Honke very kindly answers thus:

**1. Hide view:**
To hide the cutting line in a plan view, you can use this parameter:
```csharp
  viewSection.get\_Parameter( BuiltInParameter
    .SECTION\_COARSER\_SCALE\_PULLDOWN\_METRIC )
      .Set( 1 );
```

**2. Generate section view:**
In Revit 2013, you can use the static method
```csharp
  public static ViewSection CreateSection(
    Document document,
    ElementId viewFamilyTypeId,
    BoundingBoxXYZ sectionBox
  )
```

In Revit 2012, you can use document.Create.NewViewSection( BoundingBoxXYZ );

**3. Change the type of a detail view:**
Each view has a view type, e.g. a section view symbol.
These symbols have no category to help identify them, they are just generic ElementType instances.
The types can be retrieved using their name, e.g. "Section" etc.
Once you have a view symbol, you can change the view type by simply setting its type id:
```csharp
viewSection.ChangeTypeId( symbolSection.Id );
```

Since section symbols are identified by name, they are language dependent and there is no guarantee to find them at all, depending on the template being used.

Many thanks to Rudolf for these important hints!