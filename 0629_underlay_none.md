---
ai_optimized: true
author: Jeremy Tammik
complexity_score: 2.8
content_type: qa
optimization_date: '2025-12-11T11:44:14.294918'
original_url: https://thebuildingcoder.typepad.com/blog/0629_underlay_none.html
post_number: 0629
reading_time_minutes: 1
series: general
slug: underlay_none
source_file: 0629_underlay_none.htm
tags:
- csharp
- elements
- levels
- parameters
- references
- revit-api
- views
title: Set Underlay Display Property to None
word_count: 220
---

### Set Underlay Display Property to None

Here is a short and sweet question handled by Joe Ye:

**Question:** I am creating new levels using the API.
All my existing levels have their underlay display property set to None, but the new ones all have it set to the previous level.

How can I programmatically change this setting to None as well?

**Answer:** This property is stored in the "Underlay" parameter on the level.
You can set it to None by assigning a null ElementId.
Here is a sample code snippet showing how to create a null ElementId and assign it to the parameter:
```csharp
  ElementId id = new ElementId( -1 );
  e.get\_Parameter( "Underlay" ).Set( id );
```

For production code, I would suggest using the built-in parameter instead of the user visible display name to identify the parameter.
Furthermore, the ElementId class defines a static property InvalidElementId which returns the invalid ElementId whose IntegerValue is -1.
Using these two improvements, the language independent code would look like this:
```csharp
  e.get\_Parameter( BuiltInParameter.VIEW\_UNDERLAY\_ID )
    .Set( ElementId.InvalidElementId );
```

ElementId.InvalidElementId is commonly used in the Revit API to denote an invalid or void element reference, for instance to mark the
[deletion of an element referenced in extensible storage](http://thebuildingcoder.typepad.com/blog/2011/06/extensible-storage-features.html#7).