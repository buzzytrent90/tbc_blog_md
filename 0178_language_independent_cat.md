---
post_number: "0178"
title: "Language Independent Category Access"
slug: "language_independent_cat"
author: "Jeremy Tammik"
tags: ['csharp', 'elements', 'parameters', 'revit-api', 'selection']
source_file: "0178_language_independent_cat.htm"
original_url: "https://thebuildingcoder.typepad.com/blog/0178_language_independent_cat.html"
---

### Language Independent Category Access

I recently discussed
[category comparison](http://thebuildingcoder.typepad.com/blog/2009/06/category-comparison-and-model-element-selection-revisited.html),
and I keep stressing the importance of keeping the application implementation language independent as far as possible.
Jiri Smerak of
[Ing.-Software Dlubal GmbH](http://dlubal.de)
now uncovered a situation in which due to a localisation issue the language independent category comparison is in fact the only way to access certain categories in the German version of Revit Structure.

The issue has to do with the following built-in categories:

- OST\_LoadCasesLive- OST\_LoadCasesRoofLive- OST\_LoadCasesSnow- OST\_LoadCasesWind

When trying to access these from the document settings Categories collection using the get\_Item method, the German version of Revit returns null.
In the English version, they work fine and return the correct category objects.
The reason for this is that somewhere internally, a string comparison is taking place.
In the German version, all four of these categories are represented by the same string in the user interface.

Using the
[language independent category comparison](http://thebuildingcoder.typepad.com/blog/2009/06/category-comparison-and-model-element-selection-revisited.html),
it is possible to compare the built-in enum value with an element category id directly, bypassing all string comparisons, e.g.:

```csharp
  if ( (int) BuiltInCategory.OST\_Dimensions
    == dimension.Category.Id.Value )
  {
    message += "\nDimension is a permanent dimension.";
  }
```

The same comparison can be used to detect which category belongs to the desired load cases.
It works well and it doesn't depend on the language of the application.
```csharp
  LoadCase lc = element as LoadCase;

  Parameter param = lc.get\_Parameter(
    BuiltInParameter.LOAD\_CASE\_CATEGORY );

  ElementId categoryId = param.AsElementId();

  if( categoryId.Value.Equals(
    (int) BuiltInCategory.OST\_LoadCasesLive ) )
  {
    // . . .
  }
```

Many thanks to Jiri for pointing this out!