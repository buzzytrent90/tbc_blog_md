---
post_number: "1711"
title: "Dimension Line Weight"
slug: "dimension_line_weight"
author: "Jeremy Tammik"
tags: ['elements', 'parameters', 'revit-api', 'sheets', 'views']
source_file: "1711_dimension_line_weight.md"
original_url: "https://thebuildingcoder.typepad.com/blog/1711_dimension_line_weight.html"
---

### Accessing Dimension Line Weight
Today we share a quickie from
the [Revit API discussion forum](http://forums.autodesk.com/t5/revit-api-forum/bd-p/160) thread
on how to [access the line weight for dimension lines](https://forums.autodesk.com/t5/revit-api-forum/access-line-weight-for-dimension-lines/m-p/8463046):
\*\*Question:\*\* I want to programmatically access a dimension line's line weight.
Here is a screenshot of what I am after:
![Dimension line weight](img/dimension_line_weight.png)

Dimension line weight

\*\*Answer:\*\* Before you do anything else whatsoever, install [RevitLookup](https://github.com/jeremytammik/RevitLookup).
It is an interactive Revit BIM database exploration tool to view and navigate element properties and relationships.
Use that to discover the relationships between the dimension lines, their styles, and the properties you are looking for.
Once you have that installed, you will quickly be able to determine that the dimension line weight is stored in a parameter value on the dimension type.
You can access it like this using the built-in parameter enumeration value `BuiltInParameter.LINE_PEN`:
```csharp
ElementId dimTypeId = dimension.GetTypeId();
Element dimType = document.GetElement( dimTypeId );
Parameter lineWeight = dimType.get_Parameter(
BuiltInParameter.LINE_PEN );
```
Many thanks to [Bardia Jahangiri](https://forums.autodesk.com/t5/user/viewprofilepage/user-id/4145125) for the precise detailed answer.