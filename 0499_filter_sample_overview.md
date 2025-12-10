---
post_number: "0499"
title: "Filtered Element Collector Sample Overview"
slug: "filter_sample_overview"
author: "Jeremy Tammik"
tags: ['doors', 'elements', 'family', 'filtering', 'levels', 'parameters', 'revit-api', 'sheets', 'views', 'walls']
source_file: "0499_filter_sample_overview.htm"
original_url: "https://thebuildingcoder.typepad.com/blog/0499_filter_sample_overview.html"
---

### Filtered Element Collector Sample Overview

The developer conference in Paris went well, and Adam, Partha and I spent a nice day exploring Paris using their wonderful
[velib](http://en.wikipedia.org/wiki/V%C3%A9lib%27) bicycle rental system.
You pay just one euro for a one-day membership.
It allows you to pick up and return a bicycle at any one of the city's hundreds of completely automated and networked rental points.
If you return the bike within thirty minutes, it is free of charge.
Beyond that time, it costs a euro or two per half hour extra.

Here are some impressions of our tour; they also have a lot to do with filtering, by the way.
First, here is a happy Karl Osti enjoying a smoke in Vienna airport en route from Moscow to Paris:

![Karlo Osti enjoying a smoke](file:////j/photo/jeremy/2010/2010-12-11_devdays_paris/img_0037.jpg)

He can be retrieved using a dedicated **CarloFilter**, represented here by this smoker's cabin.

Karl, in turn, uses a **MarlboroFilter**.

Here are Adam and Partha using a **BicycleFilter**, after the first leg of rental bicycling up from Bercy along the
[Boulevard du Temple](http://en.wikipedia.org/wiki/Boulevard_du_Temple):

![Adam and Partha enjoying the rental bicycles](file:////j/photo/jeremy/2010/2010-12-11_devdays_paris/img_0046.jpg)

Here are the three of us using a **PedestrianBridgeFilter** crossing the
[Canal Saint-Martin](http://en.wikipedia.org/wiki/Canal_Saint-Martin) at Quai de Valmy close to the
[Gare de l'Est](http://en.wikipedia.org/wiki/Gare_de_l%27Est):

![Canal Saint-Martin close to the Gare de l'Est](file:////j/photo/jeremy/2010/2010-12-11_devdays_paris/img_0051.jpg)

And here we are on
[Montmartre](http://en.wikipedia.org/wiki/Montmartre) in
front of the
[Sacre Coeur basilica](http://en.wikipedia.org/wiki/Basilica_of_the_Sacr%C3%A9_C%C5%93ur):

![On Montmartre in front of Sacre Coeur](file:////j/photo/jeremy/2010/2010-12-11_devdays_paris/img_0063.jpg)

More photos are available in the
[Facebook album](http://www.facebook.com/album.php?aid=89693&id=1019863650&l=b9583fec63).

Meanwhile, returning to Revit, here is something I have been waiting to do for quite a while now, but new things kept cropping up.
After discussing my own
[filtered element collector benchmarks](http://thebuildingcoder.typepad.com/blog/2010/04/collector-benchmark.html) early
on in the Revit 2011 release cycle,
[Kevin's filtering samples and benchmarks](http://thebuildingcoder.typepad.com/blog/2010/12/more-kevin-filtering-benchmarks.html) last week,
and a large number of other samples in between,
I hope that we have covered everything relevant by now and the right moment has finally arrived to present a pretty extensive filtered element collector sample overview:

- My first Revit 2011 plug-in, the [pipe to conduit converter](http://thebuildingcoder.typepad.com/blog/2010/05/pipe-to-conduit-converter.html).
- A very simple collector to retrieve [view sheets](http://thebuildingcoder.typepad.com/blog/2010/06/export-data-to-xml.html).
- Some utility methods to filter for [levels and types](http://thebuildingcoder.typepad.com/blog/2010/06/set-tag-type.html) such as door type and a door tag type symbol.
- Retrieving [newly added elements](http://thebuildingcoder.typepad.com/blog/2010/04/retrieving-newly-created-elements-in-revit-2011.html).
- Retrieving [stairs on a level](http://thebuildingcoder.typepad.com/blog/2010/04/retrieve-stairs-on-level.html).
- Retrieving [views](http://thebuildingcoder.typepad.com/blog/2010/04/filter-for-views-and-istemplate-predicate.html).
- Retrieving [family instances and their family](http://thebuildingcoder.typepad.com/blog/2010/05/get-type-id-and-preview-image.html).
- Retrieving [title block symbols and instances](http://thebuildingcoder.typepad.com/blog/2010/05/determine-sheet-size.html) to determine the sheet size.
- Retrieving [AVF display styles with a specific name](http://thebuildingcoder.typepad.com/blog/2010/06/display-webcam-image-on-building-element-face.html#3).
- Retrieving [MEP elements](http://thebuildingcoder.typepad.com/blog/2010/06/retrieve-mep-elements-and-connectors.html).
- Retrieving [structural elements](http://thebuildingcoder.typepad.com/blog/2010/07/retrieve-structural-elements.html).
- Retrieving [text notes and text note types](http://thebuildingcoder.typepad.com/blog/2010/11/purge-unused-text-note-types.html).
- Retrieving [model elements](http://thebuildingcoder.typepad.com/blog/2010/10/selecting-model-elements.html), with some [enhancements](http://thebuildingcoder.typepad.com/blog/2010/10/model-elements-revisited.html).
- A [collector performance benchmark](http://thebuildingcoder.typepad.com/blog/2010/04/collector-benchmark.html) including several collector samples.
- [Using LINQ and anonymous methods in VB](http://thebuildingcoder.typepad.com/blog/2010/04/anonymous-methods-in-vb.html).
- A RevitLookup update and [retrieving all database elements](http://thebuildingcoder.typepad.com/blog/2010/05/revitlookup-update.html).
- Why [**not** to retrieve all database elements](http://thebuildingcoder.typepad.com/blog/2010/06/filter-for-all-elements.html).
- Many examples of [parameter filters](http://thebuildingcoder.typepad.com/blog/2010/06/parameter-filter.html).
- [Parameter filter for an element name](http://thebuildingcoder.typepad.com/blog/2010/06/element-name-parameter-filter-correction.html).
- [Parameter filter for a shared parameter](http://thebuildingcoder.typepad.com/blog/2010/08/elementparameterfilter-with-a-shared-parameter.html).
- Filtering for a [non-native class](http://thebuildingcoder.typepad.com/blog/2010/08/filtering-for-a-nonnative-class.html), i.e. "an element type that exists in the API, but not in Revit's native object model".
- Resolving another non-native class issue, the Panel class in the [MeasurePanelArea](http://thebuildingcoder.typepad.com/blog/2010/10/measurepanelarea_update.html) SDK sample.
- Filter for [view and phase](http://thebuildingcoder.typepad.com/blog/2010/09/filter-for-view-and-phase.html).
- [Level filter benchmark](http://thebuildingcoder.typepad.com/blog/2010/10/level-filter-benchmark.html) comparing Level filter versus parameter filter versus LINQ post-processing to retrieve elements on a given level.
- [XML family usage report](http://thebuildingcoder.typepad.com/blog/2010/12/xml-family-usage-report.html) using FamilySymbolFilter and FamilyInstanceFilter.
- Retrieving [intersecting elements](http://thebuildingcoder.typepad.com/blog/2010/12/find-intersecting-elements.html) using collector viewId constructor, BoundingBoxIntersectsFilter and exclusion filter.
- Retrieving [wall types with a specific name](http://thebuildingcoder.typepad.com/blog/2010/11/launching-a-revit-command.html) and [walls using a specific wall type](http://thebuildingcoder.typepad.com/blog/2010/11/launching-a-revit-command.html) using parameter filters.
- Kevin's [optimisation tips and tricks](http://thebuildingcoder.typepad.com/blog/2010/10/filtered-element-collectors.html) presentation.
- [Kevin's filtering samples and benchmarks](http://thebuildingcoder.typepad.com/blog/2010/12/more-kevin-filtering-benchmarks.html).

I hope this provides a useful knowledge base to enable you to solve all your element retrieval needs.
Obviously, some creativity on your side will also be required.