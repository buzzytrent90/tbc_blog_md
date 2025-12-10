---
post_number: "0397"
title: "Do Not Filter For All Elements"
slug: "filter_all_elements"
author: "Jeremy Tammik"
tags: ['csharp', 'elements', 'filtering', 'levels', 'revit-api', 'views']
source_file: "0397_filter_all_elements.htm"
original_url: "https://thebuildingcoder.typepad.com/blog/0397_filter_all_elements.html"
---

### Do Not Filter For All Elements

One question that has come up several times is how to retrieve all elements from the Revit database.
This was the default behaviour in previous releases of the Revit API.

In the Revit 2011 API, the filtered element collectors have been intentionally designed to warn you if you try to retrieve all elements in a project, since this is not something that the Revit API designers expect a normal application ever to have to do. However, there are some special types of applications which do indeed require this kind of functionality, and it is very easy to work around this limitation.

One way to do so is simply to create two filtered element collector instances, one of them using any quick filter that you like, the other using the negation of it, and then creating the union of the results.

One example of doing so, and also a legitimate need for obtaining all Revit database elements is presented by the RevitLookup sample. The code it uses for that is located in the method InitializeTreeView and looks like this:
```csharp
  FilteredElementCollector coll
    = new FilteredElementCollector(
      m\_app.ActiveUIDocument.Document );

  coll.WherePasses(
    new LogicalOrFilter(
      new ElementIsElementTypeFilter( false ),
      new ElementIsElementTypeFilter( true ) ) );
```

**Addendum** from Scott Conover, Revit Software Development Manager:

Related to this post, I would like to point out even stronger NOT to iterate over all elements, for any reason whatsoever...

"Because the old API used to do it and my code was already written this way" is not a good reason.
That is simply an excuse for not making the application code cleaner and better.
It is very possible that misusing the new collector facility to iterate over all elements the new way will result in a larger memory footprint and poorer performance than in 2010.

We have found internally that all of the iterations performed at the UI level could take advantage of at least one, and often several, filters to improve performance and reduce the number of items to process.