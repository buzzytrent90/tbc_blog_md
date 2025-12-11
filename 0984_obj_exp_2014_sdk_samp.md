---
ai_optimized: true
author: Jeremy Tammik
complexity_score: 3.7
content_type: qa
optimization_date: '2025-12-11T11:44:15.051432'
original_url: https://thebuildingcoder.typepad.com/blog/0984_obj_exp_2014_sdk_samp.html
post_number: 0984
reading_time_minutes: 5
series: general
slug: obj_exp_2014_sdk_samp
source_file: 0984_obj_exp_2014_sdk_samp.htm
tags:
- elements
- family
- geometry
- revit-api
- rooms
- schedules
- views
title: Revit 2014 OBJ Exporter and New SDK Samples
word_count: 1097
---

### Revit 2014 OBJ Exporter and New SDK Samples

Funnily enough, after dealing with the new Revit 2014
[custom exporter framework](http://thebuildingcoder.typepad.com/blog/2013/07/graphics-pipeline-custom-exporter.html) last
week, and even suggesting rewriting my
[Revit 2013 OBJ exporter](http://thebuildingcoder.typepad.com/blog/2013/07/graphics-pipeline-custom-exporter.html#6) to make use of it, a request came in for a version of the
[OBJ exporter for Revit 2014](#2), so I discuss that below.
I also add some more background to the list of new
[Revit 2014 SDK samples](#3).
Finally, to wrap up, we'll look at a surprisingly unbalanced distribution of Revit API
[blog page views per country](#4).

#### OBJ Exporter for Revit 2014

As
[mentioned](http://thebuildingcoder.typepad.com/blog/2013/07/graphics-pipeline-custom-exporter.html#6),
it would be nice to rewrite my OBJ exporter for Revit 2013 using the new Revit 2014 custom exporter framework.

However, a request already came in for a Revit 2014 version:

**Question:** Until now I have been using
[Lumion](http://www.lumion3d.com) for
visualisation.
It is fast and the result is OK.
However, I now discovered the
[Octane renderer](http://render.otoy.com) and
would like to try that as a visualisation tool instead.
Apparently, the only way to export a Revit model to Octane is via the OBJ file format.

**Answer:** In order to provide something immediately, I simply flat ported the Revit 2013 OBJ exporter to Revit 2014.
I also made use of the
[DisableMismatchWarning](http://thebuildingcoder.typepad.com/blog/2013/07/recursively-disable-architecture-mismatch-warning.html) utility
to fix the processor architecture mismatch warning MSB3270.

The current implementation thus still traverses all the Revit model elements one by one, retrieves their solids, exports the graphics face by face, and determines some, but not all, graphical attributes as described last year:

- [OBJ model export considerations](http://thebuildingcoder.typepad.com/blog/2012/06/obj-model-export-considerations.html)
- [Take one](http://thebuildingcoder.typepad.com/blog/2012/06/obj-model-exporter-take-one.html)
- [Colours](http://thebuildingcoder.typepad.com/blog/2012/07/obj-model-exporter-with-colours.html)
- [Multiple solid support](http://thebuildingcoder.typepad.com/blog/2012/07/obj-model-exporter-with-multiple-solid-support.html)
- [Transparency support](http://thebuildingcoder.typepad.com/blog/2012/07/obj-model-exporter-with-transparency-support.html)

Some of that complexity could be removed by rewriting this using the Revit 2014 custom exporter API.
Above all, the graphical properties could easily be better supported.

Running the OBJ exporter on the RAC basic sample model generated these
[OBJ](src/ObjExport/test/rac_basic_sample_project.obj) and
[MTL](src/ObjExport/test/rac_basic_sample_project.mtl) output
files and reported the following result:

![OBJ exporter result in RAC basic sample model](img/obj_export_2014_result_dlg.png)

Here is a sample visualisation in Octane of a Revit 2014 model exported to OBJ:

![Octane rendering of Revit model exported to OBJ](img/obj_export_2014_octane.jpeg)

As you can see, the texture mapping is rather strange.
Maybe the MTL file is not being properly processed.

Probably some face normals are also wrong, or, equivalently, the polygon loop vertex ordering is inverted.

Anyway, here is
[ObjExport2014.zip](zip/ObjExport2014.zip) containing the complete flat ported source code, Visual Studio solution and add-in manifest for the Revit 2014 OBJ exporter.

#### Revit 2014 SDK Samples

I already presented the list of
[new Revit SDK samples](http://thebuildingcoder.typepad.com/blog/2013/06/wishlist-survey-reminder-and-new-sdk-sample-overview.html#3) without
going into much detail.

I fleshed it out a bit more for the
[Revit API DevCamp in Moscow](http://thebuildingcoder.typepad.com/blog/2013/06/key-concepts-of-the-family-editor.html),
as part of my presentation on the
[Revit 2014 API](http://thebuildingcoder.typepad.com/blog/2013/03/revit-2014-api-and-room-plan-view-boundary-polygon-loops.html#2) news.

The shortest useful summary of the main new API features that I have been able to achieve is this:

- Copy and paste API – within or between documents, incl. view specific elements
- Project browser API – commands, macros, selected elements
- Schedule API – formatting, read-write data items
- Command API – launch macro, add-in and built-in
- Displaced elements API – exploded views
- Join geometry API – create, remove and control joins
- FreeForm element API – modification of imported solids
- Site API – editing of topography surface points and sub-regions
- Add-in API – loading and execution
- Macro API – list, create, delete and execute
- MEP calculations in external services
- Structural Analysis SDK
- Direct API access to rendering pipeline

The three last items are the only ones not covered by any new SDK samples.

For that reason, I recently went to some effort to explore the
[Structural Analysis SDK](http://thebuildingcoder.typepad.com/blog/2013/06/structural-analytical-code-checking-and-results-builder.html) and the
[custom exporter framework](http://thebuildingcoder.typepad.com/blog/2013/07/graphics-pipeline-custom-exporter.html).

The third item not covered yet concerns the MEP calculations in external services.
We are working on a good sample for that, hopefully coming soon.

All other important new features are covered by the following SDK samples:

- DisplacementElementAnimation
- DockableDialogs
- DuplicateViews
- ExtensibleStorageUtility
- FreeFormElement
- PostCommandWorkflow
- ScheduleAutomaticFormatter
- ScheduleToHTML
- SinePlotter
- Site
- Units
- WinderStairs

Several of these have already been discussed in the
[DevDays online presentation, recording and sample code](http://thebuildingcoder.typepad.com/blog/2013/03/revit-2014-api-and-room-plan-view-boundary-polygon-loops.html#2
) and other separate posts.

Here is my Moscow DevCamp
[Revit 2014 API slide deck](file:////a/j/adn/devcamp/2013/doc/2-1_revit_2014_api.pdf) that
includes one slide for each sample to provide a quick first impression of each.

Let's close for today with something not directly API related:

#### BIM Boosting Booms Much More in Four Specific Countries

Cheers to Australia, Denmark, Netherlands and Sweden!

When I visited Harry Mattison in Boston in connection with the
[Autodesk Tech Summit](http://thebuildingcoder.typepad.com/blog/2013/06/correct-detail-component-rotation-in-elevation-view.html#6), he mentioned that his
[Boost your BIM](http://boostyourbim.wordpress.com) page
views had an extremely unbalanced distribution for different countries, and very kindly provided some numbers correlated with the total country population:
![Boost my BIM page views per capita](img/boost_bim_page_views_jt.png)

As you can see, the top four countries have 4, 5 and even up to almost 7 page views per 100000 population, whereas Canada has 2.5, the USA and UK well under 2, and the rest of the world well under 1.

Rather strange, isn't it?

Thank you, Harry, for this perplexing little item.