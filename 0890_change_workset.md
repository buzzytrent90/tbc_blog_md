---
ai_optimized: true
author: Jeremy Tammik
complexity_score: 2.4
content_type: qa
optimization_date: '2025-12-11T11:44:14.804833'
original_url: https://thebuildingcoder.typepad.com/blog/0890_change_workset.html
post_number: 0890
reading_time_minutes: 1
series: transactions
slug: change_workset
source_file: 0890_change_workset.htm
tags:
- csharp
- elements
- filtering
- parameters
- references
- revit-api
- selection
- transactions
- views
title: Change Element Workset
word_count: 264
---

### Change Element Workset

We discussed
[reading the workset of an element](http://thebuildingcoder.typepad.com/blog/2011/11/read-only-workset-api.html),
either using the built-in parameter ELEM\_PARTITION\_PARAM or, more easily, the dedicated document GetWorksetId method.

This is actually another example of the possibility to choose your data access method by using either a dedicated property or a parameter, as we recently explored to achieve
[changing the viewport type](http://thebuildingcoder.typepad.com/blog/2013/01/changing-viewport-type.html).

Although the Revit API provides a dedicated property to read an element workset, it does not do so for changing it, which begs the question:

**Question:** The Element.WorksetId property is read-only.

I would like to change an element's workset to other one, though.

Is this possible?

**Answer:** You can change the element's workset via the built-in parameter ELEM\_PARTITION\_PARAM.

Here is a code snippet to show how to retrieve all workset ids and set each one to a given element in turn:
```csharp
  Reference r = uidoc.Selection.PickObject( ObjectType.Element );
  Element e = doc.GetElement( r.ElementId );

  if( e == null )
    return;

  WorksetId wid = e.WorksetId;

  TaskDialog.Show( "Workset id", wid.ToString() );

  Parameter wsparam = e.get\_Parameter(
    BuiltInParameter.ELEM\_PARTITION\_PARAM );

  if( wsparam == null )
    return;

  // Find all user worksets

  FilteredWorksetCollector worksets
    = new FilteredWorksetCollector( doc )
      .OfKind( WorksetKind.UserWorkset );

  using( Transaction tx = new Transaction( doc ) )
  {
    tx.Start( "Change workset id" );

    foreach( Workset ws in worksets )
    {
      wsparam.Set( ws.Id.IntegerValue );
    }

    tx.Commit();
  }
  wid = e.WorksetId;

  TaskDialog.Show( "worksetid", wid.ToString() );
```

Many thanks to Phil Xia for this hint!