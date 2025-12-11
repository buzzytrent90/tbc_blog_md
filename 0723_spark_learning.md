---
ai_optimized: true
author: Jeremy Tammik
complexity_score: 1.9
content_type: tutorial
optimization_date: '2025-12-11T11:44:14.465452'
original_url: https://thebuildingcoder.typepad.com/blog/0723_spark_learning.html
post_number: '0723'
reading_time_minutes: 4
series: general
slug: spark_learning
source_file: 0723_spark_learning.htm
tags:
- elements
- geometry
- levels
- revit-api
- sheets
- views
- walls
title: BIM versus Free Geometry and Product Training
word_count: 819
---

### BIM versus Free Geometry and Product Training

Before even thinking about developing an add-in for Revit, i.e. before making any use of the Revit API, you first need to ensure that you have a good understanding of the Revit product and its philosophy.

This is especially important for developers with experience with other CAD products, especially AutoCAD.

#### Programming for a Parametric BIM versus Freely Defined Geometry

In AutoCAD, as a developer, you have complete freedom to mess around with the database and its elements in any way you please.

In Revit, almost the opposite is true.
Because all elements are completely parametrically driven and everything is tightly linked to everything else, you have almost no freedom whatsoever.
In particular, the way the system works and objects are added and linked to each other is completely predetermined.

For instance, consider how the height or thickness of a wall is determined.
In some CAD systems, you might simply go to the wall object, or even to some underlying geometric entities such as a polyline, face, or solid body, and query its length.
It might even be possible to change the wall geometry by modifying the length of some geometric entity.

In Revit, the thickness of a wall is determined by its wall type, so you cannot modify it on the wall at all without changing its wall type.
This also means that it is not possible to naively apply a scaling operation to a Revit element, since e.g. scaling a wall might change its width, which cannot be permitted, since that might conflict with the width determined by its wall type.

Similarly, in Revit, the height of a wall is determined by the level it lives on, determining its bottom extent, and its top bounding level.
If these levels do not exactly match the height required by the wall, you can define additional bottom and top offsets.
So to determine the wall height, you have to go off and determine the bottom and top levels elevation and consider the two optional offsets.
And thus obviously changing the wall height is not a matter of simply modifying some property on the wall itself, let alone its geometry, but needs to take its relationships with and dependencies on other elements into account.

This is such a radically different approach than simply grabbing the wall geometry that it takes some getting used to, and cannot be repeated too often to Revit API beginners coming with experience from other non-parametric non-BIM CAD systems.

So much for a quick intro to free geometry versus parametric BIM.

Obviously, the BIM paradigm and its effect on the API goes much further than this, and I am not best suited to explain it all.

The underlying important point is that the Revit product itself has incredibly efficient and powerful functionality that needs to be understood in depth before you start trying to enhance it by providing additional features through your add-in.

As in all programming areas, and especially when coming with experience from other CAD systems, there is vast freedom to shoot yourself in the foot.

So, in short, make sure you really understand the product, its functionality, and the optimal workflow in depth before starting to think of making any attempts to enhance it.

#### Product Training Material

Here are some product learning materials which were recently pointed out to me.

The
[Getting Started](http://wikihelp.autodesk.com/Spark/enu/TP_1.0/Help/0000-Getting_0) area
of the
[Project Spark](http://thebuildingcoder.typepad.com/blog/2011/09/autodesk-lab-projects-spark-and-storm.html) wikihelp
includes
a very useful set of tutorials.
These Getting Started videos with matching step-by-step instruction and datasets also work for Revit.

Here are links to further training materials from the product support and product teams:

The last
[Revit Architecture tutorials](http://usa.autodesk.com/adsk/servlet/item?siteID=123112&id=13080067) were
produced for the 2010 release.
For 2011 and later, the help team switched to creating video tutorials and no longer maintain the step by step ones.

Another highly recommended place to learn about Revit is the
[BIM Workshop](http://bimcurriculum.autodesk.com).
It also presents video tutorials, and they are very well structured.

Furthermore, the Autodesk site provides an
[overview of Revit Architecture tutorials](http://usa.autodesk.com/adsk/servlet/autoindex?siteID=123112&id=3640745&linkID=9243097) for
different topics such as
[working with title blocks and sheets](http://usa.autodesk.com/adsk/servlet/item?siteID=123112&id=13945669&linkID=9243097).

The
[Revit Families Guide](http://usa.autodesk.com/adsk/servlet/index?siteID=123112&id=13080413&linkID=9243097) is
very helpful and important and a good starting point to understand Revit families in depth.

Finally, we have the
[Revit Clinic product blog](http://revitclinic.typepad.com) providing
lots of real-world tips and tricks.

That should be ample to get you started.
Have fun!