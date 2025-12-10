---
post_number: "1827"
title: "Headless"
slug: "headless"
author: "Jeremy Tammik"
tags: ['elements', 'family', 'levels', 'revit-api', 'sheets']
source_file: "1827_headless.md"
original_url: "https://thebuildingcoder.typepad.com/blog/1827_headless.html"
---

### Split Pipe
A short note on splitting pipes.
This pretty standard functionality in the Revit MEP user interface can be a bit tricky to find in the API:
\*\*Question:\*\* The UI provides the split command (SL) to split a pipe into two without losing other connected elements.
How can I achieve the same in API?
\*\*Answer:\*\* You can use the `PlumbingUtils` `BreakCurve` method.
This is also available for duct work in `MechanicalUtils` `BreakCurve`.
![Split pipe retaining connections](img/split_pipe_retaining_connections.jpg "Split pipe retaining connections")