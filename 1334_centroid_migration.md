---
ai_optimized: true
author: Jeremy Tammik
complexity_score: 3.3
content_type: qa
optimization_date: '2025-12-11T11:44:15.792950'
original_url: https://thebuildingcoder.typepad.com/blog/1334_centroid_migration.html
post_number: '1334'
reading_time_minutes: 5
series: general
slug: centroid_migration
source_file: 1334_centroid_migration.htm
tags:
- elements
- family
- references
- revit-api
- views
title: Dynamo, Centroid & Volume Calculation Migration Blitz
word_count: 967
---

### Dynamo, Centroid & Volume Calculation Migration Blitz

My French colleague
[Olivier Bayle](http://villagebim.typepad.com/villagebim/olivier-bayle.html),
co-author of the French AEC-related
[Village BIM](http://villagebim.typepad.com) blog,
just re-raised the topic of my old
[solid centroid and volume calculation](http://thebuildingcoder.typepad.com/blog/2012/12/solid-centroid-and-volume-calculation.html) add-in.

Let's also point out one or two of the numerous topics we tackled in the past few days on the Revit API discussion forum:

- [Graphically displaying the centre of gravity using Dynamo](#2)
- [GetCentroid on GitHub and blitz migration across four Revit API releases](#3)
- [Finding the orientation of welded pipe outlets](#4)
- [How to set the change type for DMU AddTrigger](#5)

#### Graphically Displaying the Centre of Gravity using Dynamo

Olivier uses the GetCentroid add-in to implement a Dynamo script graphically displaying the centre of gravity of a bunch of selected Revit elements, as demonstrated by his two-minute June 22 YouTube video
[Revit afficher le centre de gravité à l'aide de Dynamo](https://www.youtube.com/watch?v=jBgkhXqWh-w):

Le but de cet article est de vous montrer que Dynamo devient le véritable compagnon de Revit notamment lorsqu’il permet d’accroitre les fonctionnalités de Revit.

We demonstrate how Dynamo is becoming a true companion of Revit, e.g., by enabling addition of enhanced functionality.

Looking forward to seeing the upcoming Village BIM article on this topic   :-)

#### GetCentroid on GitHub and Blitz Migration Across Four Revit API Releases

Prompted by Olivier's request, I created a new home for the original
[GetCentroid add-in](http://thebuildingcoder.typepad.com/blog/2012/12/solid-centroid-and-volume-calculation.html) add-in implementation, which now lives in its own cosy little
[GetCentroid GitHub repository](https://github.com/jeremytammik/GetCentroid).

I performed a 30-minute blitz migration of it from Revit 2013 across all later versions up to Revit 2016:

- [2013.0.0.0](https://github.com/jeremytammik/GetCentroid/releases/tag/2013.0.0.0) – initial release from december 2012
- [2014.0.0.0](https://github.com/jeremytammik/GetCentroid/releases/tag/2014.0.0.0) – flat migration to Revit 2014
- [2015.0.0.0](https://github.com/jeremytammik/GetCentroid/releases/tag/2015.0.0.0) – flat migration to Revit 2015
- [2015.0.0.1](https://github.com/jeremytammik/GetCentroid/releases/tag/2015.0.0.1) – suppressed architecture mismatch warning
- [2015.0.0.2](https://github.com/jeremytammik/GetCentroid/releases/tag/2015.0.0.2) – eliminated obsolete API usage
- [2016.0.0.0](https://github.com/jeremytammik/GetCentroid/releases/tag/2016.0.0.0) – flat migration to Revit 2016
- [2016.0.0.1](https://github.com/jeremytammik/GetCentroid/releases/tag/2016.0.0.1) – set copy local false on Revit API assemblies

You can use the GitHub diff functionality to see clearly and exactly what changed from version to version, e.g.
[...GetCentroid/compare/2015.0.0.0...2016.0.0.1](https://github.com/jeremytammik/GetCentroid/compare/2015.0.0.0...2016.0.0.1) to
compare the version 2015.0.0.0 with 2016.0.0.1.

#### Finding the Orientation of Welded Pipe Outlets

A quick summary of the Revit API discussion forum thread on
[finding the orientation of welded pipe outlets](http://forums.autodesk.com/t5/revit-api/finding-the-orientation-of-welded-outlets-of-a-pipe/m-p/5688668):

**Question:**
I have a pipe with several grooved or welded branches and outlets:

![Pipe with welded outlets](img/welded_pipe_outlets.jpeg)

I would like to programmatically determine the orientation of the outlets.

Please can anyone suggest me how to achieve this?

**Answer:**
Each of the branches or outlets is connected to the duct.

Navigate to the corresponding Connection object, e.g. via the FamilyElement > MEPSystem > ConnectionManager properties.

From that, you can determine the connection orientation. Here are three different articles discussing this topic:

- [Connector orientation (2010)](http://thebuildingcoder.typepad.com/blog/2010/03/connector-orientation.html)
- [Connector direction (2010)](http://thebuildingcoder.typepad.com/blog/2010/11/connector-direction-and-createairhandler.html)
- [Connector orientation (2012)](http://thebuildingcoder.typepad.com/blog/2012/05/connector-orientation.html)

#### How to Set the Change Type for DMU AddTrigger

A summary of another Revit API discussion forum thread, on
[how to use GetChangeType to get change of View3D orientation](http://forums.autodesk.com/t5/revit-api/how-can-i-use-getchangetype-to-get-change-of-view3d-orientation/m-p/5687401):

**Question:**
I am using the [Dynamic Model Update framework DMU](http://thebuildingcoder.typepad.com/blog/about-the-author.html#5.31) to create a stereo view and want to update the right and left eye views affected by a modification of the source view orientation.

However I have a problem catching 'Orientation modified'.

I want to use this line of code:

```
  UpdaterRegistry.AddTrigger(
    m_updaterId, doc, idsToWatch, ??? );
```

`idsToWatch` is a list of element ids specifying the source view.

What should I use for the last argument?

How can I get view orientation modified?

**Answer:**
The last argument to the AddTrigger method is the change type.

The safest change type to specify is the one returned by the Element.GetChangeTypeAny method.

Here are some discussions mentioning the AddTrigger method on The Building Coder:

- [Structural Dynamic Model Update Sample](http://thebuildingcoder.typepad.com/blog/2010/08/structural-dynamic-model-update-sample.html)
- [Access Deleted Element](http://thebuildingcoder.typepad.com/blog/2010/10/access-deleted-element.html)
- [Lock the Model, e.g. Prevent Deletion](http://thebuildingcoder.typepad.com/blog/2011/11/lock-the-model-eg-prevent-deletion.html)
- [DocumentChanged versus Dynamic Model Updater](http://thebuildingcoder.typepad.com/blog/2012/06/documentchanged-versus-dynamic-model-updater.html)
- [How to Trigger a Dynamic Model Updater by Specific Element Ids](http://thebuildingcoder.typepad.com/blog/2014/07/createlinkreference-sample-code.html#3)

I hope this helps.