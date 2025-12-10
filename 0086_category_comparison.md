---
post_number: "0086"
title: "Category Comparison"
slug: "category_comparison"
author: "Jeremy Tammik"
tags: ['csharp', 'elements', 'revit-api']
source_file: "0086_category_comparison.htm"
original_url: "https://thebuildingcoder.typepad.com/blog/0086_category_comparison.html"
---

### Category Comparison

Still giving Revit Structure API training in Verona, Italy, we ran into a problem in one of the Revit SDK samples today.
The FrameBuilder sample works with structural beams and framing elements.
It implements a FrameData class which defines an Initialize() method to set up internal lists of column, beam and brace symbols.
These are sorted into different lists depending on their type, which is determined by comparing the category names.
Unfortunately, the original SDK sample compares the category names with the language dependent strings "Structural Framing" and "Structural Columns".
In the Italian version of Revit, these strings change to "Pilastri strutturali" and "Telaio strutturale" and the comparison will never succeed.

Here is the original, language dependent comparison code that we would like to replace by language independent code:

```csharp
// add symbols to lists according to category name
string categoryName = symbol.Category.Name;
if ("Structural Framing" == categoryName)
{
  m\_beambracesSymbolsMgr.AddSymbol(symbol);
}
else if ("Structural Columns" == categoryName)
{
  m\_columnSymbolsMgr.AddSymbol(symbol);
}
```

To replace this by language independent code, we make use of the built-in category enumeration.
We use the enumeration values OST\_StructuralColumns and OST\_StructuralFraming to obtain the corresponding document categories from the document settings and its categories collections, and store their element ids:

```csharp
Document doc =
  m\_commandData.Application.ActiveDocument;

Categories categories
  = doc.Settings.Categories;

BuiltInCategory bipColumn
  = BuiltInCategory.OST\_StructuralColumns;

BuiltInCategory bipFraming
  = BuiltInCategory.OST\_StructuralFraming;

ElementId idColumn
  = categories.get\_Item( bipColumn ).Id;

ElementId idFraming
  = categories.get\_Item( bipFraming ).Id;
```

This can be done right at the beginning of the Intialize() method.
Then we can replace the language dependent code listed above by:

```csharp
// add symbols to lists according to category
ElementId categoryId = symbol.Category.Id;
if( idFraming.Equals( categoryId ) )
{
  m\_beambracesSymbolsMgr.AddSymbol( symbol );
}
else if( idColumn.Equals( categoryId ) )
{
  m\_columnSymbolsMgr.AddSymbol( symbol );
}
```

Note that in previous versions of Revit, 2008 and earlier, it was possible to compare the categories directly, i.e. use something like this:

```csharp
Category cat = symbol.Category;
if( catFraming.Equals( cat ) )
{
  m\_beambracesSymbolsMgr.AddSymbol( symbol );
}
else if( catColumn.Equals( cat ) )
{
  m\_columnSymbolsMgr.AddSymbol( symbol );
}
```

This stopped working reliably in Revit 2009, so nowadays you need to compare the category element ids instead of directly comparing the categories.