---
ai_optimized: true
author: Jeremy Tammik
complexity_score: 1.6
content_type: qa
optimization_date: '2025-12-11T11:44:13.281939'
original_url: https://thebuildingcoder.typepad.com/blog/0046_format_unit.html
post_number: '0046'
reading_time_minutes: 2
series: general
slug: format_unit
source_file: 0046_format_unit.htm
tags:
- elements
- parameters
- revit-api
title: Formatting Unit Strings
word_count: 404
---

### Formatting Unit Strings

What methods can I use to create a properly formatted unit string according to the project units?

If the value you are formatting resides in a Revit parameter, the answer is simple and a complete solution immediately provided by the Revit API. On one hand, the Parameter class provides the following public methods to read and write its parameter value:

- AsDouble: Provides access to the double precision number within the parameter.
- AsElementId: Provides access to the element id stored within the parameter.
- AsInteger: Provides access to the integer number within the parameter.
- AsString: Provides access to the string contents of the parameter.
- Set: Defines a new value for the parameter.

On the other hand, you can ask the parameter to return its current value as a preformatted string using the current unit settings:

- AsValueString: Get the parameter value as a string with unit.

It also provides access to set the value making use of such a formatted string with units:

- SetValueString: Set the parameter value according to the input string.

If the value you wish to format does not reside in a parameter, you do not have access to these methods.
In that case, you might be able to temporarily hijack some existing parameter on an existing element and use that for your formatting task.

Unfortunately, you cannot simply create an own parameter for temporary use as a formatting helper, because the constructor is for internal use only. This is due to the fact that a parameter cannot live on its own; it is linked to a definition and needs to be hooked up properly in the Revit database system to work.

If you wish to avoid the hijacking, you could create an own shared parameter for an element category that occurs only once in the project and use that. How to do this is discussed the post on
[adding a shared parameter to a DWG file](http://thebuildingcoder.typepad.com/blog/2008/11/adding-a-shared-parameter-to-a-dwg-file.html),
and the
[Revit API introduction labs](http://thebuildingcoder.typepad.com/blog/files/rac_labs_20081117.zip)
provide samples for creating and analysing parameters in Revit.

Finally, for full control over the formatting details, you can query Revit for the current unit settings using the ProjectUnit class and use that information to do the formatting yourself as you see fit.
Querying the unit settings is demonstrated by the ProjectUnit SDK sample.