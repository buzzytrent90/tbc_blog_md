---
ai_optimized: true
author: Jeremy Tammik
complexity_score: 1.3
content_type: qa
optimization_date: '2025-12-11T11:44:13.596768'
original_url: https://thebuildingcoder.typepad.com/blog/0234_system_category.html
post_number: '0234'
reading_time_minutes: 1
series: elements
slug: system_category
source_file: 0234_system_category.htm
tags:
- elements
- family
- levels
- parameters
- revit-api
title: System Versus User Family Category
word_count: 201
---

### System Versus User Family Category

Here is a simple question and suggestion on classifying categories into built-in system versus user defined.

**Question:** Given an arbitrary element, how can I determine whether it belongs to a system family as opposed to a user-defined family?
I thought I might look at its category, but how can I see whether that is system-defined?
For example, I see that a duct has a category id of -2008000.
Can I depend on the fact that this category id value is negative?
Or is there any other way to determine this?

**Answer:** There is currently no API method to distinguish system categories from user defined ones short of creating your own hardcoded list.
All built-in categories have negative values, but this includes system family types, family types, and subcategories of each, so that will not help you resolve this issue.

How could such a list be created?

Unfortunately, that would have to be done manually.
This Family Category and Parameters dialogue lists all the family based categories:

![Family categories](img/categories.jpg)

If the category is a top level category and not in this list, then it is probably a system family category.