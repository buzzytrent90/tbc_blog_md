---
ai_optimized: true
author: Jeremy Tammik
complexity_score: 2.2
content_type: tutorial
optimization_date: '2025-12-11T11:44:15.430911'
original_url: https://thebuildingcoder.typepad.com/blog/1163_updates_adva_viewer.html
post_number: '1163'
reading_time_minutes: 3
series: views
slug: updates_adva_viewer
source_file: 1163_updates_adva_viewer.htm
tags:
- levels
- revit-api
- views
title: Updated SDK, DevTV, BIM 360 News and Viewer
word_count: 617
---

### Updated SDK, DevTV, BIM 360 News and Viewer

A whole bunch of updates, with the last one mentioned being the most exciting by far:

- [Revit SDK Update Release 2](#2)
- [DevTV Introduction to Revit 2015 API Programming](#3)
- [DevDay Online – BIM 360 Glue](#4)
- [DevDay Online – BIM 360 Field](#5)
- [Sneak Peek at the New Autodesk 360 Viewer](#6)

#### Revit SDK Update Release 2

The Revit SDK provided on the
[Revit Developer Centre](http://www.autodesk.com/developrevit) has been updated again to include the complete add-in manager, which was lacking in the
[last update](http://thebuildingcoder.typepad.com/blog/2014/04/compiling-the-revit-2015-sdk-and-migrating-bc-samples.html#3) labelled April 11, 2014.

Here is the direct link to the Revit 2015 UR2 SDK:

- [Revit 2015 SDK (Update May 14, 2014)](http://images.autodesk.com/adsk/files/REVIT2015SDK_UR2.msi) (msi - 242795Kb)

Please refer to the presentation of the last update for more suggestions on how to
[install and compile the SDK samples](http://thebuildingcoder.typepad.com/blog/2014/04/compiling-the-revit-2015-sdk-and-migrating-bc-samples.html).

#### DevTV Introduction to Revit 2015 API Programming

Talking about updates, my colleague
[Augusto Gonçalves](http://adndevblog.typepad.com/aec/augusto-goncalves.html) provided
an update of the DevTV Revit API tutorial for Revit 2015:
[DevTV: Introduction to Revit 2015 API Programming](http://adndevblog.typepad.com/aec/2014/04/devtv-introduction-to-revit-2015-api-programming.html):

#### DevDay Online – BIM 360 Glue

Augusto also published the
[DevDays Online recordings on BIM 360 Glue and Field](http://adndevblog.typepad.com/aec/2014/05/devday-online-bim-360-glue-and-field.html),
discussing the current status, showing the product, APIs and usage scenarios:

#### DevDay Online – BIM 360 Field

#### Sneak Peek at the New Autodesk 360 Viewer

Last but not least, Kean Walmsley just presented the first
[sneak peek at the new Autodesk 360 viewer](http://through-the-interface.typepad.com/through_the_interface/2014/05/a-sneak-peek-at-the-new-autodesk-360-viewer.html).

This is very exciting stuff!

Here is a Revit model of the Autodesk Waltham office building with good internal structure:

Aside from the standard zoom, pan and orbit, in this model, you can press the **structure** button
![Structure button](http://through-the-interface.typepad.com/.a/6a00d83452464869e201a73dccce59970d-pi "Structure button") to
browse down through the model's assembly structure or component hierarchy.
You can use this to isolate specific components in your model, hiding everything else.

Double click an individual building component to highlight it, list its identity and other properties.

Here is Kean's kitchen model sporting nice materials, but lacking structure to explore:

Please note that these super-simple single-line embedded viewers do not immediately support all the possible functionality.
It is – or will be soon, we hope – really easy to implement that in a slightly more full-fledged context, though.

Aside from the need to support a huge array of formats, the viewer is really good at streaming large models – displaying them at appropriate levels of detail – and allowing you to get in and work with the structure of these models.

For more information, please refer to
[Kean's original post](http://through-the-interface.typepad.com/through_the_interface/2014/05/a-sneak-peek-at-the-new-autodesk-360-viewer.html).

By the way, the
[vA3C](https://va3c.github.io) open
source three.js based AEC viewer project and
[RvtVa3c](http://thebuildingcoder.typepad.com/blog/2014/05/rvtva3c-revit-va3c-generic-aec-viewer-json-export.html) Revit
vA3C JSON model exporter that I worked on during the
[AEC Hackathon](http://thebuildingcoder.typepad.com/blog/2014/05/aec-hackathon-from-the-midst-of-the-fray.html) is
based on similar technology.