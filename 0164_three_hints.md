---
ai_optimized: true
author: Jeremy Tammik
complexity_score: 2.3
content_type: documentation
optimization_date: '2025-12-11T11:44:13.469488'
original_url: https://thebuildingcoder.typepad.com/blog/0164_three_hints.html
post_number: '0164'
reading_time_minutes: 2
series: general
slug: three_hints
source_file: 0164_three_hints.htm
tags:
- csharp
- elements
- parameters
- revit-api
title: Three Coding and Performance Hints
word_count: 448
---

### Three Coding and Performance Hints

We received three interesting and important coding and performance hints from
[Piotr](http://www.piotrzurek.net)
and
[Guy Robinson](http://redbolts.com)
on the post on Linq yesterday.
I find them too important to leave them dozing away in the comment section, so here is their promotion to post status:

1. Use
   [auto-implemented properties](http://msdn.microsoft.com/en-us/library/bb384054.aspx)
   to simplify the helper class code.
2. Use ParametersMap instead of looping over all element parameters.
3. The .NET Framework 3.5 SP1 is significantly improved over .NET 3.5.

As Piotr pointed out, auto-implemented properties are a nice new feature that has been available since .NET 3.0.
It saves some typing and significantly simplifies and shortens the helper class definition.

Guy underlined that Joel's speed increases probably are a result of using proxy objects rather than Linq.
Proxies are useful for data binding as well, though, so they get used more often than not, and Linq is pretty cool.
Also, using .NET3.5SP1 is much better than .NET3.5. It includes a number of important fixes, and WPF is faster with SP1.

Guy also points out that looping over all the element parameters of each element is very costly, so you will get a nice additional speed improvement and a simpler class again for large datasets by using the ParametersMap in the constructor, rather than looping through all parameters.

Jeremy adds that you should please be aware that using the parameter name strings as keys in the parameter lookup has the disadvantage of making the code language dependent. If possible, that should be avoided, and built-in parameter enumeration values or GUIDs should be used instead.
I should think that is faster still, though maybe only marginally.

This is what the helper class definition looks like after applying the first two recommendations:

```csharp
class InstanceData
{
#region Properties
public Element Instance { get; set; }
public String Param1 { get; set; }
public bool Param2 { get; set; }
public int Param3 { get; set; }
#endregion

#region Constructors
public InstanceData( Element instance )
{
  Instance = instance;

  ParameterMap m = Instance.ParametersMap;

  Parameter p = m.get\_Item( "Param1" );
  Param1 = ( p == null ) ? string.Empty : p.AsString();

  p = m.get\_Item( "Param2" );
  Param2 = ( p == null ) ? false : ( 0 != p.AsInteger() );

  p = m.get\_Item( "Param3" );
  Param3 = ( p == null ) ? 0 : p.AsInteger();
}
#endregion
}
```

Many thanks again to Joel for the original article and to Piotr and Guy for the valuable hints!

I still wonder whether using ParametersMap is faster than simply using Parameters.
Another interesting thing to look at some day would be a benchmark comparing various different parameter access methods.
One of these days, when we have lots of time ...