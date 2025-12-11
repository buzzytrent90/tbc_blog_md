---
ai_optimized: true
author: Jeremy Tammik
complexity_score: 1.2
content_type: qa
optimization_date: '2025-12-11T11:44:15.503469'
original_url: https://thebuildingcoder.typepad.com/blog/1203_custom_exporter_camera.html
post_number: '1203'
reading_time_minutes: 1
series: general
slug: custom_exporter_camera
source_file: 1203_custom_exporter_camera.htm
tags:
- csharp
- elements
- filtering
- parameters
- revit-api
- views
title: Custom Exporter GetCameraInfo
word_count: 281
---

### Custom Exporter GetCameraInfo

Here is a simple yet longstanding question with a surprisingly simple answer that should prove extremely helpful for all those who really need it, presumably including Mohan Sawant, who raised this issue in a
[comment](http://thebuildingcoder.typepad.com/blog/2009/04/dwf-view-definition.html?cid=6a00e553e16897883301b8d061675b970c#comment-6a00e553e16897883301b8d061675b970c) on
the
[DWF view definition](http://thebuildingcoder.typepad.com/blog/2009/04/dwf-view-definition.html) and
its camera settings:

**Question:** Is there any programmatic access to the Revit camera target and
[FOV](http://en.wikipedia.org/wiki/Field_of_view)?

For my application, I have to define a camera using position, target, up vector, projection type and field of view.

I tried to read the camera parameters from the View3d object returned by the filtered element collector. From this I can get the up vector and position.

How can I determine the other parameters for the camera object, especially the target and FOV?

**Answer by Arnošt Löbel:** The only way I know of is using a custom exporter.

When a view is processed and run through a custom context, its properties will be used to populate a ViewNode instance.

One of its methods is GetCameraInfo, which provides information that ought to cover everything you need to know about the view's camera.

Granted, this is not the most straightforward way to get the information, but keep in mind that the exporter itself would be very simple and would not need to do anything else whatsoever.

Many thanks, Arnošt, for your help!

I put together a list of
[custom exporter](http://thebuildingcoder.typepad.com/blog/about-the-author.html#5.1) discussions
that you can refer to for example implementations.