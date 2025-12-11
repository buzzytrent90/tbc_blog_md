---
ai_optimized: true
author: Jeremy Tammik
complexity_score: 3.2
content_type: qa
optimization_date: '2025-12-11T11:44:15.547665'
original_url: https://thebuildingcoder.typepad.com/blog/1224_pipe_wall_thickness.html
post_number: '1224'
reading_time_minutes: 4
series: mep
slug: pipe_wall_thickness
source_file: 1224_pipe_wall_thickness.htm
tags:
- csharp
- elements
- family
- filtering
- geometry
- parameters
- revit-api
- views
- walls
- mep
title: Brussels Hackathon, Pipe Wall Thickness and Voids
word_count: 733
---

### Brussels Hackathon, Pipe Wall Thickness and Voids

I received an email asking whether it is possible to determine the Revit MEP [pipe element wall thickness](#3), and also about the API access to [voids in the family editor](#4).

Before getting to that, let me mention that I am now in Brussels at the [Open Data Hackathon](#2).

#### Hackathon Open Data Brussels

I arrived in Gent last night and returned to Brussels this morning for the
[Hackathon Open Data Brussels](http://www.transformabxl.be/agenda/event/hackathon-open-data-brussels) taking
place today and tomorrow, October 17-18 2014.

It is intended to promote the use of open data:

- [#hackabxl](https://twitter.com/hashtag/hackabxl)
- [#opendata](https://twitter.com/hashtag/opendata)

The
[list of available data sets](http://www.transformabxl.be/blog/post/hackathon-open-data-brussels-list-of-available-datasets) is
huge, comprising thousands – if not tens of thousands – of databases.

So far, this led me to take quick looks at a number of fascinating new topics and tools, e.g.:

- The [R programming language](http://cran.r-project.org) for statistics and mapping
- The [Kartograph](http://kartograph.org) framework for building interactive map applications
- [Crossfilter](http://square.github.io/crossfilter), fast multidimensional filtering for coordinated views
- The [D3.js](http://d3js.org) JavaScript library for manipulating documents based on data

Me and Cyrille co-initiated and joined the
[PoiPointer](https://github.com/PoiPointer) project,
with a goal of implementing an app pointing out points of interest.

Backend probably based on [ElasticSearch](http://www.elasticsearch.org).

Cyrille also presented our standard spiel on the Autodesk Data and View API:

- [Autodesk View and Data API](http://thebuildingcoder.typepad.com/blog/2014/07/autodesk-view-and-data-api.html)
- [View and Data API Presentation Material](http://thebuildingcoder.typepad.com/blog/2014/07/view-and-data-api-presentation-material.html)
- [Autodesk View and Data API Webinar](http://thebuildingcoder.typepad.com/blog/2014/09/autodesk-view-and-data-api-webinar.html)
- [ADVA Webinar notes](http://thebuildingcoder.typepad.com/blog/2014/10/adva-webinar-free-student-software-and-au.html#2)

I'll stop here and report more on this later.

Back to the Revit API...

#### Determining Pipe Wall Thickness

**Question:** I would like to compute the wall thickness of a given Revit MEP pipe instance and populate this value so it can be tagged on the pipe.

The desired output could be something like 'OD x WT', i.e. outer diameter x wall thickness, e.g. '21.3 x 2.0' for a DN15 pipe.

Is it possible to get at the necessary data through the API?

**Answer:** Revit does not provide a wall thickness parameter, but it can easily be derived from other data available and accessible in the model:

- The pipe instance has Pipe Segment and Diameter parameters.
- Manage > MEP Settings > Mechanical Settings > Pipe Settings > Segments and Sizes displays a drop list for each Pipe Segment
- Each pipe segment has a properties specifying its nominal, inside and outside diameters: Nominal (Diameter), ID (Inside diameter) and OD (Outside Diameter).
- The desired value WT (wall thickness) = (ID – OD) / 2.

Most Revit element properties are stored in parameters.

When possible, the parameter values should be access using the language independent and more efficient built-in parameter enumeration values instead of the language dependent display strings displayed to the used.

In this case, the built-in parameter enumeration values RBS\_PIPE\_INNER\_DIAM\_PARAM and RBS\_PIPE\_OUTER\_DIAMETER can be used, e.g. like this:

```csharp
  const BuiltInParameter bipDiameterInner
    = BuiltInParameter.RBS\_PIPE\_INNER\_DIAM\_PARAM;

  const BuiltInParameter bipDiameterOuter
    = BuiltInParameter.RBS\_PIPE\_OUTER\_DIAMETER;

  static double GetWallThickness( Pipe pipe )
  {
    double dinner = pipe.get\_Parameter(
      bipDiameterInner ).AsDouble();

    double douter = pipe.get\_Parameter(
      bipDiameterOuter ).AsDouble();

    return 0.5 \* ( douter - dinner );
  }
```

#### Voids in the Family Editor

**Question:** Are voids available in Revit 2015 through API?

I need programmatic access to the ability to create Void geometry in Family Editor.

It is required in order to create native geometry within a family, including some voids to get the right graphical representation.

**Answer:** Yes, sure.

An example of making use of programmatically created voids is given by the recent discussion of
[InstanceVoidCutUtils](http://thebuildingcoder.typepad.com/blog/2014/04/instancevoidcututils-and-need-for-regeneration.html).

The actual programmatic generation of families is illustrated by various examples making use of the
[Family API](http://thebuildingcoder.typepad.com/blog/about-the-author.html#5.25).