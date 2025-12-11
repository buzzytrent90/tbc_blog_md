---
ai_optimized: true
author: Jeremy Tammik
complexity_score: 3.3
content_type: qa
optimization_date: '2025-12-11T11:44:13.857545'
original_url: https://thebuildingcoder.typepad.com/blog/0384_elem_name_param_filter.html
post_number: 0384
reading_time_minutes: 3
series: filtering
slug: elem_name_param_filter
source_file: 0384_elem_name_param_filter.htm
tags:
- csharp
- elements
- filtering
- levels
- parameters
- revit-api
title: Element Name Parameter Filter Correction
word_count: 532
---

### Element Name Parameter Filter Correction

The AEC DevCamp has kicked off here in Boston and it is great to be here with all the fascinating presentations and exciting meetings with colleagues and developers.
Somehow, I am also able to keep up with other issues in parallel, and one of them has been a lively discussion with Kevin Vandecar which prompted the presentation of lots of new
[parameter filter samples](http://thebuildingcoder.typepad.com/blog/2010/04/collector-benchmark.html) this morning.

Kevin pointed out that the method I presented in the
[filtered element collector benchmarking](http://thebuildingcoder.typepad.com/blog/2010/04/collector-benchmark.html) to
retrieve a specific level from the Revit model by name using a parameter filter did not work for him.
The reason is that the element name on the level is not stored in the ELEM\_NAME\_PARAM built-in parameter, as I had assumed but apparently not tested, but in BuiltInParameter.DATUM\_TEXT.

I updated the code to be able to specify any built-in parameter by defining the following generic GetFirstElementOfTypeWithBipString method.
It uses a parameter filter to return the first element of the given type and with the specified string-valued built-in parameter matching the given name:
```csharp
Element GetFirstElementOfTypeWithBipString(
  Type type,
  BuiltInParameter bip,
  string name )
{
  FilteredElementCollector a
    = GetElementsOfType( type );

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

This matches the method GetFirstNamedElementOfTypeUsingParameterFilter that I presented in the original version, except that the built-in parameter can be passed in as an argument instead of being hardwired to use the erroneous ELEM\_NAME\_PARAM.

In the benchmark mainline, I now call it like this:
```csharp
  level = null;

  using( JtTimer pt = new JtTimer(
    "Parameter filter" ) )
  {
    //level = GetFirstElementOfTypeWithBipString(
    //  t, BuiltInParameter.ELEM\_NAME\_PARAM, name );

    level = GetFirstElementOfTypeWithBipString(
      t, BuiltInParameter.DATUM\_TEXT, name );
  }

  Debug.Assert( null != level,
    "expected to find a valid level" );
```

Note that I added an assertion to ensure that a valid level really was retrieved, now that Kevin pointed out to me that this was previously not the case.

The results of running this are analogous to the previously reported ones, i.e. using the parameter filter is still significantly faster than all other options using some post-processing of the collector results to search for the specific named level, less than 6 seconds being less than half the time required by the post-processing approaches, which all require over 13 seconds:
```csharp
---------------------------------------------------------------
Retrieve specific named level:
Percentage Seconds Calls Process
---------------------------------------------------------------
0.00% 0.00 1000 Empty method \*
0.19% 0.11 1000 Collector with no name check \*
9.19% 5.46 1000 Parameter filter
22.53% 13.37 1000 Explicit
22.57% 13.40 1000 Anonymous named
22.73% 13.49 1000 Anonymous
22.73% 13.49 1000 Linq
100.00% 59.35 1 TOTAL TIME
---------------------------------------------------------------
```

Here is
[version 2011.0.70.2](zip/bc_11_70_2.zip)
of The Building Coder sample source code and Visual Studio solution including the updated filtered element collector benchmark command.

Many thanks to Kevin Vandecar for pointing this out and prompting me to update the original article!