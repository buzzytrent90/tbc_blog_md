---
ai_optimized: true
author: Jeremy Tammik
complexity_score: 3.6
content_type: qa
optimization_date: '2025-12-11T11:44:13.239591'
original_url: https://thebuildingcoder.typepad.com/blog/0019_more_filter_performance.html
post_number: 0019
reading_time_minutes: 3
series: filtering
slug: more_filter_performance
source_file: 0019_more_filter_performance.htm
tags:
- doors
- elements
- filtering
- parameters
- revit-api
- windows
title: More Filter Performance
word_count: 529
---

### More Filter Performance

This post is a response to Guy Robinson's contribution on
[Filter Performance](http://thebuildingcoder.typepad.com/blog/2008/10/filter-performa.html),
and is also by a reader, Ralf Huvendiek. Ralf only recently started working on the Revit API, but brings such a wealth of prior programming experience and enthusiasm that I am very grateful for his invaluable input. He says:

Guy raises an interesting question regarding the performance of the Revit filters. I think he comes to the right conclusion, but not the best solution or explanation.

First we need to agree about a few things. Let us assume the following:

1. Our drawing contains x elements.
2. x is very high.
3. Our drawing contains d doors and w windows.
4. d+w is a lot smaller than x.
5. The Revit TypeFilter has to use some kind of RTTI and is thus much slower than a plain CategoryFilter.

Now let us return to the original filter, which was

```
  (TypeFilter && (CategoryFilter || CategoryFilter))
```

To evaluate that filter, Revit has to invoke the TypeFilter x times, because it is the first filter in our Boolean expression.

If we look at the points 'b' and 'e' above, we see that this is not very desirable. With a look at our Boolean expression, and the fact that '&&' is commutative, we can reorder the filter like this:

```
   ((CategoryFilter || CategoryFilter) && TypeFilter)
```

Now our cheap CategoryFilter will be called ~x times and the expensive TypeFilter will only be called (w+d) times.

With this knowledge, we can reorder Jeremy's original code from:

```
  Filter f5 = cf.NewLogicAndFilter( f1, f4 );
```

To:

```
  Filter f5 = cf.NewLogicAndFilter( f4, f1 );
```

The CmdRelationshipInverter modified as described is just as fast as the Guy's solution. This is not surprising, because his code does the same thing in a different way: prefilter everything with f4, and then use the expensive RTTI to filter out the rest afterwards.
But I think Jeremy's solution from with a modified filter f5 is much cleaner than filtering using an anonymous method.

What can we conclude from this:

1. If we're writing a performance critical filter, we should be aware how to organize it best: put the expensive filters as far to the right as possible.
2. We can't blame the Revit API for being slow here. It's just the filter that is not optimal.

Ralf measured the times running the various variants of the command that we discussed so far on one of the standard Revit sample files, using the following setup parameters:

- Revit version: Revit Architecture Build 20080508\_1900 (German)
- Project: m\_Conference.rvt from Revit Samples
- Number of elements in model: 14153
- Number of windows and doors: 50
- Machine used for testing: Windows XP SP3 German / CPU: Core 2 Duo E6850 / 2GB RAM
- Compiler: Visual Studio 2005 Team Edition (SP1)

Here are the resulting times:

| Test run | Original filter | Guy's code | Optimised filter |
| --- | --- | --- | --- |
| 1 | 170 | 58 | 58 |
| 2 | 170 | 57 | 58 |
| 3 | 171 | 57 | 58 |
| 4 | 191 | 57 | 58 |
| 5 | 171 | 57 | 58 |
| 6 | 171 | 57 | 58 |
| 7 | 170 | 57 | 58 |
| 8 | 170 | 57 | 58 |
| 9 | 197 | 57 | 58 |
| 10 | 172 | 57 | 58 |