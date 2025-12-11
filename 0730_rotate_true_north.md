---
ai_optimized: true
author: Jeremy Tammik
complexity_score: 2.0
content_type: qa
optimization_date: '2025-12-11T11:44:14.476051'
original_url: https://thebuildingcoder.typepad.com/blog/0730_rotate_true_north.html
post_number: '0730'
reading_time_minutes: 1
series: general
slug: rotate_true_north
source_file: 0730_rotate_true_north.htm
tags:
- csharp
- elements
- revit-api
title: Rotate True North
word_count: 232
---

### Rotate True North

We have previously looked at how to handle the
[project location](http://thebuildingcoder.typepad.com/blog/2010/01/project-location.html) properties
and more specifically how to handle the effect of a
[rotated Project North](http://thebuildingcoder.typepad.com/blog/2009/10/unrotate-north.html) on
an element azimuth.

Here is a question on the 'opposite' topic, how to set up a project north rotation in the first place:

**Question:** Is there any way to rotate the truth north using the Revit API?

It can be done in the user interface using the Manage > Project Location > Position > Rotate True North command:

![Rotate True North in Japanese](img/rotate_true_north_jp.png)

Here is the English command with its expanded tooltip:

![Rotate True North in English](img/rotate_true_north_en.png)

**Answer:** I believe the same effect can be achieved by rotating the **Project** North which is available in the API.

If this fits the bill, then here are some working code snippets making use of the related classes and methods:
```csharp
  ProjectLocation plCurrent
    = \_doc.ActiveProjectLocation;

  // . . .

  ProjectPosition newPosition
    = \_app.Create.NewProjectPosition(
      scaleToFT \* cs.OriginX,
      scaleToFT \* cs.OriginY,
      scaleToFT \* cs.OriginZ,
      theRotationYouWant );

  plCurrent.set\_ProjectPosition(
    ptOrigin, newPosition );
```

You may have to do this for **all** project locations.
A bit of 3D maths and brainwork may be required, but it should be doable.

Many thanks to Katsuaki Takamizawa and Miroslav Schonauer for this discussion and suggestion!