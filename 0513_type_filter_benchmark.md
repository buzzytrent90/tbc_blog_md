---
ai_optimized: true
author: Jeremy Tammik
complexity_score: 2.9
content_type: code_example
optimization_date: '2025-12-11T11:44:14.082128'
original_url: https://thebuildingcoder.typepad.com/blog/0513_type_filter_benchmark.html
post_number: '0513'
reading_time_minutes: 3
series: filtering
slug: type_filter_benchmark
source_file: 0513_type_filter_benchmark.htm
tags:
- doors
- elements
- family
- filtering
- levels
- python
- revit-api
- transactions
- views
- windows
title: Type Filter Benchmark Update
word_count: 523
---

### Type Filter Benchmark Update

We have come to trust the Revit 2011 API filters to really be the most performant approach to retrieve sets of database elements fulfilling a complex combination of criteria.
To reinforce that message, here is another benchmark by Piotr Zurek, who says:

While searching for some answers in your blog I have stumbled across an old post where Guy Robinson compared the performance of the Revit 2010 API
[class filter vs an anonymous method](http://thebuildingcoder.typepad.com/blog/2008/10/filter-performa.html).

I found the result really intriguing since I always assumed that the filter would be quicker.
Since the post was a bit old I decided to check how those results would look in the current version.
I also decided to add a third element to the comparison – filtering using LINQ.
You know, just for the kicks... :-)

I'm happy to report that the situation has improved, by which I mean the speed of the "elegant" Revit filter is on par with both other methods.
I probably should have tried to test it on a bigger model with more elements to filter to get a bit measurable result but what I got clearly shows that the 3x difference is gone.
Here are my results from five runs:

| Method |  | Time in ms to retrieve 860 elements | | | | |
| --- | --- | --- | --- | --- | --- | --- |
| Class filter |  | 108 | 40 | 36 | 37 | 37 |
| Anonymous method |  | 192 | 36 | 39 | 38 | 37 |
| LINQ |  | 129 | 37 | 37 | 39 | 39 |

Here is the
[source code](https://gist.github.com/772300) used, reproduced here below for your convenience:
```python
[Transaction( TransactionMode.Manual )]
[Regeneration( RegenerationOption.Manual )]
public class Commands : IExternalCommand
{
  public Result Execute(
      ExternalCommandData commandData,
      ref string message,
      ElementSet elements )
  {
    try
    {
      UIApplication uiApp = commandData.Application;
      UIDocument uiDoc = uiApp.ActiveUIDocument;
      Application app = uiApp.Application;
      Document doc = uiDoc.Document;

      Stopwatch sw = Stopwatch.StartNew();

      // f5 = f1 && f4
      // = f1 && (f2 || f3)
      // = family instance and (door or window)

      #region Filters and collector definitions

      ElementClassFilter f1
        = new ElementClassFilter(
          typeof( FamilyInstance ) );

      ElementCategoryFilter f2
        = new ElementCategoryFilter(
          BuiltInCategory.OST\_Doors );

      ElementCategoryFilter f3
        = new ElementCategoryFilter(
          BuiltInCategory.OST\_Windows );

      LogicalOrFilter f4
        = new LogicalOrFilter( f2, f3 );

      LogicalAndFilter f5
        = new LogicalAndFilter( f1, f4 );

      FilteredElementCollector collector
        = new FilteredElementCollector( doc );

      #endregion

      //#region Filtering with a class filter
      //List<Element> openingInstances =
      //  collector.WherePasses(f5).ToElements()
      //    as List<Element>;
      //#endregion

      //#region Filtering with an anonymous method
      //List<Element> openings = collector
      //  .WherePasses(f4)
      //  .ToElements() as List<Element>;
      //List<Element> openingInstances
      //  = openings.FindAll(
      //    e => e is FamilyInstance );
      //#endregion

      #region Filtering with LINQ
      List<Element> openings = collector
        .WherePasses( f4 )
        .ToElements() as List<Element>;

      List<Element> openingInstances
        = ( from instances in openings
          where instances is FamilyInstance
          select instances ).ToList<Element>();
      #endregion

      int n = openingInstances.Count;
      sw.Stop();

      Debug.WriteLine( string.Format(
        "Time to get {0} elements: {1}ms",
        n, sw.ElapsedMilliseconds ) );

      return Result.Succeeded;
    }
    catch( Exception ex )
    {
      message = ex.Message + ex.StackTrace;
      return Result.Failed;
    }
  }
}
```

Many thanks to Piotr for testing and sharing these results!

By the way, this comparison and the results are quite comparable to the
[level filter benchmark](http://thebuildingcoder.typepad.com/blog/2010/10/level-filter-benchmark.html) included
in Kevin Vandecar's
[filtered element collector overview and benchmarking suite](http://thebuildingcoder.typepad.com/blog/2010/10/filtered-element-collectors.html).