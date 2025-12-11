---
ai_optimized: true
author: Jeremy Tammik
complexity_score: 1.5
content_type: qa
optimization_date: '2025-12-11T11:44:16.827970'
original_url: https://thebuildingcoder.typepad.com/blog/1827_headless.html
post_number: '1827'
reading_time_minutes: 1
series: general
slug: headless
source_file: 1827_headless.md
tags:
- elements
- family
- levels
- revit-api
- sheets
title: Headless
word_count: 84
---

### Split Pipe
A short note on splitting pipes.
This pretty standard functionality in the Revit MEP user interface can be a bit tricky to find in the API:
\*\*Question:\*\* The UI provides the split command (SL) to split a pipe into two without losing other connected elements.
How can I achieve the same in API?
\*\*Answer:\*\* You can use the `PlumbingUtils` `BreakCurve` method.
This is also available for duct work in `MechanicalUtils` `BreakCurve`.
![Split pipe retaining connections](img/split_pipe_retaining_connections.jpg "Split pipe retaining connections")