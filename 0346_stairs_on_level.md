---
post_number: "0346"
title: "Retrieve Stairs on Level"
slug: "stairs_on_level"
author: "Jeremy Tammik"
tags: ['csharp', 'elements', 'filtering', 'geometry', 'levels', 'parameters', 'revit-api']
source_file: "0346_stairs_on_level.htm"
original_url: "https://thebuildingcoder.typepad.com/blog/0346_stairs_on_level.html"
---

### Retrieve Stairs on Level

As mentioned in the discussion on
[selecting model elements](http://thebuildingcoder.typepad.com/blog/2009/05/selecting-model-elements.html),
stairs are not represented by an own class in the Revit API, but using the generic Revit Element class instead.
They do have a valid built-in category assigned to them, however, which makes it easy to retrieve them from the database and use the generic element and parameter access to retrieve and modify a lot of their data.
We also discussed other aspects of stairs in the past, such as
[listing the railing types](http://thebuildingcoder.typepad.com/blog/2009/02/list-railing-types.html),
[material quantity extraction](http://thebuildingcoder.typepad.com/blog/2010/02/material-quantity-extraction.html), and
[geometry retrieval](http://thebuildingcoder.typepad.com/blog/2010/02/retrieving-column-and-stair-geometry.html).

Rocky now raised a
[question](http://thebuildingcoder.typepad.com/blog/2009/05/selecting-model-elements.html?cid=6a00e553e1689788330133ecc952ca970b#comment-6a00e553e1689788330133ecc952ca970b) on
retrieving stair elements from the database that allows us to take another quick look at the new Revit 2011 filtering capabilities:

**Question:** Will you please help me out to know how to retrieve the stairs on certain levels?
E.g., if there are two stairs on the first level of the building, then on second level, how can we get these stairs?

**Answer:** Retrieving all the stairs on a given level is easy.
We can use the stairs built-in category OST\_Stairs to identify the stairs themselves, and the Element class Level property or an appropriate built-in parameter to determine what level they are on.
Since the Revit filtering API is so flexible and powerful, it provides us with a number of options for the approach to use:

- Explicit iteration and manual checking of a property.- Using [LINQ](http://thebuildingcoder.typepad.com/blog/2009/07/language-integrated-query-linq.html).- Using an [anonymous method](http://thebuildingcoder.typepad.com/blog/2010/04/anonymous-methods-in-vb.html).- Using a parameter filter.

We have demonstrated examples of all of these in several recent posts, e.g. in our analysis of
[collector performance](http://thebuildingcoder.typepad.com/blog/2010/04/collector-benchmark.html).

In all of the approaches above, one would obviously first apply a filter to check for the built-in category, for two reasons:

- First, it is a quick filter, so it should be applied before any slow filters or other processing.- Secondly, we know that it will eliminate the vast majority of all the Revit database elements, so very few elements will remain to check.

The first three options listed above all make use of post-processing of the results returned by the quick category filter, and are more or less equivalent in speed.
Below, we present untested source code sample implementation snippets for all three of these approaches:
Here is the built-in stair category constant and the element id of the level that we are searching for:
```csharp
  ElementId id = level.Id;

  BuiltInCategory bic
    = BuiltInCategory.OST\_Stairs;
```

Here is the retrieval using explicit iteration and manual checking of a property:
```csharp
  FilteredElementCollector collector
    = new FilteredElementCollector( doc );

  collector.OfCategory( bic );

  List<Element> stairs = new List<Element>();

  foreach( Element e in collector )
  {
    if( e.Level.Id.Equals( id ) )
    {
      stairs.Add( e );
    }
  }
```

Using LINQ, it might look like this:
```csharp
  FilteredElementCollector collector
    = new FilteredElementCollector( doc );

  collector.OfCategory( bic );

  IEnumerable<Element> stairsOnLevelLinq =
    from e in collector
    where e.Level.Id.Equals( id )
    select e;
```

Using an anonymous method, it is even shorter:
```csharp
  FilteredElementCollector collector
    = new FilteredElementCollector( doc );

  collector.OfCategory( bic );

  IEnumerable<Element> stairsOnLevelAnon =
    collector.Where<Element>( e
      => e.Level.Id.Equals( id ) );
```

As said, the test of the built-in category uses a quick filter, so that is good.
The post-processing is very expensive and not optimal, though.

As we demonstrated in our
[collector performance](http://thebuildingcoder.typepad.com/blog/2010/04/collector-benchmark.html) analysis,
a parameter filter is twice as fast as post-processing the results, even though it is a slow filter and not a quick one.

To use a parameter filter on the stair elements retrieved by the category filter, we need a built-in parameter to test, instead of the Element Level property.
The stair object stores its base and top levels in the built-in parameters STAIRS\_BASE\_LEVEL\_PARAM and STAIRS\_TOP\_LEVEL\_PARAM, so that is no problem.

Here is an untested source code example of setting up a category and a parameter filter to retrieve all stairs on a given level:
```csharp
  FilteredElementCollector collector
    = new FilteredElementCollector( doc );

  collector.OfCategory( bic );

  BuiltInParameter bip
    = BuiltInParameter.STAIRS\_BASE\_LEVEL\_PARAM;

  ParameterValueProvider provider
    = new ParameterValueProvider(
      new ElementId( bip ) );

  FilterNumericRuleEvaluator evaluator
    = new FilterNumericEquals();

  FilterRule rule = new FilterElementIdRule(
    provider, evaluator, id );

  ElementParameterFilter filter
    = new ElementParameterFilter( rule );

  return collector.WherePasses( filter );
```

I hope that fully answers your question, Rocky, and provides a useful working example for many others as well.