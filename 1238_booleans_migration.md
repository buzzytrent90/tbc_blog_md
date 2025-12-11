---
ai_optimized: true
author: Jeremy Tammik
complexity_score: 4.3
content_type: code_example
optimization_date: '2025-12-11T11:44:15.580972'
original_url: https://thebuildingcoder.typepad.com/blog/1238_booleans_migration.html
post_number: '1238'
reading_time_minutes: 6
series: general
slug: booleans_migration
source_file: 1238_booleans_migration.htm
tags:
- csharp
- doors
- elements
- family
- geometry
- parameters
- revit-api
- selection
- views
title: Migrating Deprecated API and 2D Boolean Operations
word_count: 1199
---

### Migrating Deprecated API and 2D Boolean Operations

While my colleagues are busy with the
[DevDays in Asia](http://thebuildingcoder.typepad.com/blog/2014/11/the-devdays-2014-conferences-have-started.html),
Let me mention two little questions that I addressed in the past few hours:

- [2D Boolean operations](#2)
- [Getting started migrating deprecated API](#3)

#### 2D Boolean Operations

**Question:** In AutoCAD development, the ObjectARX AcGe library provides powerful tools such as the AcDbRegion::booleanOper computational geometry unions and differences.

Does the Revit API provide anything similar for surfaces?
Is there any way to calculate and display the difference between two areas?

Revit is a great three-dimensional software, the above function is a common operation, but I did not find the relevant API.

Please help, thank you!

**Answer:** The Revit API does not provide any general-purpose geometric engine like the AutoCAD ARX AcGe library, because it is highly specialised and purely dedicated to BIM.

If you have a requirement for additional geometric functionality such as 2D Boolean operations, you will need to implement it yourself or make use of some additional external libraries. Numerous such libraries exist.

The Building Coder has implemented and presented several examples making use of 2D Boolean operations using external libraries.

They can be found through the topic list 5.2. on
[2D Booleans and Adjacent Areas](http://thebuildingcoder.typepad.com/blog/about-the-author.html#5.2),
which I just updated for you, adding a couple of new recent posts.

I hope this helps.

#### Getting Started Migrating Deprecated API

Raised by Fimo in a
[comment](http://thebuildingcoder.typepad.com/blog/2014/11/functional-programming-view-and-data-api-demos.html?cid=6a00e553e16897883301b7c70196b9970b#comment-6a00e553e16897883301b7c70196b9970b) or
[two](http://thebuildingcoder.typepad.com/blog/2014/11/functional-programming-view-and-data-api-demos.html?cid=6a00e553e16897883301b8d08c17c0970c#comment-6a00e553e16897883301b8d08c17c0970c) on
[functional programming, View and Data API demos](http://thebuildingcoder.typepad.com/blog/2014/11/functional-programming-view-and-data-api-demos.html):

**Question:** I am new to this blog and new to the Revit 2015 API, still not really knowing what I am doing and struggling through the examples here and in other places.

There are lots of useful examples and many of them are based on using "ElementSet". So I create an element set, like "ElementSet elems = selection.Elements;" That produces a message saying "don't use Selection.Elements, it's deprecated, use GetElementIds() instead." Of course I want to be a good guy, so I use "GetElementIds", only to find out that this does not work for "ElementSet".

Okay, so I abandon "ElementSet" and use "ICollectionElems" which is (grammatically) fine and produces no compiler errors. But now I have an ICollection and all the loads of code which I had found in the web so far do not work for me any more, as they were based on "ElementSet".

So here are my questions: is there a way to cast/convert "ICollection" to "ElementSet"? Am I the first one to come across this problem? What would be the professional approach to deal with this?

Many thanks for you advice...

**Answer:** Thank you for a very valid question.

ElementSet is deprecated in Revit 2015, just as the message tells you.
I guess the main change is caused by the
[Selection API changes](http://thebuildingcoder.typepad.com/blog/2014/04/whats-new-in-the-revit-2015-api.html#2.10).

However, as long as it has not been removed, you can still use it and ignore the warning.

All new code should avoid making use of it, since it will probably no longer work (and be removed) in the next release.

In most cases, you can replace it by a collection of ElementId objects.
However, its members are element ids and not Element instances, so there will be some differences in usage.

That is the issue you are encountering. Basically, the old code needs to be rewritten to take the collection of ids instead of the set of elements.

The Revit API used to define quite a number of custom collections.

They are all being replaced by generic .NET collections, by and by.

Also, a number of methods are now using ElementId arguments instead of straight Element ones.

This is part of a multi-year transition.

I hope this explains.

**Response:** Thanks for your detailed response.

Yes, it does explain...

But, alas, my personal conclusion is supposed to be the following: as there are mostly old style code snippets available and I do not know how to convert them into new style, I have to stick to the obsolete variants in order to get some results. That means I am now learning the old style. I hope Autodesk will wait a little longer before removing the old code... :-)

Anyway thanks a lot for your explanation.

**Answer:** Please do not jump to the wrong conclusion.

My personal conclusion from that is to learn the new style and convert to old snippets to the new style.

The conversion is mostly trivial. Here is an example: the
[DataToBimCountoursAndBuildings migration from Revit 2014 to 2015](https://github.com/jeremytammik/DataToBimCountoursAndBuildings/commit/88637e90dfa984ae424d655304e9e0a43f6b19c7).

Look specifically at the modification of the file DataToBim/DataToBuilding.cs:

```
    Family family = null;
    this.document.LoadFamily( path, out family );
    FamilySymbol symbol = null;
-   foreach( FamilySymbol s in family.Symbols )
+   //foreach( FamilySymbol s in family.Symbols ) // 2014
+   foreach( ElementId id in family.GetFamilySymbolIds() ) // 2015
    {
-     symbol = s;
+     symbol = document.GetElement( id ) as FamilySymbol;
      break;
    }
    symbols.Add( symbol );
```

I have migrated dozens of projects on The Building Coder and often documented the migration steps in great detail.

Simply look at The Building Coder category on
[Migration](http://thebuildingcoder.typepad.com/blog/migration).

Actually, I went one step further, just for your sake, and fixed some of the numerous deprecated API usage warnings still present in The Building Coder samples until I hit the exact one you are encountering and struggling with.

I started out with these
[65 warnings](zip/bc_migr_2015_b.txt).

I then globally replaced `sel.Elements.Size` by `sel.GetElementIds().Count`, reducing the count to
[46 warnings](zip/bc_migr_2015_c.txt).

Now I fixed just one of the warnings about iterating over the selection elements, in the module CmdListMarks.cs, by replacing the `Selection.Elements` iteration by a call to GetElementIds and iterating over the element ids instead, afterwards producing
[45 remaining warnings](zip/bc_migr_2015_d.txt).

Here is the new code, marked with '// 2015', and the old code, commented out and marked with '// 2014':

```csharp
  if( \_modify\_existing\_marks )
  {
    //ElementSet els = uidoc.Selection.Elements; // 2014

    ICollection<ElementId> ids = uidoc.Selection.GetElementIds(); // 2015

    //foreach( Element e in els ) // 2014

    foreach( ElementId id in ids ) // 2015
    {
      Element e = doc.GetElement( id ); // 2015

      if( e is FamilyInstance
        && null != e.Category
        && (int) BuiltInCategory.OST\_Doors
          == e.Category.Id.IntegerValue )
      {
        e.get\_Parameter(
          BuiltInParameter.ALL\_MODEL\_MARK )
          .Set( \_the\_answer );

        ++n;
      }
    }
  }
```

I very much hope that this suffices to convince you to fix the warnings you encounter.

As always, the most up to date version of The Building Coder samples is provided in
[its GitHub repository](https://github.com/jeremytammik/the_building_coder_samples),
and the version described above is
[release 2015.0.115.1](https://github.com/jeremytammik/the_building_coder_samples/releases/tag/2015.0.115.1).

I hope this helps.