---
ai_optimized: true
author: Jeremy Tammik
complexity_score: 1.5
content_type: qa
optimization_date: '2025-12-11T11:44:13.518908'
original_url: https://thebuildingcoder.typepad.com/blog/0191_project_data.html
post_number: 0191
reading_time_minutes: 2
series: general
slug: project_data
source_file: 0191_project_data.htm
tags:
- elements
- family
- filtering
- levels
- parameters
- references
- revit-api
- views
title: Store Project Data
word_count: 480
---

### Store Project Data

We recently discussed
[storing structured data](http://thebuildingcoder.typepad.com/blog/2009/07/store-structured-data.html)
in a Revit file, and received a number of very helpful additional
[comments](http://thebuildingcoder.typepad.com/blog/2009/07/store-structured-data.html#comments)
on the topic.
To store custom application data in a Revit project, one needs to attach a shared parameter to some Revit elements.
As we have seen in the discussion on
[model group shared parameters](http://thebuildingcoder.typepad.com/blog/2009/06/model-group-shared-parameter.html),
almost any element can be used to host the shared parameter.
Here is a common related question:

**Question:**
Where can I store project-wide information in a Revit file, data which is not related to any specific individual element?

**Answer:**
First of all, there is no need to separate the two issues of element-specific versus global data, because the only way to store application data in Revit is to use shared parameters on certain elements.
Luckily, there is one object which occurs exactly once in the entire project and therefore constitutes a suitable repository for global document data, also known as a singleton object: the project information or ProjectInfo element.
It can be retrieved using Revit 2009 filtering by searching for the "Project Information" category with the built-in category OST\_ProjectInformation.
This is discussed in the standard Revit programming introduction presentation and demonstrated by the labs 4-4-1 and 4-4-2 in the
[Revit API introduction labs](zip/rac_labs_2009-07-30.zip).

The ProjectInfo singleton element is not present in a family file, so we have to resort to some other method in that case.

#### Per-document shared parameter in a family document

**Question:**
How can we add a per-document shared parameter to a family document?
The lab 4-4-1 contains code to create and bind shared parameters to the Project Information category, or basically to anything which appears as a singleton in the document.
How can we implement the same for a family document?
It does not contain a Project Information element.

**Answer:**
Looking at a new family file using RvtMgdDbg, I can see one likely candidate, the ProjectUnit object.
I imagine that element also only occurs exactly once.
There are some other elements which may or may not also be unique: the default material, the front view, the basic reference level, the solid fill FillPattern, the Work Plane Grid element, etc.
I suggest you pick an element, any element.
Build a filter to select all such elements.
Include an assertion to verify that exactly one is returned in all cases.
Any element will do, it just has to be easily selectable and unique.

Actually, in a family document, you can probably also use a hidden family parameter instead of attaching the data to any specific element contained within the document.