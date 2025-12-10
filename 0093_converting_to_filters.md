---
post_number: "0093"
title: "Converting to Filters"
slug: "converting_to_filters"
author: "Jeremy Tammik"
tags: ['csharp', 'elements', 'family', 'filtering', 'levels', 'revit-api']
source_file: "0093_converting_to_filters.htm"
original_url: "https://thebuildingcoder.typepad.com/blog/0093_converting_to_filters.html"
---

### Converting to Filters

This is a follow-up on the
[category comparison](http://thebuildingcoder.typepad.com/blog/2009/01/category-comparison.html)
topic we brought up during the
[Revit Structure API training in Verona](http://thebuildingcoder.typepad.com/blog/2009/01/verona-revit-api-training.html),
Italy.
We first ran into a language problem in the Revit SDK FrameBuilder sample and fixed that by converting the language dependent category comparison code to a language independent version.
Stefano Carradore did not let it rest at that but went one step further and converted the old-fashioned and ineffective 2008-style iteration to a more effective style making use of the 2009-style API filtering functionality.

As noted in the discussion of the
[category comparison](http://thebuildingcoder.typepad.com/blog/2009/01/category-comparison.html),
the FrameBuilder sample works with structural beams and framing elements.
It implements a FrameData class which defines an Initialize() method to set up internal collections of column, beam and brace symbols.
They are sorted into different lists depending on their type, which is determined by comparing the category names.
In the original SDK sample, the iteration is performed over all Revit elements.
It picks out all the Level elements on one hand, and the structural framing and columns on the other.
The levels are identified by their class, i.e. System.Type, and the structural elements by their category name.
We already showed how to convert the category name comparison to a comparison of the category element id to remove the language dependence, which was preventing the code from running on the Italian version of Revit.
Still, the 2008-style iteration over the entire collection of all Revit elements remains extremely inefficient.

In Stefano's new implementation, we make use of two separate API filters to extract the levels and the structural elements. This has the additional advantage of being intrinsically language independent.

Here is the original code:

```csharp
ElementIterator itor
= m\_commandData.Application.ActiveDocument.Elements;
itor.Reset();
while (itor.MoveNext())
{
  object obj = itor.Current;

  // add level to list
  Level aLevel = obj as Level;
  if (null != aLevel)
  {
    m\_levels.Add(aLevel.Elevation, aLevel);
    continue;
  }

  // get Family to get FamilySymbols
  Family aFamily = obj as Family;
  if (null == aFamily)
  {
    continue;
  }

  foreach (FamilySymbol symbol in aFamily.Symbols)
  {
    if (null == symbol.Category)
    {
      continue;
    }

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
  }
}
```

We replace this code by two separate API filters.
We could also use three filters, replacing the loop sorting the structural elements into the two separate collections by two separate dedicated filters, one each for columns and framing elements:

```csharp
List<Element> a = new List<Element>();
doc.get\_Elements( typeof( Level ), a );

foreach( Level lev in a )
{
  m\_levels.Add( lev.Elevation, lev );
}

a.Clear();
Autodesk.Revit.Creation.Filter cf
  = app.Create.Filter;

Filter filterSymbols
  = cf.NewTypeFilter( typeof( FamilySymbol ) );
Filter filterCols
  = cf.NewCategoryFilter( bipColumn );
Filter filterFram
  = cf.NewCategoryFilter( bipFraming );
Filter filterStruct
  = cf.NewLogicOrFilter( filterCols, filterFram );
Filter filter
  = cf.NewLogicAndFilter( filterSymbols, filterStruct );

doc.get\_Elements( filter, a );

foreach( FamilySymbol symbol in a )
{
  ElementId categoryId = symbol.Category.Id;

  if( idFraming.Equals( categoryId ) )
  {
    m\_beambracesSymbolsMgr.AddSymbol( symbol );
  }
  else if( idColumn.Equals( categoryId ) )
  {
    m\_columnSymbolsMgr.AddSymbol( symbol );
  }
}
```

When filtering only for the System.Type FamilySymbol without the additional check for the structural categories, our sample project was returning over 500 family symbols.
Adding the category filters reduced this number to 40.

[Here](C:/a/lib/revit/2009/SDK/Samples/FrameBuilder/CS/FrameData.cs)
is the updated source code for
[FrameData.cs](C:/a/lib/revit/2009/SDK/Samples/FrameBuilder/CS/FrameData.cs)
with the three different variants enclosed in conditional compilation pragmas.