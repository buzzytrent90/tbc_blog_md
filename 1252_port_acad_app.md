---
post_number: "1252"
title: "Porting an AutoCAD Application to Revit"
slug: "port_acad_app"
author: "Jeremy Tammik"
tags: ['revit-api', 'schedules']
source_file: "1252_port_acad_app.htm"
original_url: "https://thebuildingcoder.typepad.com/blog/1252_port_acad_app.html"
---

﻿

### Porting an AutoCAD Application to Revit

As repeatedly noted, AutoCAD and Revit and their APIs are
[very different animals](http://thebuildingcoder.typepad.com/blog/about-the-author.html#5.41).

Still, the question of porting applications from other CAD systems to Revit does keep popping up, e.g. in this Revit API discussion forum thread on
[importing an AutoCAD application](http://forums.autodesk.com/t5/revit-api/importing-an-autocad-application/m-p/5429829),
so I'll reiterate some aspects of that here once again:

**Question:** We are looking for a bit of updated information in regards to porting an AutoCAD MEP application into Revit MEP. I found a
[discussion on this topic dated back to 2012](http://thebuildingcoder.typepad.com/blog/2012/10/porting-an-autocad-application.html).
Does this information remain relevant today?
We develop and maintain software that matches the criteria that the first question in the link presents.
We create a custom ribbon, create grips and graphic overrules in our software, use MEP objects to store property data and show labels on them and perform extractions to various file types and interfaces.

Are any of these items possible using the Revit API?

**Answer:**

Everything said in the blog post back then remains equally valid today.

Here are some more discussions on the topic of
[Revit and its API compared to other CAD systems](http://thebuildingcoder.typepad.com/blog/about-the-author.html#5.41).

Answers to your questions:

1. Forget about *porting*. Think *rewrite* from scratch.
2. Before you do anything else at all, learn and understand the Revit UI, workflows and best practices and talk to experienced users about what Revit can and cannot do. It may do everything you are thinking of programming already, or it may be completely beside the point.
3. Custom ribbon... well, no, not really. No such thing as grips. Graphic overrules... well, yes, sort of, but probably nowhere near what you are thinking of. Property data: yes, either shared properties or extensible storage. Labels: use tags, cf. user interface. Perform extractions: cf. schedules or custom add-ins.

Yes, everything is possible.

No, porting probably makes no sense whatsoever.

Yes, rewriting with all your experience will certainly be fun and interesting and rewarding.

To start looking at the API, take a few hours to work through the
[getting started material](http://thebuildingcoder.typepad.com/blog/about-the-author.html#2),
especially the *My first Revit plugin* and *DevTV* tutorials.

Now, after that quick excursion into getting started with the Revit API and porting applications from other CAD systems, back to the excitement here at Autodesk University in Las Vegas...