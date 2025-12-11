---
ai_optimized: true
author: Jeremy Tammik
complexity_score: 14.3
content_type: code_example
optimization_date: '2025-12-11T11:44:13.762114'
original_url: https://thebuildingcoder.typepad.com/blog/0333_collector_benchmark.html
post_number: '0333'
reading_time_minutes: 14
series: filtering
slug: collector_benchmark
source_file: 0333_collector_benchmark.htm
tags:
- csharp
- elements
- family
- filtering
- levels
- parameters
- python
- references
- revit-api
- views
- walls
- windows
title: Collector Benchmark
word_count: 2788
---

### Collector Benchmark

After all the preparation creating a
[profiling tool](http://thebuildingcoder.typepad.com/blog/2010/03/performance-profiling.html),
let us now put it to use on the Revit 2011 collectors, which is one of the areas that has been most heavily reworked in this release and at the same time affects more applications than any other, in fact just about every single one.
This has led to a rather huge post for today, but I really wanted to share these important results with just as soon as possible.

First of all, what is this all about?
Here is a quick first introduction to the Revit 2011 filtering and pointers to further reading:

#### New Element Iteration Interfaces

The element iteration part of the Revit API has been completely redesigned.
This will affect virtually all existing add-ins, since they all need to access elements to query or modify their properties.
The new API is much more aligned and provides better access to internal optimised functionality within Revit, providing a significant speed increase and smaller memory footprint.
Here are some of its advantages:

- Iterate and filter elements from a document, or only elements from an arbitrary list of element ids, or elements visible in a view (replacing View.Elements).- Clearly identify so-called quick filters which are designed for best performance and do not expand the element in memory when evaluating whether it passes the filter.- Use chained shortcuts which automatically apply commonly used filters:
      ```csharp
      collector
        .OfClass( typeof( Wall ) )
        .ContainedInDesignOption( myDesignOptionId );
      ```- Logically group more than two filters.- Match derived types automatically when using the type filter and type filter shortcut.- Iterate elements from all design options or from any specific design option.- Process the collector results using foreach statements and LINQ queries:
              ```csharp
              IEnumerable<FamilySymbol> symbols =
                from FamilySymbol fs in collector
                where fs.Family.Name == familyName
                select fs;
              ```

The element filtering is performed by FilteredElementCollector instances which are instantiated for a given document, view or list of elements to work with.
Numerous filtering options can be applied, and a collection of elements matching the specified criteria is returned.
This collection supports further filtering using .NET functionality such as foreach and
[LINQ](http://thebuildingcoder.typepad.com/blog/2009/07/language-integrated-query-linq.html).

Here is a small VSTA sample that looks for a specific family and returns all its symbols:
```csharp
public void MyTest()
{
  string familyName = "Single-Flush";
  Document doc = this.ActiveUIDocument.Document;

  // get the family we want

  FilteredElementCollector fec
    = new FilteredElementCollector( doc );

  fec.OfClass( typeof( Family ) );

  IEnumerable<Family> families =
    from Family f in fec
    where f.Name == familyName
    select f;

  // get the symbols of that family

  FamilySymbolFilter fsf
    = new FamilySymbolFilter(
        families.First<Family>().Id );

  fec = new FilteredElementCollector( doc );
  fec.WherePasses( fsf );

  // list them

  StringBuilder str = new StringBuilder();
  foreach( FamilySymbol fs in fec )
    str.Append( fs.Name + "\n" );

  System.Windows.Forms.MessageBox.Show(
    str.ToString(),
    "FamilySymbols of " + familyName );
}
```

Further information on this topic is provided in the Revit 2011 SDK, in the API Reference RevitAPI.chm, in the sections on Whats New and Element Iteration API, and also in the Developer Guide.

So with the introduction out of the way, let's get on with our research and analysis to find out how to make optimal use of it.

#### Benchmarking Element Iteration Collectors

I implemented a new command CmdCollectorPerformance in The Building Coder sample application, the first pure Revit 2011 one.
It performs the following three steps:

1. Create a largish number of levels for us to play making use of the CreateLevel helper method, which simply creates a new level at a given elevation.- BenchmarkAllLevels –
     Benchmark various approaches to using
     filtered collectors to retrieve
     all levels in the model,
     and measure the time required to
     create IList and List collections from them.- BenchmarkSpecificLevel –
       Benchmark using a parameter filter versus
       various kinds of post processing of the
       results returned by the filtered element
       collector to find the level specified by
       iLevel.

#### Setting up the Test Model

We simply use a number of levels as a test set.
Here is the code used to create an individual level:
```csharp
Level CreateLevel( int elevation )
{
  Level level = \_doc.Create.NewLevel( elevation );
  level.Name = "Level " + elevation.ToString();
  return level;
}
```

This loop is used to drive it to create a large number of levels:
```csharp
  int maxLevel = 1000;
  for( int i = 3; i < maxLevel; ++i )
  {
    CreateLevel( i );
  }
```

If this is run in a default new Revit Architecture model, we will end up with the two pre-defined levels 1 and 2 plus newly generated ones numbered up to 999 for a total of 999 levels in all.

#### Test Methods to Retrieve all Levels

In the following benchmarking tests, I always included a test using an empty method that does nothing at all, just to ensure that the minimal overhead of calling the method and running the test itself is negligible compared to the functionality that I am actually benchmarking.
That is the reason for implementing these pretty trivial test methods:
```csharp
Element EmptyMethod( Type type )
{
  return null;
}

Element EmptyMethod( Type type, string name )
{
  return null;
}
```

Here are the basic minimal collector methods which we need to get any access at all to the Revit database elements:

- GetNonElementTypeElements – Return all non ElementType elements.- GetElementsOfType – Return a collector of all elements of the given type.- GetFirstElementOfType – Return the first element of the given type without any further filtering.

The first is used to return all elements which are not derived from ElementType.
This is used to compare the time required to check the type of elements manually against the time it takes the dedicated Revit filtering functionality used by GetElementsOfType to do the same thing.
```csharp
FilteredElementCollector GetNonElementTypeElements()
{
  return new FilteredElementCollector( \_doc )
    .WhereElementIsNotElementType();
}

FilteredElementCollector GetElementsOfType(
  Type type )
{
  return new FilteredElementCollector( \_doc )
    .OfClass( type );
}

Element GetFirstElementOfType(
  Type type )
{
  return new FilteredElementCollector( \_doc )
    .OfClass( type )
    .FirstElement();
}
```

Here are two methods that use explicit coding and a LINQ query to filter for a specific element type:

- GetElementsOfTypeUsingExplicitCode –
  Return a list of all elements matching
  the given type using explicit code to test
  the element type.- GetElementsOfTypeUsingLinq –
    Return a list of all elements matching
    the given type using a LINQ query to test
    the element type.

```python
List<Element> GetElementsOfTypeUsingExplicitCode(
  Type type )
{
  FilteredElementCollector a
    = GetNonElementTypeElements();

  List<Element> b = new List<Element>();
  foreach( Element e in a )
  {
    if( e.GetType().Equals( type ) )
    {
      b.Add( e );
    }
  }
  return b;
}

IEnumerable<Element> GetElementsOfTypeUsingLinq(
  Type type )
{
  FilteredElementCollector a
    = GetNonElementTypeElements();

  IEnumerable<Element> b =
    from e in a
    where e.GetType().Equals( type )
    select e;

  return b;
}
```

The performance of these is then compared with GetElementsOfType using the OfClass method to achieve the same thing.

#### Benchmarking Retrieval of all Levels

Here is the mainline code that we use to drive the benchmarking of the time required to retrieve all levels in different ways:
```csharp
  int nLevels = GetElementsOfType( typeof( Level ) )
    .ToElements().Count;

  int nRuns = 1000;

  JtTimer totalTimer = new JtTimer(
    "TOTAL TIME" );

  using( totalTimer )
  {
    for( int i = 0; i < nRuns; ++i )
    {
      BenchmarkAllLevels( nLevels );
    }
  }

  totalTimer.Report( "Retrieve all levels:" );
```

The interesting question now is what exactly happens in the BenchmarkAllLevels method, and what the reported results look like.

BenchmarkAllLevels takes one argument, the count of levels, which is simply used to verify that the results from some of the test methods make sense.
It benchmarks several different approaches to using filtered collectors to retrieve all levels in the model and measure the time required to create IList and List collections from them:
```csharp
void BenchmarkAllLevels( int nLevels )
{
  Type t = typeof( Level );
  int n;

  using( JtTimer pt = new JtTimer(
    "Empty method \*" ) )
  {
    EmptyMethod( t );
  }

  using( JtTimer pt = new JtTimer(
    "NotElementType \*" ) )
  {
    FilteredElementCollector a
      = GetNonElementTypeElements();
  }

  using( JtTimer pt = new JtTimer(
    "NotElementType as IList \*" ) )
  {
    IList<Element> a
      = GetNonElementTypeElements().ToElements();
    n = a.Count;
  }
  Debug.Assert( nLevels <= n,
    "expected to retrieve all non-element-type elements" );

  using( JtTimer pt = new JtTimer(
    "NotElementType as List \*" ) )
  {
    List<Element> a = new List<Element>(
      GetNonElementTypeElements() );

    n = a.Count;
  }
  Debug.Assert( nLevels <= n,
    "expected to retrieve all non-element-type elements" );

  using( JtTimer pt = new JtTimer( "Explicit" ) )
  {
    List<Element> a
      = GetElementsOfTypeUsingExplicitCode( t );

    n = a.Count;
  }
  Debug.Assert( nLevels == n,
    "expected to retrieve all levels" );

  using( JtTimer pt = new JtTimer( "Linq" ) )
  {
    IEnumerable<Element> a =
      GetElementsOfTypeUsingLinq( t );

    n = a.Count<Element>();
  }
  Debug.Assert( nLevels == n,
    "expected to retrieve all levels" );

  using( JtTimer pt = new JtTimer(
    "Linq as List" ) )
  {
    List<Element> a = new List<Element>(
      GetElementsOfTypeUsingLinq( t ) );

    n = a.Count;
  }
  Debug.Assert( nLevels == n,
    "expected to retrieve all levels" );

  using( JtTimer pt = new JtTimer( "Collector" ) )
  {
    FilteredElementCollector a
      = GetElementsOfType( t );
  }

  using( JtTimer pt = new JtTimer(
    "Collector as IList" ) )
  {
    IList<Element> a
      = GetElementsOfType( t ).ToElements();

    n = a.Count;
  }
  Debug.Assert( nLevels == n,
    "expected to retrieve all levels" );

  using( JtTimer pt = new JtTimer(
    "Collector as List" ) )
  {
    List<Element> a = new List<Element>(
      GetElementsOfType( t ) );

    n = a.Count;
  }
  Debug.Assert( nLevels == n,
    "expected to retrieve all levels" );
}
```

Here are the results of running this, i.e. 1000 repetitions of retrieving all the 999 levels in several different ways:

```
------------------------------------------------------------------------
Retrieve all levels:
 Percentage   Seconds   Calls   Process
------------------------------------------------------------------------
      0.00%      0.00    1000   Empty method *
      0.01%      0.01    1000   NotElementType *
      0.02%      0.03    1000   Collector
      3.85%      7.95    1000   Collector as IList
      6.42%     13.26    1000   Collector as List
      9.24%     19.07    1000   NotElementType as IList *
     20.03%     41.37    1000   Explicit
     20.07%     41.46    1000   Linq
     20.12%     41.54    1000   NotElementType as List *
     20.21%     41.73    1000   Linq as List
    100.00%    206.51       1   TOTAL TIME
------------------------------------------------------------------------
```

The entries marked with an asterisk \* do not perform the full operation that the others complete.
They have been added to measure specific minimum overhead delays.
For example, we call an empty method to determine the overhead of the function call itself and to prove that this is completely negligible in comparison to the overall time.

There are lots of things to point out here:

- The calls to the empty method really are negligible.- The three calls to NotElementType return over 3000 elements, i.e. more than just the levels, and do not have any overhead for filtering out any specific type.
    They also give us an idea of the overhead required to convert the collector results to a generic IList and List instance.
    As one might expect, creating a List causes a significantly larger overhead than an IList.
    An IList is returned directly by the collector ToElements method, whereas a List requires an explicit call to a copy constructor.- The pure collector call using OfClass is fastest and easiest. Again, conversion to IList or List is much more expensive than the filtering operation itself.- Using explicit code or LINQ to filter is a thousand times as expensive as using the built-in collector filtering.- There is hardly any performance difference between using LINQ or explicit code.

I find these results very interesting and illuminating.

The really good news is that the Revit filtering is extremely efficient, and anything you add to it, such as converting the results to generic List, will cost much more time than the filtering operation itself.

I want to discuss the results of selecting an individual element as well, but I will have to postpone that until after Easter, because I am really running out of time here.

Or no, I will just cut it really short.

#### Benchmarking Retrieval of a Specific Named Level

The following BenchmarkSpecificLevel method lists the various tests that I implemented and compared to retrieve a specific named element.
It benchmarks the use of a parameter filter versus
various kinds of post processing of the
results returned by the filtered element
collector to find the level specified by the iLevel argument:
```csharp
void BenchmarkSpecificLevel( int iLevel )
{
  Type t = typeof( Level );
  string name = "Level " + iLevel.ToString();

  using( JtTimer pt = new JtTimer(
    "Empty method \*" ) )
  {
    Element level
      = EmptyMethod(
        t, name );
  }

  using( JtTimer pt = new JtTimer(
    "Collector with no name check \*" ) )
  {
    Element level = GetFirstElementOfType( t );
  }

  using( JtTimer pt = new JtTimer(
    "Parameter filter" ) )
  {
    Element level
      = GetFirstNamedElementOfTypeUsingParameterFilter(
        t, name );
  }

  using( JtTimer pt = new JtTimer( "Explicit" ) )
  {
    Element level
      = GetFirstNamedElementOfTypeUsingExplicitCode(
        t, name );
  }
  using( JtTimer pt = new JtTimer( "Linq" ) )
  {
    Element level
      = GetFirstNamedElementOfTypeUsingLinq(
        t, name );
  }
  using( JtTimer pt = new JtTimer(
    "Anonymous named" ) )
  {
    Element level
      = GetFirstNamedElementOfTypeUsingAnonymousButNamedMethod(
        t, name );
  }
  using( JtTimer pt = new JtTimer( "Anonymous" ) )
  {
    Element level
      = GetFirstNamedElementOfTypeUsingAnonymousMethod(
        t, name );
  }
}
```

Ah yes, we still have not presented all the test methods that we are comparing yet.
They are:

- GetFirstNamedElementOfTypeUsingExplicitCode – Return the first element of the given type and name using explicit code.- GetFirstNamedElementOfTypeUsingLinq – Return the first element of the given type and name using LINQ.- GetFirstNamedElementOfTypeUsingAnonymousButNamedMethod – Return the first element of the given type and name using an anonymous method to define a named method.- GetFirstNamedElementOfTypeUsingAnonymousMethod – Return the first element of the given type and name using an anonymous method.

```python
Element GetFirstNamedElementOfTypeUsingExplicitCode(
  Type type,
  string name )
{
  FilteredElementCollector a
    = GetElementsOfType( type );
  //
  // explicit iteration and manual checking of a property:
  //
  Element ret = null;
  foreach( Element e in a )
  {
    if( e.Name.Equals( name ) )
    {
      ret = e;
      break;
    }
  }
  return ret;
}

Element GetFirstNamedElementOfTypeUsingLinq(
  Type type,
  string name )
{
  FilteredElementCollector a
    = GetElementsOfType( type );
  //
  // using LINQ:
  //
  IEnumerable<Element> elementsByName =
    from e in a
    where e.Name.Equals( name )
    select e;

  return elementsByName.First<Element>();
}

Element GetFirstNamedElementOfTypeUsingAnonymousButNamedMethod(
  Type type,
  string name )
{
  FilteredElementCollector a
    = GetElementsOfType( type );
  //
  // using an anonymous method to define a named method:
  //
  Func<Element, bool> nameEquals = e => e.Name.Equals( name );
  return a.First<Element>( nameEquals );
}

Element GetFirstNamedElementOfTypeUsingAnonymousMethod(
  Type type,
  string name )
{
  FilteredElementCollector a
    = GetElementsOfType( type );
  //
  // using an anonymous method:
  //
  return a.First<Element>(
    e => e.Name.Equals( name ) );
}
```

The most important method of all, as we shall see, is the one making use of the Revit filtering API to search for the named level.
Setting up a parameter filter is a little bit convoluted, since you need to make use of a number of different helper classes to specify separately the provider, evaluator, rule and filter, but have a look further down to convince yourself that it is worthwhile.
This method returns the first element of the given type and name using a parameter filter:
```csharp
Element GetFirstNamedElementOfTypeUsingParameterFilter(
  Type type,
  string name )
{
  FilteredElementCollector a
    = GetElementsOfType( type );

  BuiltInParameter bip
    = BuiltInParameter.ELEM\_NAME\_PARAM;

  ParameterValueProvider provider
    = new ParameterValueProvider(
      new ElementId( bip ) );

  FilterStringRuleEvaluator evaluator
    = new FilterStringEquals();

  FilterRule rule = new FilterStringRule(
    provider, evaluator, name, true );

  ElementParameterFilter filter
    = new ElementParameterFilter( rule );

  return a.WherePasses( filter ).FirstElement();
}
```

I drive this by selecting some target level at random and searching for it by name, and repeating that 1000 times over, as follows:
```csharp
  nRuns = 1000;
  Random rand = new Random();
  totalTimer.Restart( "TOTAL TIME" );

  using( totalTimer )
  {
    for( int i = 0; i < nRuns; ++i )
    {
      int iLevel = rand.Next( 1, maxLevel );
      BenchmarkSpecificLevel( iLevel );
    }
  }

  totalTimer.Report(
    "Retrieve specific named level:" );
```

Here are the results of running this test, i.e. 1000 repetitions of retrieving one specific level from the 999 levels in the model by searching for it by name:

```
---------------------------------------------------------------
Retrieve specific named level:
 Percentage   Seconds   Calls   Process
---------------------------------------------------------------
      0.00%      0.00    1000   Empty method *
      0.16%      0.10    1000   Collector with no name check *
     12.65%      7.96    1000   Parameter filter
     21.60%     13.60    1000   Anonymous named
     21.75%     13.70    1000   Explicit
     21.85%     13.76    1000   Anonymous
     21.88%     13.77    1000   Linq
    100.00%     62.96       1   TOTAL TIME
---------------------------------------------------------------
```

Again, the entries marked with an asterisk \* do not do the full job and are just included for baseline comparison purposes.
The empty method entry does nothing at all.
The collector with no name check collects all levels and simply returns the first one without checking its name.

The important conclusions that I draw from this are:

- Once again, making use of the Revit filtering API by using the parameter filter is by far the most efficient approach.
  The parameter filter is a slow filter, not a quick one, but it is still a lot faster than resorting to any other means.- There is virtually no performance difference at all between explicit coding, LINQ, and using generic algorithms with anonymous methods.

Once again, very illuminating, I would say.

In quintessence, you should do everything that you possibly can using the Revit filters, and avoid all post-processing and manual if at all possible.

Here is
[version 2011.0.0.63](zip/bc_11_63.zip)
of the complete Building Coder source code and Visual Studio solution including the new CmdCollectorPerformance external command and the JtTimer profiling classes.

I hope this keeps you happily informed and occupied over Easter and wish you good luck searching for eggs.

Since I am giving the Revit API training class in Warsaw next week, the posts may become sparse for a while.

**Correction:** Please note that the method GetFirstNamedElementOfTypeUsingParameterFilter does not work properly using the built-in parameter ELEM\_NAME\_PARAM, because it should be looking at DATUM\_TEXT instead.
For more details, please refer to the subsequent
[element name parameter filter correction](http://thebuildingcoder.typepad.com/blog/2010/06/element-name-parameter-filter-correction.html).