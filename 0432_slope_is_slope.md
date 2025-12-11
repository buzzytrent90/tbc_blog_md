---
ai_optimized: true
author: Jeremy Tammik
complexity_score: 1.3
content_type: qa
optimization_date: '2025-12-11T11:44:13.936925'
original_url: https://thebuildingcoder.typepad.com/blog/0432_slope_is_slope.html
post_number: '0432'
reading_time_minutes: 1
series: general
slug: slope_is_slope
source_file: 0432_slope_is_slope.htm
tags:
- parameters
- revit-api
title: Slope is Slope, Not Radians
word_count: 259
---

### Slope is Slope, Not Radians

I am still in Thailand, flying home tonight.
Still, I had time to go through some email today, and picked up lots of interesting tidbits.
Let me start with this one:

**Question:** When I use the sample code from the Revit API help file RevitAPI.chm to create a NewFootPrintRoof, the slope is set to 0.5 radians.
This results in a slope of 26.57 degrees for the newly created roof.
This value is incorrect, however, since 0.5 radian equals about 28.65 degrees.

Using the Revit Lookup tool, I can see that the value has indeed been set to 26.57, which is also what we get to see in the UI, and the internal value in radians is 0.5.
What is wrong here, and where does this inconsistency come from?

**Answer:** The value is not in radians.
It is really a 'slope', i.e. the relation between the horizontal and vertical components of the corresponding vector, e.g. 0.5 means half a foot = 6 / 12 up per foot sideways, 0.75 means three quarters of a foot = 9 / 12, etc., and
atan (0.5) ≈ 26.57°

Thanks to Saikat and Scott for raising and clarifying this issue!

In other words, and from another case, if you set a parameter of type slope to a value of 1, the value returned by AsValueString will be 45 degrees (and not 57.217 = 180 / π, for instance).

This definition of slope is mentioned in the description of the FootPrintRoof.SlopeAngle property.