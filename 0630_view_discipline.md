---
ai_optimized: true
author: Jeremy Tammik
complexity_score: 2.1
content_type: qa
optimization_date: '2025-12-11T11:44:14.296576'
original_url: https://thebuildingcoder.typepad.com/blog/0630_view_discipline.html
post_number: '0630'
reading_time_minutes: 1
series: views
slug: view_discipline
source_file: 0630_view_discipline.htm
tags:
- csharp
- parameters
- revit-api
- sheets
- transactions
- views
title: View Discipline Enumeration Values
word_count: 281
---

### View Discipline Enumeration Values

In his workaround to
[refresh the referencing sheet parameter display](http://thebuildingcoder.typepad.com/blog/2010/11/refresh-referencing-sheet-parameter-display.html),
Piotr Zurek pointed out that you can change the discipline of the view back and forth using the built-in parameter VIEW\_DISCIPLINE.
That leads to the following query on which values to use for the different disciplines:

**Question:** I can create a new plan view using Document.Create.NewViewPlan.
How can I set the discipline of the resulting view?

I noticed the built-in parameter VIEW\_DISCIPLINE.
It values seem to reflect the view discipline in some way, maybe as an enumeration, but I cannot find the corresponding values in the Revit API documentation.

Can you tell me which enumeration values I can use here?

**Answer:** You are perfectly right, and you can use the built-in parameter VIEW\_DISCIPLINE to determine discipline of a view.
Its storage type is Integer.
The most direct and effective method for you to determine which values to use would be to simply set the different disciplines through the user interface one by one and make a note of the resulting parameter values.
Here is such a list of the values to use and their corresponding disciplines:

1. Architecture- Structure- Mechanical- Electrical- Coordination

As far as I can tell, the Revit API currently does not define an explicit enumeration of these values anywhere, so you can simply define your own or use the integer values directly, for example like this:
```csharp
  Document doc = commandData.Application
    .ActiveUIDocument.Document;

  Transaction trans = new Transaction( doc );
  trans.Start( "Change Discipline" );

  doc.ActiveView.get\_Parameter(
    BuiltInParameter.VIEW\_DISCIPLINE )
      .Set( 4 );

  trans.Commit();
```