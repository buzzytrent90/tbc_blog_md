---
post_number: "0464"
title: "Level Filter Benchmark"
slug: "level_filter_benchmark"
author: "Jeremy Tammik"
tags: ['elements', 'family', 'filtering', 'levels', 'parameters', 'python', 'references', 'revit-api', 'rooms', 'transactions']
source_file: "0464_level_filter_benchmark.htm"
original_url: "https://thebuildingcoder.typepad.com/blog/0464_level_filter_benchmark.html"
---

### Level Filter Benchmark

Yesterday I published Kevin Vandecar's handout document from his AEC DevCamp presentation on
[filtered element collectors](http://thebuildingcoder.typepad.com/blog/2010/10/filtered-element-collectors.html),
and right away an interesting case cropped up which motivated a closer look at one of his samples.
By the way, many of his samples are very interesting to have a closer look at, which I plan to do soon as well.
For now, let's start off with an analysis of the ElementLevelFilter, based on the following question:

**Question:** A quick question this time:
I want to retrieve a set of elements filtered by the floor level they are on.
For example – get all the rooms on Level 1.

I can use the Level property, but I understand this would be significantly slower than a parameter filter.

Among the countless built-in parameters, I cannot seem to find one for this purpose.

Any suggestions?

**Answer:** Thank you for your short query.
The answer has turned out to be significantly longer and more interesting than I initially expected.

Just as you say, you could simply postprocess your elements by querying the Level property using LINQ or explicit coding, and that would indeed be slower than using some built-in Revit element filtering functionality.

To begin with, I thought that I can answer your question twice over by referring to the
[parameter filter](http://thebuildingcoder.typepad.com/blog/2010/06/parameter-filter.html#1) blog post.

On one hand, it answers your question by pointing to the ElementLevelFilter, which is a slow filter used to match elements by their associated level, exactly as you require.
Example code for using this filter is given in the Revit API help document RevitAPI.chm under 'ElementLevelFilter Class'.

On the other hand, the post also points out that this filter cannot be used for beams, since its Level property is not implemented, and for those you can use a parameter filter checking for the built-in parameter INSTANCE\_REFERENCE\_LEVEL\_PARAM instead.
This parameter is probably not applicable for all elements, though, so you might have to use some other one for other element types.

Maybe (and hopefully) the ElementLevelFilter will fulfil all your specific needs.

On the third hand, assuming we don't run out of hands, I am just working on Kevin Vandecar's filtered element collector benchmark code which he created for AEC DevCamp in Boston back in June, and it includes one sample command which exactly answers your question and includes some benchmark code to prove the point.

The command FilterParameterEx2 determines all family instances on a specific level using and benchmarking three different methods:

- ElementLevelFilter.- ElementParameterFilter based on the built-in FAMILY\_LEVEL\_PARAM parameter.- LINQ post-processing check of the family instance Level property.

Here is the complete code of the command:
```python
[Transaction( TransactionMode.Automatic )]
[Regeneration( RegenerationOption.Manual )]
public class FilterParameterEx2 : IExternalCommand
{
  static string \_level\_name = "Level 2"; // "02 - Floor"

  public Result Execute(
    ExternalCommandData commandData,
    ref string message,
    ElementSet elements )
  {
    UIApplication uiapp = commandData.Application;
    UIDocument uidoc = uiapp.ActiveUIDocument;
    Application app = uiapp.Application;
    Document doc = uidoc.Document;

    // Level 2 example criteria
    ElementId levelId = Util.FindLevelId( doc,
      \_level\_name );

    Stopwatch sw = new Stopwatch();
    string names = string.Empty;

    // Use the Level filter to find all
    // FamilyInstances with desired Level.
    // Note this is a slow filter.

    sw.Reset();
    sw.Start();

    ElementLevelFilter filterElementsOnLevel
      = new ElementLevelFilter( levelId );

    FilteredElementCollector collector1
      = new FilteredElementCollector( doc );

    collector1.OfClass( typeof( FamilyInstance ) )
      .WherePasses( filterElementsOnLevel );

    ICollection<Element> listLevel
      = collector1.ToElements();

    // Now that we have only what we want,
    // foreach process them into a report.

    names = string.Empty;
    foreach( FamilyInstance instance in listLevel )
    {
      names += "\nLevel Name = " + instance.Level.Name
        + "   Instance name = " + instance.Name
        + "   id: " + instance.Id.ToString();
    }

    sw.Stop();

    Util.ShowElapsedTime( sw,
      "Using ElementLevelFilter to find "
      + listLevel.Count().ToString()
      + " family instances on a given level: "
      + names );

    // Use ElementParameterFilter to find all
    // FamilyInstances with desired Level by
    // finding the data in the FAMILY\_LEVEL\_PARAM
    // parameter.

    sw.Reset();
    sw.Start();

    BuiltInParameter bip
      = BuiltInParameter.FAMILY\_LEVEL\_PARAM;

    ParameterValueProvider provider = new
      ParameterValueProvider( new ElementId( bip ) );

    FilterNumericRuleEvaluator evaluator
      = new FilterNumericEquals();

    FilterRule rule = new FilterElementIdRule(
      provider, evaluator, levelId );

    ElementParameterFilter filter
      = new ElementParameterFilter( rule );

    FilteredElementCollector collector2
      = new FilteredElementCollector( doc );

    collector2.OfClass( typeof( FamilyInstance ) )
      .WherePasses( filter );

    ICollection<Element> listLevelParam
      = collector2.ToElements();

    // Now that we have only what we want,
    // foreach process them into a report.

    names = string.Empty;
    foreach( FamilyInstance instance in listLevelParam )
    {
      names += "\nLevel Name = " + instance.Level.Name
        + "   Instance name = " + instance.Name
        + "   id: " + instance.Id.ToString();
    }

    sw.Stop();

    Util.ShowElapsedTime( sw,
      "Using Element Parameter Filter: "
      + listLevelParam.Count().ToString() + names );

    // Use LINQ to find all FamilyInstances
    // with desired Level.

    sw.Reset();
    sw.Start();
    FilteredElementCollector collector3
      = new FilteredElementCollector( doc );

    collector3.OfClass( typeof( FamilyInstance ) );

    IEnumerable<FamilyInstance> listLevelLINQ
      = collector3.OfType<FamilyInstance>();

    // Use LINQ to filter down those that
    // match the desired level name.

    Level level = null;
    IEnumerable<FamilyInstance> listFiOnLevelLINQ
      = from fi in listLevelLINQ
        where ( ( level = fi.Level ) != null )
          && ( level.Id.Equals( levelId ) )
        select fi;

    // Now that we have only what we want,
    // foreach process them into a report.

    names = string.Empty;
    foreach( FamilyInstance instance in listFiOnLevelLINQ )
    {
      names += "\nLevel Name = " + instance.Level.Name
        + "   Instance name = " + instance.Name
        + "   id: " + instance.Id.ToString();
    }

    sw.Stop();

    Util.ShowElapsedTime( sw,
      "Using LINQ to find "
      + listFiOnLevelLINQ.Count<FamilyInstance>().ToString()
      + " family instances on a specific level: "
      + names );

    return Result.Succeeded;
  }
}
```

I ran it on the ArchSample.rvt model file included in Kevin's
[presentation material](http://thebuildingcoder.typepad.com/blog/2010/10/filtered-element-collectors.html) posted yesterday.

The first results showed that the parameter filter was faster than the level filter, even doing the exact job that the element filter was designed to do.
In several different attempts, it consistently took more or less the following numbers of milliseconds to retrieve 52 elements using the three different methods:

- 48 ms - element level filter- 38 ms - parameter filter- 71 ms - LINQ

After rerunning the tests several more times, it does seem that the level filter is faster in general after all.
Here are some more test run results in ArchSample.rvt, as well as in rac\_advanced\_sample\_project.rvt, a sample file included in the basic Revit Architecture installation.
To run in the letter model, you need to change the target level name to something else than "Level 2", e.g. "02 - Floor".

In the following table, A stands for the ArchSample.rvt model file, B for rac\_advanced\_sample\_project.rvt, and L, P and Q for the time for each run in milliseconds for the level filter, parameter filter and LINQ query, respectively:

| Model | L | P | Q |
| --- | --- | --- | --- |
| A | 11 | 24 | 38 |
| A | 11 | 24 | 37 |
| A | 11 | 8 | 62 |
| A | 11 | 23 | 38 |
| A | 7 | 24 | 37 |
| A | 46 | 25 | 68 |
| A | 23 | 28 | 39 |
| A | 11 | 7 | 38 |
| A | 13 | 24 | 11 |
| B | 79 | 101 | 273 |
| B | 56 | 102 | 276 |
| B | 84 | 104 | 173 |

It seems that the level filter is fastest after all, as we would hope.
And certainly the LINQ query is consistently slowest.

You might want to compare the performances of the various filters with your own data sets.

I would also assume that the level filter provides maximum simplicity and probably minimal future maintenance worries.

Here is
[FilterExamples02.zip](zip/FilterExamples02.zip) containing
an updated version of Kevin's code samples and benchmarking code.