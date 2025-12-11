---
ai_optimized: true
author: Jeremy Tammik
complexity_score: 5.1
content_type: qa
optimization_date: '2025-12-11T11:44:15.599280'
original_url: https://thebuildingcoder.typepad.com/blog/1246_webgl_sorted_level.html
post_number: '1246'
reading_time_minutes: 4
series: general
slug: webgl_sorted_level
source_file: 1246_webgl_sorted_level.htm
tags:
- csharp
- elements
- filtering
- levels
- revit-api
- views
title: WebGL Goes Mobile and Sorted Level Retrieval
word_count: 775
---

### WebGL Goes Mobile and Sorted Level Retrieval

Today, let's look at a generic WebGL and a specialised Revit API issue: [WebGL on all Apple platforms](#2) and [sorted level retrieval](#3).

#### WebGL Goes Mobile

Apple announced
[full support of WebGL for its desktop and mobile browsers](http://venturebeat.com/2014/11/18/thanks-to-apple-webgl-goes-truly-cross-platform),
so WebGL goes truly cross platform, as pointed out by Paul Flanagan:

> Previous claims of cross-platform support for this HTML5 standard were largely worthless because they did not include the iPhone, iPad, or desktop Safari. That has changed, and this standard for 3D and interactive web experiences now works across all... platforms that matter.
>
> Rich 3D browser experience will be everything we currently see on the Web, from more interactive and engaging websites through to games which will work cross platform without the need for a big porting effort and all of the advantages of a real-time cloud platform.

This raises an interesting question: will WebGL apps running in web components be performant enough that nobody will need a native SDK for the mobile platforms? It seems that the web version of WebGL will be available for native apps through web components very soon.

We are happily riding the front of this wave with the WebGL-based Autodesk [View and Data API](http://thebuildingcoder.typepad.com/blog/about-the-author.html#5.46).

#### Retrieve Levels Sorted by Elevation

A quick look at a code snippet based on the Revit API forum discussion thread on
[sorting levels by elevation](http://forums.autodesk.com/t5/revit-api/sort-levels-by-elevation/m-p/5407203) raised
by jmcclanathan with contributions from peterjegan, sanjaymann and my colleague
[Aaron Lu](http://adndevblog.typepad.com/aec/aaron-lu.html) –
by the way, Aaron joined the DevTech team this very month and brings with him a wealth of experience in AEC products, both cloud and desktop – a warm welcome to Aaron, and thank you for your active participation in the forum and everywhere else!

**Question:** How would I go about getting a collection of levels sorted by elevation?
From reading the info I've found so far I think that filtered element collector returns an IEnumerable and that can be sorted by using 'OrderBy', then ToElementIds could be used to create the collection of element ids in the correct order.
Is that correct?

Here is some code I tried and didn't work:

```csharp
  UIDocument uiDoc = this.ActiveUIDocument;
  Document doc = uiDoc.Document;

  FilteredElementCollector levCollector
    = new FilteredElementCollector( doc );

  ICollection<Element> levelsCollection
    = levCollector.OfClass( typeof( Level ) )
      .OrderBy( lev => lev.Elevation )
      .ToElementIds();
```

Can someone please point me in the right direction?

Also, is an ICollection sortable?

**Answer 1:** I didn't look at all the other stuff, but if you define an Element collection, you should fill it with Elements, not ElementIds:

```csharp
  ICollection<Element>... .ToElementIds();
```

**Answer 2:**
```csharp
  static IOrderedEnumerable<Level> FindAndSortLevels(
    Document doc )
  {
    return new FilteredElementCollector( doc )
      .WherePasses( new ElementClassFilter( typeof( Level ), false ) )
      .Cast<Level>()
      .OrderBy( e => e.Elevation );
  }
```

**Answer 3:** You can change this part:

```csharp
  ICollection<Element> levelsCollection
    = levCollector.OfClass( typeof( Level ) )
      .OrderBy( lev => lev.Elevation )
      .ToElementIds();
```

To this:

```csharp
  List<Level> levelsCollection
    = levCollector.OfClass( typeof( Level ) )
      .OfType<Level>()
      .OrderBy( lev => lev.Elevation )
      .ToList();
```

Reasons:

- Return value can't be ICollection<Element> when you call .ToElementIds.
- OfClass() method will return a collection of Element, you can only order them by elevation after casting Element to Level, that's why I suggest using OfType<Level>().
- OrderBy() method returns IOrderedEnumerable<T>, if you want to have index operator, i.e. list[i], you can call ToList() to let it return a List<T>.

**Answer 4:** Additional comments from Jeremy:

The shortcuts are always preferable over explicit filter instantiations, since they are guaranteed to be ***quick*** filters, which is why Aaron prefers OfClass to the explicit ElementClassFilter instantiation.

You should also always avoid instantiating new collections when possible, since they may create copies of the existing data.

The filtered element collector itself is already iterable, so often there is no need to instantiate any .NET collections at all, c.f. the discussion on
[FindElement and collector optimisation](http://thebuildingcoder.typepad.com/blog/2012/09/findelement-and-collector-optimisation.html).

As a result of those considerations, here is my suggestion for an optimal solution:

```csharp
  IOrderedEnumerable<Level> GetSortedLevels( Document doc )
  {
    return new FilteredElementCollector( doc )
      .OfClass( typeof( Level ) )
      .Cast<Level>()
      .OrderBy( lev => lev.Elevation );
  }
```

For many other filtered element collector code examples, including this one, please refer to
[The Building Coder samples](https://github.com/jeremytammik/the_building_coder_samples), specifically the module
[CmdCollectorPerformance.cs](https://github.com/jeremytammik/the_building_coder_samples/blob/master/BuildingCoder/BuildingCoder/CmdCollectorPerformance.cs).