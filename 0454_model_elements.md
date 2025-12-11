---
ai_optimized: true
author: Jeremy Tammik
complexity_score: 2.2
content_type: qa
optimization_date: '2025-12-11T11:44:13.973167'
original_url: https://thebuildingcoder.typepad.com/blog/0454_model_elements.html
post_number: '0454'
reading_time_minutes: 2
series: elements
slug: model_elements
source_file: 0454_model_elements.htm
tags:
- csharp
- elements
- filtering
- revit-api
- selection
- transactions
- views
title: Model Elements Revisited
word_count: 328
---

### Model Elements Revisited

Scott Conover responded with a wealth of ideas to yesterday's
[model element selection](http://thebuildingcoder.typepad.com/blog/2010/10/selecting-model-elements.html) discussion,
which in turn has gone through numerous previous iterations, so this really seems like an endless theme.

Actually it is not even yesterday's, yet, it is still today's.
We both wanted to fix it fast.

Here are Scott's comments and suggestions:

First, on the original solution:
```csharp
  IList<Element> found
    = collector
      .WhereElementIsNotElementType()
      .WhereElementIsViewIndependent()
      .WherePasses( new LogicalOrFilter(
        new ElementIsElementTypeFilter( false ),
        new ElementIsElementTypeFilter( true ) ) )
      .ToElements();
```

The WherePasses clause is totally unnecessary.
I know this is the 'original' example, but I certainly hope that nobody copies this for further use.

Your 'first solution' post from 2009 appends filters to each other one at a time.
Our new LogicalOrFilters support more than two inputs, so it should actually be updated to make more efficient use of 2011 filtering.

Regarding your second solution regarding HasMaterialQuantities – I guess it may not find all required elements.
Furniture, for example, doesn't have material quantities, but it is a physical element in the model.

Another solution might be:

- Start temporary transaction- Create a default 3D view- Create FilteredElementCollection on view elements.- Apply WhereElementIsNotElementType- Iteration- Rollback to remove created view

Since it seems that 3D elements are what is wanted here, all elements visible in the new (unfiltered) 3D view are targets.

Finally, the final solution contains a redundancy:
```csharp
  FilteredElementCollector collector
    = new FilteredElementCollector( doc );

  collector
    .WhereElementIsNotElementType()
    .WhereElementIsViewIndependent()
    .ToElements();

  foreach( Element element in collector )
  {
    // . . .
  }
```

The redundancy consists in the call to ToElements.
It executes the filter over the document and returns all found elements in a collection.
Foreach over the collector does the same execution a second time.
You should either skip the ToElements call (which is what I would do), or declare a local IList to hold the return of ToElements and run foreach over that.

Many thanks to Scott for this valuable feedback!