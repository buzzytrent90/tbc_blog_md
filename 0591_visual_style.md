---
ai_optimized: true
author: Jeremy Tammik
complexity_score: 2.9
content_type: qa
optimization_date: '2025-12-11T11:44:14.224667'
original_url: https://thebuildingcoder.typepad.com/blog/0591_visual_style.html
post_number: 0591
reading_time_minutes: 2
series: general
slug: visual_style
source_file: 0591_visual_style.htm
tags:
- csharp
- family
- parameters
- revit-api
- views
title: Set the Visual Style of a View
word_count: 327
---

### Set the Visual Style of a View

Today is the day of the
[Chinese Revit API webcast](http://thebuildingcoder.typepad.com/blog/2011/05/chinese-revit-2012-api-webcast.html),
and also the holiday of
[Ascension Day](http://en.wikipedia.org/wiki/Ascension_Day) in
many part of Europe.
Just like last year, it is raining cats and dogs again.

Anyway, here is a useful answer to a little question that arises from time to time:

**Question:** I'm creating a new 3D view of a family instance like this:
```csharp
  Dim v As View3D = doc.Create.NewView3D( \_
    famInst.FacingOrientation)
  v.SectionBox = famInst.BoundingBox(v)
```

Now I would like to set its visual style to 'Shaded' or 'Shaded with Edges'.

Is there any way to achieve this using the API?

**Answer:** The view style can be changed by modifying the value of the 'Visual Style' parameter, which corresponds to the built-in parameter MODEL\_GRAPHICS\_STYLE.
Each option in the 'Visual Style' property drop-down list in the property dialog corresponds to an integer value from 0 to 6:

1. Wireframe- Hidden line- Shaded- Shaded with Edges- Consistent Colors- Realistic

In order to change the visual style to 'Shaded with edges', please set the parameter value to 4, for instance like this:
```csharp
  view.get\_Parameter(
    BuiltInParameter.MODEL\_GRAPHICS\_STYLE )
    .set( 4 );
```

These numbers may have changed since they were first established.
You can determine the correct numbers to use yourself by simply cycling through all the styles manually and checking the current parameter value as an integer each time, e.g. using the
[RevitLookup](http://thebuildingcoder.typepad.com/blog/2010/05/revitlookup-update.html) tool.

#### Project Vasari 2

The
[technology preview 2](http://labs.autodesk.com/utilities/vasari/) of
the conceptual design and analysis tool Project Vasari, which includes access to the
[Revit API](http://thebuildingcoder.typepad.com/blog/2010/11/project-vasari-api.html),
is now available in an updated version compatible with Revit 2012: