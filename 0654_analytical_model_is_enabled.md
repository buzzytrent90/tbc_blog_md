---
ai_optimized: true
author: Jeremy Tammik
complexity_score: 1.4
content_type: documentation
optimization_date: '2025-12-11T11:44:14.337941'
original_url: https://thebuildingcoder.typepad.com/blog/0654_analytical_model_is_enabled.html
post_number: '0654'
reading_time_minutes: 1
series: structural
slug: analytical_model_is_enabled
source_file: 0654_analytical_model_is_enabled.htm
tags:
- elements
- parameters
- references
- revit-api
- walls
- structural
title: Analytical Model IsEnabled Method and Parameter
word_count: 156
---

### Analytical Model IsEnabled Method and Parameter

I recently mentioned the changes in
[accessing the analytical model](http://thebuildingcoder.typepad.com/blog/2011/08/reference-to-analytical-curve.html) in
the Revit Structure 2012 API.

We just discovered a little issue and an effective workaround related to the AnalyticalModel IsEnabled method, which reports whether the analytical model is currently enabled or disabled.
Right now, it does not do that, but throws an exception for all members except walls.
The Enable and CanDisable methods have similar problems.
This will soon be resolved.
Meanwhile, here is a simple workaround to avoid the issue by accessing the underlying data directly:

The correct "enable" parameter is on the physical element, not its analytical model, and accessible through the built-in parameter STRUCTURAL\_ANALYTICAL\_MODEL.
To enable the analytical model, e.g. for a wall, simply set STRUCTURAL\_ANALYTICAL\_MODEL to true on the wall element, not on the analytical wall element.