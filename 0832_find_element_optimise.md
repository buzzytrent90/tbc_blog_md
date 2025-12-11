---
ai_optimized: true
author: Jeremy Tammik
complexity_score: 3.4
content_type: code_example
optimization_date: '2025-12-11T11:44:14.695613'
original_url: https://thebuildingcoder.typepad.com/blog/0832_find_element_optimise.html
post_number: 0832
reading_time_minutes: 4
series: elements
slug: find_element_optimise
source_file: 0832_find_element_optimise.htm
tags:
- csharp
- doors
- elements
- family
- filtering
- parameters
- python
- references
- revit-api
- selection
- walls
title: FindElement and Collector Optimisation
word_count: 853
---

﻿

### FindElement and Collector Optimisation

As I recently
[mentioned](http://thebuildingcoder.typepad.com/blog/2012/09/video-animated-ribbon-item-tooltip.html#2),
Mikako Harada published a very nice article on
[programmatically adjusting beam cutback](http://adndevblog.typepad.com/aec/2012/09/adjusting-cutback-programmatically-.html).

It was followed it up by a description of some commonly used
[helper methods](http://adndevblog.typepad.com/aec/2012/09/helper-functions-findfamilytype-and-findelement.html) to
find certain named elements and family symbols in a project.
One of them is FindElement, implemented as follows in that post:
```python
  public static Element FindElement(
    Document doc,
    Type targetType,
    string targetName )
  {
    // Get the elements of the given class

    FilteredElementCollector collector
      = new FilteredElementCollector( doc );

    collector.WherePasses(
      new ElementClassFilter( targetType ) );

    // Parse the collection for the
    // given name using LINQ query.

    IEnumerable<Element> targetElems =
      from element in collector
      where element.Name.Equals( targetName )
      select element;

    IList<Element> elems = targetElems.ToList();

    if( elems.Count > 0 )
    {
      // We should have only one with the given name.

      return elems[0];
    }

    // Cannot find it.

    return null;
  }
```

It is important to be aware that this method can be optimised further in some aspects and should not be used indiscriminately.

Here are some points I would like to highlight:

1. Language dependence: should be avoided if possible.- Speed: could be improved by using a
     [parameter filter instead of LINQ](http://thebuildingcoder.typepad.com/blog/2010/06/element-name-parameter-filter-correction.html).- Conversion from a filtered element collector to .NET collections can often be avoided.

I avoid using the helper methods in this form wherever I can if performance is a factor.
When is it not?

It is not always possible to avoid the language dependence.
In any case, it is definitely important to be aware of the issue.

In detail:

1. Language dependence: if there is any way to identify the target element except by name, it is normally preferable to do so.- Speed: using a parameter filter to find a name may be complicated by the fact that different Revit element types store their names in different parameter values.
     The LINQ query can mostly be replaced by a parameter filter, though, which normally halves the execution time, since the marshalling of all the non-target data from internal Revit memory to external .NET and LINQ space is eliminated.- A filtered element collector is already iterable, so the conversion to an IEnumerable<Element> and to IList<Element> is often unnecessary.
       If all filtering can be achieved using filtered element collector functionality, the instantiation of additional lists and consequent duplication and copying of all contained data members can be completely avoided.
       This is obviously especially important for large collections.

I mentioned performance hints such as these numerous times in the past between the lines.
Maybe the time is ripe now to bring them up as a topic of their own.

Mikako underlines that the main intention of the helper method above is to make it as easy as possible for the reader to copy and paste these code snippets to quickly run the test command instead of having to download, explore and install the whole ADN training material zip file, and performance is a completely secondary consideration.

If performance becomes a bottleneck, each developer needs examine it and implement her own optimised version.
Accessing a wall type, for example, does not require this kind of filtering from the whole element list.

#### Collector Optimisation

While we are on the topic of efficient coding, here is another snippet of typical add-in code presenting a surprising number of opportunities for improvement in a very few lines.

It retrieves all family symbols in the document, uses them to find door families, and processes each one in turn:
```csharp
  FilteredElementCollector collector
    = new FilteredElementCollector( doc );

  FilteredElementIterator itor = collector
    .OfClass( typeof( FamilySymbol ) )
    .GetElementIterator();

  itor.Reset();

  while( itor.MoveNext() )
  {
    FamilySymbol symbol = itor.Current
      as Autodesk.Revit.DB.FamilySymbol;

    // Determine family category

    Category cat = symbol.Category;

    // Process family if doors category

    if( cat != null )
    {
      if( cat.Name == "Doors" )
      {
        Family family = symbol.Family;

        // Process reference to doors family
      }
    }
  }
```

Can you spot three possibilities for improvement, either a more succinct formulation, performance enhancement, or both?
And manage not to peek?

Here are the ones I found, which is not to say that there are no others:

- The code can be significantly shortened by using foreach directly on the collector instead of explicitly setting up and using an iterator.- You should avoid explicit string comparison when you can, both for reasons of performance and language independence, e.g. use the built-in category for doors instead of the category name, which might be localised.- In this case, you could let the collector handle the category selection for you, making the code significantly more efficient.

Applying these suggestions produces this instead:
```csharp
  FilteredElementCollector collector
  = new FilteredElementCollector( doc )
    .OfCategory( BuiltInCategory.OST\_Doors )
    .OfClass( typeof( FamilySymbol ) );

  foreach( FamilySymbol symbol in collector )
  {
    Family family = symbol.Family;

    // Process reference to doors family
  }
```

Shorter, more readable, and more performant.

Since each family can contain more than one symbol, you should obviously keep track of the families already processed and skip those when looping over the symbols.