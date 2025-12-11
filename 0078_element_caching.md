---
ai_optimized: true
author: Jeremy Tammik
complexity_score: 1.6
content_type: qa
optimization_date: '2025-12-11T11:44:13.327391'
original_url: https://thebuildingcoder.typepad.com/blog/0078_element_caching.html
post_number: 0078
reading_time_minutes: 2
series: elements
slug: element_caching
source_file: 0078_element_caching.htm
tags:
- elements
- filtering
- levels
- revit-api
title: Element Caching
word_count: 365
---

### Element Caching

Here is a question from Julien Duprat of
[Vertice](http://www.vertice.fr):

**Question:**
I would like to know the best practises for data caching.
This topic is mentioned in your tutorial on "Ten Steps for enhancing your Revit add-in".
Specifically, I wonder:

- How to put the elements in a static List<Element> of cached elements?
- How to refresh this list?

**Answer:**
We do not have any official recommendations from the API or development team, so it is more a question of finding out what works best.

The one thing to be aware of is that it is currently not possible to keep track of the exact state of the Revit database, because we are still lacking any element level events. We can request notification when certain events occur at an application or document level, such as application start-up or shutdown and document creation, open, save, and close. There is no notification about adding, deleting or modifying individual elements, though.

Another point is that this issue is much less important in Revit 2009 than it was in 2008. In 2008, we did not yet have the element filtering API. That meant that it was extremely expensive to retrieve a specific set of elements from the database, because the application basically had to iterate over the entire database and filter out the elements it needed itself. To avoid repeating this lengthy iteration several times, it was important to identify all required elements up front and perform the entire iteration for all required elements only once and in one single go. In 2009, with element filtering, this is not longer the case. We now have full support from the Revit API for a highly efficient and optimised selective access, and an application can repeatedly query the database for different sets of specific elements with hardly any overhead. Therefore, the need for caching elements is drastically reduced.

The first part of your question is very easy: to add an element to a generic list, simply call the Add method.

As explained, the second part is more difficult. I would suggest that you update the list every time you need to make use of it.