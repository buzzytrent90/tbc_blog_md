---
ai_optimized: true
author: Jeremy Tammik
complexity_score: 3.1
content_type: qa
optimization_date: '2025-12-11T11:44:14.142515'
original_url: https://thebuildingcoder.typepad.com/blog/0544_element_id_equality.html
post_number: '0544'
reading_time_minutes: 1
series: elements
slug: element_id_equality
source_file: 0544_element_id_equality.htm
tags:
- csharp
- elements
- filtering
- references
- revit-api
- selection
- walls
title: Comparing Element Id for Equality
word_count: 265
---

### Comparing Element Id for Equality

Here is a very fundamental question that I was uncertain about myself, but happily Adam Nagy now clarified together with the development team:

**Question:** Is it safe to compare two element ids directly, or is it better to compare their integer values using the IntegerValue property instead?

In other words, given two ElementId variables id and id2, can I safely and reliably compare them using the following code?
```csharp
  bool equal = ( id == id2 );
```
Or should I better use the following?
```csharp
  bool equal = ( id.IntegerValue == id2.IntegerValue );
```

**Answer:** The ElementId comparison operator was added in the Revit 2011 API, so you can safely use both versions, and they should always return the same result.

By the way, I am uncertain but vaguely remember that unlike the == operator, the Equals method worked even before 2011.

Here is a quick test that I created and ran in Revit 2011 which shows that (id1 == id2) works correctly.
If the first created wall is picked, the dialogue box is displayed, otherwise not:
```csharp
  public void IdTest( UIDocument doc )
  {
    FilteredElementCollector collector
      = new FilteredElementCollector( doc.Document )
        .OfClass( typeof( Wall ) );

    ElementId id = collector.ToElementIds().First();

    Reference selRef = doc.Selection.PickObject(
      ObjectType.Element );

    if( id == selRef.Element.Id )
    {
      TaskDialog.Show( "Same Wall",
        "Just saying..." );
    }
  }
```

Many thanks to Adam, Scott Conover and Miroslav Schonauer for chipping in on this subject.

It is a relief for me to know for sure as well!
I have still been using the IntegerValue property until now, to be on the safe side, which adds unnecessary verbal and optical overload to my source code.