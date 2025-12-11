---
ai_optimized: true
author: Jeremy Tammik
complexity_score: 2.8
content_type: qa
optimization_date: '2025-12-11T11:44:14.102073'
original_url: https://thebuildingcoder.typepad.com/blog/0524_newfamilyinstance.html
post_number: '0524'
reading_time_minutes: 3
series: elements
slug: newfamilyinstance
source_file: 0524_newfamilyinstance.htm
tags:
- csharp
- doors
- family
- parameters
- revit-api
- elements
title: NewFamilyInstance Overloads
word_count: 535
---

### NewFamilyInstance Overloads

We have discussed a number of issues with finding the right one of the numerous overloads of the NewFamilyInstance method for placing instances of a specific family.
Here is another question on this, which provides a welcome opportunity to summarise what we have looked at so far:

**Question:** I have a case where I want to insert a series of families with a statement like this:
```csharp
  FamilyInstance instance =
    documentProject.Create.NewFamilyInstance(
      location, symbol,
      StructuralType.NonStructural );
```

This works fine as long as it is a simple family with an insertion point.
It doesn't work if the family is hosted or if the insertion requires two points.
How can I determine which type of insert the family requires?

**Answer:** You are absolutely right.
The NewFamilyInstance method has a number of overloads, and you need to select the correct one depending on various characteristics of the family you are inserting an instance from.
This is discussed in the developer guide in section 12.3.5 'Creating FamilyInstance Objects'.
The issue has also come up a number of times in the past here on the blog in the following situations:

- [Column](http://thebuildingcoder.typepad.com/blog/2009/02/inserting-a-column.html)- [Beam](http://thebuildingcoder.typepad.com/blog/2009/02/inserting-a-beam.html)- [Slanted column](http://thebuildingcoder.typepad.com/blog/2009/06/creating-a-slanted-column.html)- [Curved Beam](http://thebuildingcoder.typepad.com/blog/2009/06/creating-a-curved-beam.html)- [Lighting fixture](http://thebuildingcoder.typepad.com/blog/2009/08/electrical-settings-and-lighting-fixtures.html)- [Nested family](http://thebuildingcoder.typepad.com/blog/2009/11/nested-family.html)- [Face hosted sprinkler](http://thebuildingcoder.typepad.com/blog/2010/01/insert-facehosted-sprinkler.html)- [All beams require a curve](http://thebuildingcoder.typepad.com/blog/2010/04/beam-requires-curve.html)- [Regeneration after placement](http://thebuildingcoder.typepad.com/blog/2010/06/to-regenerate-or-not-to-regenerate.html)- [Door and door tag](http://thebuildingcoder.typepad.com/blog/2010/06/set-tag-type.html)- [Beam with a void](http://thebuildingcoder.typepad.com/blog/2010/07/beam-maker-using-a-void-extrusion-to-cut.html)- [Flex duct and fitting](http://thebuildingcoder.typepad.com/blog/2010/08/flex-duct-start-tangent.html)- [Detail instance](http://thebuildingcoder.typepad.com/blog/2010/10/place-detail-instance.html)- [Site component](http://thebuildingcoder.typepad.com/blog/2010/11/place-site-component.html)- [Furniture](http://thebuildingcoder.typepad.com/blog/2010/11/place-furniture-instance.html)

The second-last post includes the definition of a method named
[TestAllOverloads](http://thebuildingcoder.typepad.com/blog/2010/11/place-site-component.html) which
enables you to try out all possible overloads of the NewFamilyInstance method to see which one works best for a specific family.

The best hint of all was provided
[below](http://thebuildingcoder.typepad.com/blog/2011/01/newfamilyinstance-overloads.html?cid=6a00e553e1689788330167650dbed9970b#comment-6a00e553e1689788330167650dbed9970b) by
[Rolando Hijar](http://www.youtube.com/user/rolandohijar/videos) after
the initial publication of this.
He points out that the hosting behavior of a family can be determined by accessing the built-in parameter FAMILY\_HOSTING\_BEHAVIOR:
```csharp
int hosttype
= family.get\_Parameter(
BuiltInParameter.FAMILY\_HOSTING\_BEHAVIOR )
.AsInteger();
```

I hope these examples give you enough to get started with, at least!