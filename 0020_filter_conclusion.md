---
ai_optimized: true
author: Jeremy Tammik
complexity_score: 2.7
content_type: documentation
optimization_date: '2025-12-11T11:44:13.241036'
original_url: https://thebuildingcoder.typepad.com/blog/0020_filter_conclusion.html
post_number: '0020'
reading_time_minutes: 2
series: filtering
slug: filter_conclusion
source_file: 0020_filter_conclusion.htm
tags:
- csharp
- doors
- elements
- family
- filtering
- parameters
- revit-api
- windows
title: Filter Performance Conclusion
word_count: 496
---

### Filter Performance Conclusion

Continuing the topic of the
[Revit API filter performance](http://thebuildingcoder.typepad.com/blog/2008/10/more-filter-per.html),
Guy responds to Ralf:

Ralf raises some good points.

A good point on the order in filters, I did not think about that and I did think the speed improvement was larger than normal. I've found in past testing using the TypeFilter in most cases 2-10% slower than using some form of .NET filtering for real world projects and where other filters are not appropriate.

I agree that an anonymous method is not as elegant; I haven't used this method in any production code. It just gave you a list as per your original code. Almost without exception, all my commands use the ElementIterator getElements() rather than populating a list. And for that reason the point I should have made in the previous post was that IMO I still think the TypeFilter is redundant. This is how I would have processed the elements in the command:

```csharp
  Filter f1 = cf.NewCategoryFilter(
    BuiltInCategory.OST\_Doors );
  Filter f2 = cf.NewCategoryFilter(
    BuiltInCategory.OST\_Windows );
  Filter f3 = cf.NewLogicOrFilter( f1, f2 );
  ElementIterator itor = doc.get\_Elements( f3 );
  int n = 0;
  while( itor.MoveNext() )
  {
    if( itor.Current is FamilyInstance )
    {
      n++;
      FamilyInstance elem = itor.Current
        as FamilyInstance;
    }
  }
```

Against the 100 MB project this is still 1-3% faster than including the type filter, even including the casting to a FamilyInstance.

The other point Ralf raises is that x >> y where x is the total element number and y is the number of elements of interest, e.g. of the desired category. I would go a step further and say for most commands x >> y >> z, where z is the number of particular instances of a family you are interested in processing. For that reason a vast majority of production commands I've written use parameter filtering, yet I have none with type filtering.

I've also found in testing that type filter performance is very dependent on whether a project has been recently purged of unused objects. As users will know, purging on a regular basis helps project performance. The difference in performance between a purged versus a non purged project for API commands can be very dramatic, as the total object count often drops significantly.

As you can tell, I'm not a big fan of the type filter but every project is different ;-)

For completeness sake, here is how I would use the ElementIterator in the original code:

```csharp
private Dictionary<ElementId, List<ElementId>>
  getElementIds( ElementIterator itor)
{
  Dictionary<ElementId, List<ElementId>> dict
    = new Dictionary<ElementId, List<ElementId>>();
  string fmt = "{0} is hosted by {1}";
  while( itor.MoveNext() )
  {
    object elem = itor.Current;
    if( elem is FamilyInstance )
    {
      FamilyInstance fi = elem as FamilyInstance;
      ElementId id = fi.Id;

      ElementId idHost = fi.Host.Id;

      Debug.WriteLine( string.Format( fmt,
        ElementDescription( fi ),
        ElementDescription( idHost ) ) );
      if( !dict.ContainsKey( idHost ) )
      {
        dict.Add( idHost, new List<ElementId>() );
      }
      dict[idHost].Add( id );
    }
  }
  return dict;
}
```