---
ai_optimized: true
author: Jeremy Tammik
complexity_score: 1.5
content_type: qa
optimization_date: '2025-12-11T11:44:13.305657'
original_url: https://thebuildingcoder.typepad.com/blog/0065_creating_a_viewport.html
post_number: '0065'
reading_time_minutes: 1
series: views
slug: creating_a_viewport
source_file: 0065_creating_a_viewport.htm
tags:
- references
- revit-api
- sheets
- views
title: Creating a Viewport
word_count: 172
---

### Creating a Viewport

Here is another short and simple question and answer session from a dialogue between Greg Wesner and Harry Mattison, both of Autodesk.

**Question:** Im wondering if there is a way to create a new viewport on a sheet. I would like to automate the process of updating standard details.

I can see where I can create a sheet and I can create a view, but I do not see the ability to create a viewport on a sheet. I need to be able to create a sheet, and then create a viewport on the sheet that references a drafting view, just like you would do in the user interface by dragging a drafting view onto the sheet. Am I missing something or has this just not made it into the API yet?

**Answer:** You are probably looking for the ViewSheet.AddView method. An example of using this method is provided by the Revit SDK sample AllViews, which generates a new sheet including a certain list of interactively selected views.