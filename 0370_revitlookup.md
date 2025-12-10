---
post_number: "0370"
title: "RevitLookup Update"
slug: "revitlookup"
author: "Jeremy Tammik"
tags: ['csharp', 'elements', 'family', 'filtering', 'revit-api']
source_file: "0370_revitlookup.htm"
original_url: "https://thebuildingcoder.typepad.com/blog/0370_revitlookup.html"
---

### RevitLookup Update

I recently mentioned
[RevitLookup](http://thebuildingcoder.typepad.com/blog/2010/04/revitlookup-and-textnote-alignment.html),
the new incarnation of
[RvtMgdDbg](http://thebuildingcoder.typepad.com/blog/2009/02/rvtmgddbg.html),
which is now included in the Revit SDK, and so did
[Matt Mason](http://cadappdev.blogspot.com/2010/04/revit-2011-api-series-revitlookup-new.html).

The version provided in the initial release of the Revit 2011 SDK had several pretty fundamental problems.
We are in a continuous process of improving it, and now have a version available with the following enhancements:

- ElementType instances- Revit MEP connectors- Revit Structure analytical model

#### Retrieving all Revit Database Elements

Regarding the ElementType instances, the initial version for Revit 2011 employed the FilteredElementCollector method WhereElementIsNotElementType, so no ElementType instances were displayed, including all family symbols.
This has now been fixed by using the following algorithm to retrieve all Revit database elements by creating a Boolean union of the non-element-type instances with the element type ones:
```csharp
  FilteredElementCollector elemTypeCtor
    = ( new FilteredElementCollector( doc ) )
      .WhereElementIsElementType();

  FilteredElementCollector notElemTypeCtor
    = ( new FilteredElementCollector( doc ) )
      .WhereElementIsNotElementType();

  FilteredElementCollector allElementCtor
    = elemTypeCtor.UnionWith( notElemTypeCtor );

  ICollection<Element> founds
    = allElementCtor.ToElements();
```

Please be aware that RevitLookup is always distributed including the full source code, so you can easily fix any problem that you run into yourself.

I did that and described the detailed procedure for some
[MEP specific properties](http://thebuildingcoder.typepad.com/blog/2009/08/fixing-rvtmgddbg-for-mep-connectors.html)
back in the Revit 2010 timeframe.

Here is today's snapshot of the
[updated RevitLookup](zip/RevitLookup_2010-05-19.zip),
including its full source code and Visual Studio solution.