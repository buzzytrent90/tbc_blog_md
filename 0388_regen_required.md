---
ai_optimized: true
author: Jeremy Tammik
complexity_score: 2.6
content_type: qa
optimization_date: '2025-12-11T11:44:13.864203'
original_url: https://thebuildingcoder.typepad.com/blog/0388_regen_required.html
post_number: 0388
reading_time_minutes: 3
series: general
slug: regen_required
source_file: 0388_regen_required.htm
tags:
- csharp
- elements
- family
- levels
- revit-api
title: To Regenerate or Not to Regenerate...
word_count: 506
---

### To Regenerate or Not to Regenerate...

... that is the question.

We already discussed some
[best practices concerning the regeneration option](http://thebuildingcoder.typepad.com/blog/2010/04/regeneration-option-best-practices.html).
A related topic is how often and when to make calls to the Document.Regenerate method, and how to eliminate such calls when possible.

One of the issues we ran into during today's DevLab had to do with performance and elimination of extraneous calls to this method.
We also learned that the location curve of a newly inserted family instance does not reflect the true insertion location until such a call has been made.

One participant is inserting a large number of vertical columns into the model, which require a rotation around their axis after insertion.
The original code required a document regeneration after each insertion and before rotating each column, which was very expensive and seemed rather mysterious.
This is actually a continuation of the issues we discussed in the comments on the post on
[performance profiling](http://thebuildingcoder.typepad.com/blog/2010/03/performance-profiling.html).

We finally tracked down the problem and found a simple solution to this.
The need for the extra regeneration call was caused because the rotation axis was being determined by querying each element for its location curve, i.e. something like this:
```csharp
  XYZ p = get\_insertion\_point();
  XYZ q = p + XYZ.BasisZ;

  Line line = creapp.NewLineBound( p, q );

  FamilyInstance fi = credoc.NewFamilyInstance(
    line, symbol, level, structuralType );

  doc.Regenerate();

  LocationCurve lc = fi.Location as LocationCurve;

  Line axis = lc.Curve as Line;

  doc.Rotate( fi, axis, angle );
```

Unfortunately, the code making the call to rotate the element was implemented in a different function, and it was not at all obvious that the rotation axis could easily make use of the same data that was used to define the symbol insertion point.
Instead, querying the element location curve and using that to define the rotation axis seemed like a much more self-contained solution.

Every attempt to eliminate the regeneration call caused the element to return a stale location curve, which caused the rotation to unexpectedly move its position.

A much simpler solution is to determine the rotation axis directly from the known insertion point.
Then we no longer rely on the element location property, and the regeneration can be eliminated:
```csharp
  XYZ p = get\_insertion\_point();
  XYZ q = p + XYZ.BasisZ;

  Line line = creapp.NewLineBound( p, q );

  FamilyInstance fi = credoc.NewFamilyInstance(
    line, symbol, level, structuralType );

  Line axis = line;

  doc.Rotate( fi, axis, angle );
```

A good thing to know.
Always be aware that not regenerating may cause properties to return stale values.
If you have other reliable methods to access up-to-date information, make use of them, and avoid the danger of stale data.

Looking at the code cleaned up and isolated like this, the use of the original line as the rotation axis seems very obvious.
In the original version, with the rotation was performed in a different function, it was not obvious at all.