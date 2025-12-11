---
ai_optimized: true
author: Jeremy Tammik
complexity_score: 2.4
content_type: qa
optimization_date: '2025-12-11T11:44:14.757191'
original_url: https://thebuildingcoder.typepad.com/blog/0864_view_mep_link_api.html
post_number: 0864
reading_time_minutes: 3
series: views
slug: view_mep_link_api
source_file: 0864_view_mep_link_api.htm
tags:
- parameters
- references
- revit-api
- rooms
- schedules
- views
title: AU Classes on the View, MEP and Link APIs
word_count: 521
---

### AU Classes on the View, MEP and Link APIs

Wednesday was my first normal and full AU day, after completing the developer conference and DevLab activities.
I was able to attend two interesting classes held by Revit API developers Steven Mycynek and Arnošt Löbel, and present my own Revit MEP API one.
Here they are in chronological order:

- [CP3133](https://www.autodeskuniversity2012.com/connect/sessionDetail.ww?SESSION_ID=3133)
  Using the Revit [Schedule and View APIs](#2) by Steven Mycynek- [CP4108](https://www.autodeskuniversity2012.com/connect/sessionDetail.ww?SESSION_ID=4108)
    [Revit MEP Programming](#3): All Systems Go by Jeremy Tammik- [CP3455](https://www.autodeskuniversity2012.com/connect/sessionDetail.ww?SESSION_ID=3455)
      [Managing Revit Links](#4) with the Revit API by Arnošt Löbel

I am including the class materials below, both for your and my own convenience, and also to ensure that all this juicy stuff really is picked up and returned by Internet search engines.

#### Schedule and View APIs

Revit 2013 includes a major expansion of the Schedule and View APIs.
This class introduces the API for Schedule and Elevation views as well as many methods and properties that apply to all views.
Learn how to create and modify these views to help document and display your Revit model.

- Handout
- Presentation
- Sample code
- Sample models

This class should really resolve all those recurring questions associated with the creation of different views once and for all!
Take a special deep look at Steve's multitude of useful utility classes in the sample code.

#### Revit MEP Programming

This class shows how to work programmatically with Revit MEP models.
It provides an overview of the entire Revit MEP API with specific focus on the major enhancements in Revit 2013.
The most important 2013 feature is the routing preference functionality.
An overview of the available samples is provided.
All MEP domains including HVAC, electrical, and plumbing are addressed.
It shows how to analyse existing systems and create new MEP models from scratch; covers mechanical and electrical system traversal; display of system hierarchies in a tree view; components such as circuits, ducts, pipes, fittings, connectors, cable trays, and conduits; and automatic calculation and sizing based on room and space requirements.

- Handout
- Presentation
- Sample code

#### Managing Revit Links

This class explains common pitfalls working with links and how to avoid them.
It details the creation of Revit link types and instances, and recommends how to easily check or modify the parameters of links.
Material describing ways to inquire for data about the various types of external files, including Revit links, is included.
It concludes with an overview of a larger Revit link API application – the eTransmit add-in, which enables users to move an entire package of Revit files while maintaining relationships with all linked files.

- Presentation
- Sample code and models

#### Implementing a BIM Business Transformation

Unrelated to any direct programming topic, here is a white paper on
[implementing a BIM business transformation](zip/Implementing_a_BIM_Business_Transformation.pdf) that
is of interest to visionaries and decision makers and neatly complements the recently released
[BIM for Infrastructure](http://thebuildingcoder.typepad.com/blog/2012/11/survey-and-project-base-point.html#2) one.