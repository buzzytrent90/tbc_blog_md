---
ai_optimized: true
author: Jeremy Tammik
complexity_score: 2.8
content_type: documentation
optimization_date: '2025-12-11T11:44:13.647940'
original_url: https://thebuildingcoder.typepad.com/blog/0267_filter_units.html
post_number: '0267'
reading_time_minutes: 2
series: filtering
slug: filter_units
source_file: 0267_filter_units.htm
tags:
- elements
- filtering
- parameters
- revit-api
- rooms
title: Parameter Filter Units
word_count: 434
---

### Parameter Filter Units

The Western European DevDays conference tour has begun, and I will be presenting and travelling all week, leaving very little time to spend blogging or responding to comments.
I visited relatives in London the preceding weekend, and took some time to write and post this before the real crunch began.

We have several old posts dealing with various aspects of unit handling in the Revit API:

- [Units](http://thebuildingcoder.typepad.com/blog/2008/09/units.html).
- [Formatting unit strings](http://thebuildingcoder.typepad.com/blog/2008/11/formatting-units-strings.html).
- [Unit types and format options](http://thebuildingcoder.typepad.com/blog/2009/01/unit-types-and-format-options.html).
- [Unit conversion](http://thebuildingcoder.typepad.com/blog/2009/08/unit-conversion.html).
- [Unit Suffix and the ProjectUnit SDK sample](http://thebuildingcoder.typepad.com/blog/2009/10/unit-suffix-and-the-projectunit-sdk-sample.html).
- [DisplayUnitType and UnitSymbolType](http://thebuildingcoder.typepad.com/blog/2009/10/dut-versus-ust.html).

Here is a new related information nugget, a surprising little quirk when defining Revit API filters that just came to my attention last week.
In brief, a Revit API parameter filter specifying a length or area will be evaluated using the current user interface units, and not the database units, as one would expect.
This is an inconsistency that we expect to change in future releases.
This means that to make proper use of filters for selecting elements depending on some unit-based parameter, an add-in using the 2010 API will have to convert the target value to the current user interface unit.
This is unexpected.
I would expect that when working purely within the API, all units should remain in the internal database units, with no exceptions, not for filters nor for anything else.
In future, this conversion will hopefully be removed, so that add-ins can work consistently and exclusively in Revit database units.

Specifically, the Filter.NewParameterFilter method taking the three arguments BuiltInParameter, CriteriaFilterType, and double returns the selected elements based on the units displayed in the user interface instead of the internal database units.

**Example:** This can be reproduced using the ElementsFilter SDK sample.
I tested it by running the ElementsFilter in a minimal model with two rooms:

```
ROOM_AREA  Area  double   6.840 m2   73.63 sqf
ROOM_AREA  Area  double  14.440 m2  155.43 sqf
```

Specifying a filter to select all rooms with a value of ROOM\_AREA exceeding 14 returns only the second, larger room, i.e. it is evaluating the parameter in the user interface area unit 'square meters' rather than the database area unit 'square feet'.