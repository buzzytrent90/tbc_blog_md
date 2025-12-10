---
post_number: "0017"
title: "Filter Performance"
slug: "filter_performance"
author: "Jeremy Tammik"
tags: ['doors', 'elements', 'family', 'filtering', 'python', 'revit-api', 'walls', 'windows']
source_file: "0017_filter_performance.htm"
original_url: "https://thebuildingcoder.typepad.com/blog/0017_filter_performance.html"
---

### Filter Performance

This post is entirely thanks to Guy Robinson, who has always been the quickest to test and comment on performance issues in the Revit API, in response to the
[previous post](http://thebuildingcoder.typepad.com/blog/2008/10/relationship-in.html).
He says:

Like you, I initially thought the TypeFilter was a more elegant filter for FamilyInstances etc., but having done a lot of testing I now have my suspicions regarding how filters work internally and why it's not necessarily a good filter option for certain element types.

In the relationship code for this situation, it is significantly faster to just use the category filter as demonstrated in the command below. On a 100 MB project, I compared the times required to retrieve 571 elements using the type filter as shown in the previous post versus using an anonymous method to filter away objects of the wrong type after receiving them from Revit. These are the execution times for running the two commands a repeated number of times:

```
using TypeFilter:
Time to get 571 elements 37226
Time to get 571 elements  6042
Time to get 571 elements  6024
Time to get 571 elements  6172

using anonymous method:
Time to get 571 elements 33769
Time to get 571 elements  2052
Time to get 571 elements  2050
Time to get 571 elements  2048
Time to get 571 elements  2034
Time to get 571 elements  2037
```

As you can see, using the anonymous method is three times as fast. So while filters are fantastic, they're not always the best solution ;-)

So much for the comment from Guy, whom I would like to thank very much for this invaluable input. Here is the alternative code he is describing, using an anonymous method instead of the TypeFilter:

```python
class CmdFilterPerformance
{
  public CmdResult Execute(
    ExternalCommandData commandData,
    ref string message,
    ElementSet elements )
  {
    try
    {
      Application app = commandData.Application;
      Document doc = app.ActiveDocument;

      Stopwatch sw = Stopwatch.StartNew();

      // f5 = f1 && f4
      //    = f1 && (f2 || f3)
      //    = family instance and (door or window)

      Autodesk.Revit.Creation.Filter cf
        = app.Create.Filter;

      //Filter f1 = cf.NewTypeFilter(
      //  typeof( FamilyInstance ) );

      Filter f2 = cf.NewCategoryFilter(
        BuiltInCategory.OST\_Doors );
      Filter f3 = cf.NewCategoryFilter(
        BuiltInCategory.OST\_Windows );

      Filter f4 = cf.NewLogicOrFilter( f2, f3 );
      //Filter f5 = cf.NewLogicAndFilter( f1, f4 );

      //int n = doc.get\_Elements( f5, openings );

      List<Element> openings = new List<Element>();
      doc.get\_Elements( f4, openings );

      List<Element> openingInstances
        = openings.FindAll( delegate( Element element )
          {
            return !( element is Symbol );
          } );
      int n = openingInstances.Count;

      sw.Stop();

      Debug.WriteLine( string.Format(
        "Time to get {0} elements {1}",
        n, sw.ElapsedMilliseconds ) );

      return CmdResult.Succeeded;
    }
    catch( Exception ex )
    {
      message = ex.Message + ex.StackTrace;
      return CmdResult.Failed;
    }
  }
}
```

It really goes to show how extremely important testing on real life projects and scrupulous profiling and benchmarking is to ensure that you as a professional developer really deliver optimal quality to your clients. It also shows how easy it is for me to play around in an ivory tower and that my messages should always be taken with large grains of salt. And everybody else's too, of course. Guy has already pointed out some performance traps when blindly trusting Revit API filters to be fast in the past ... this one really brings the message across!

I am adding the complete Visual Studio solution here. This version 1.0.0.4 includes the four commands we discussed so far: CmdListWalls, CmdRelationshipInverter and CmdWallDimensions and the new CmdFilterPerformance.