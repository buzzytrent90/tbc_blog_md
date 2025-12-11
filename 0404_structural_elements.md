---
ai_optimized: true
author: Jeremy Tammik
complexity_score: 1.9
content_type: qa
optimization_date: '2025-12-11T11:44:13.893308'
original_url: https://thebuildingcoder.typepad.com/blog/0404_structural_elements.html
post_number: '0404'
reading_time_minutes: 3
series: elements
slug: structural_elements
source_file: 0404_structural_elements.htm
tags:
- csharp
- elements
- family
- filtering
- revit-api
- walls
title: Retrieve Structural Elements
word_count: 542
---

### Retrieve Structural Elements

I recently discussed the retrieval of all Revit
[MEP elements and their connectors](http://thebuildingcoder.typepad.com/blog/2010/06/retrieve-mep-elements-and-connectors.html).
Here is a similar question on retrieving structural elements:

**Question:** How can I get all the structural elements like structural walls, slabs, columns, and beams from the model?
I have a problem with the columns and beams because they are family instances and I cannot just check only their type.
I currently have to go through all the family instances and check whether they are a column or beam to be modified in my model.

**Answer:** Yes, certainly you can efficiently and directly retrieve all structural elements from a Revit model database.
The key to doing so is to use the filtered element collectors.
All you need to do is identify certain criteria by which to identify all your elements and encode these into filters to apply to the collector.

One approach to achieve this is described for the
[retrieval of MEP elements](http://thebuildingcoder.typepad.com/blog/2010/06/retrieve-mep-elements-and-connectors.html),
which shows how to set up a combined filtered element collector which restricts the categories of the family instances.

In that solution, some elements such as ducts and pipes that can be directly identified by their System.Type.

Others such as equipment and fittings are represented by family instances and can be further identified by their category. For instance, for the MEP elements, we specified both the System.Type FamilyInstance as well as categories such as duct fitting, duct terminal, and pipe fitting.

To set up a similar filter for your structural elements, you will have to analyse your model to determine exactly which categories your columns and beams have.

Here is an example to retrieve certain structural elements that I make use of for the RST link application:
```csharp
FilteredElementCollector GetStructuralElements(
  Document doc )
{
  // what categories of family instances
  // are we interested in?

  BuiltInCategory[] bics = new BuiltInCategory[] {
    BuiltInCategory.OST\_StructuralColumns,
    BuiltInCategory.OST\_StructuralFraming,
    BuiltInCategory.OST\_StructuralFoundation
  };

  IList<ElementFilter> a
    = new List<ElementFilter>( bics.Count() );

  foreach( BuiltInCategory bic in bics )
  {
    a.Add( new ElementCategoryFilter( bic ) );
  }

  LogicalOrFilter categoryFilter
    = new LogicalOrFilter( a );

  LogicalAndFilter familyInstanceFilter
    = new LogicalAndFilter( categoryFilter,
      new ElementClassFilter(
        typeof( FamilyInstance ) ) );

  IList<ElementFilter> b
    = new List<ElementFilter>( 6 );

  b.Add( new ElementClassFilter(
    typeof( Wall ) ) );

  b.Add( new ElementClassFilter(
    typeof( Floor ) ) );

  b.Add( new ElementClassFilter(
    typeof( ContFooting ) ) );

  b.Add( new ElementClassFilter(
    typeof( PointLoad ) ) );

  b.Add( new ElementClassFilter(
    typeof( LineLoad ) ) );

  b.Add( new ElementClassFilter(
    typeof( AreaLoad ) ) );

  b.Add( familyInstanceFilter );

  LogicalOrFilter classFilter
    = new LogicalOrFilter( b );

  FilteredElementCollector collector
    = new FilteredElementCollector( doc );

  collector.WherePasses( classFilter );

  return collector;
}
```

You will have to analyse your model and see whether this filtering really suits your needs and whether all elements you are interested in are covered.

In addition to the explicit filtering demonstrated here, which gives you full and detailed control over what exactly is retrieved, the StructuralInstanceUsageFilter might fit your needs more exactly with less effort.
You might also find the StructuralMaterialTypeFilter useful.
In any case, you will have to test them in your specific models.

Please refer to the
[MEP element retrieval](http://thebuildingcoder.typepad.com/blog/2010/06/retrieve-mep-elements-and-connectors.html) for
links to further discussions and examples of filtered element collectors.