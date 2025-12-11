---
ai_optimized: true
author: Jeremy Tammik
complexity_score: 1.5
content_type: news
optimization_date: '2025-12-11T11:44:15.324063'
original_url: https://thebuildingcoder.typepad.com/blog/1110_tech_summit.html
post_number: '1110'
reading_time_minutes: 4
series: general
slug: tech_summit
source_file: 1110_tech_summit.htm
tags:
- doors
- elements
- parameters
- revit-api
- rooms
- schedules
- views
- walls
- windows
title: Back from Desert and Two Happy Events
word_count: 861
---

### Back from Desert and Two Happy Events

I returned from my hike near
[Zagora](http://en.wikipedia.org/wiki/Zagora,_Morocco) in
the Sahara desert in Morocco.

I was welcomed back by two happy news items, or 双 喜 临 门, as the Chinese might say; "two happy events came to my door": during my absence, my presentation proposal for the internal Autodesk Technical Summit 2014 on a more generic cloud-based Revit BIM editor was accepted, and my first grandchild was born:

- [A Generic Cloud-based Round-trip Real-time 2D Revit BIM Editor](#3)
- [I'm a grandpa](#4)

#### Desert Impressions

Before getting to those, here are some quick impressions of the beauty to be encountered in the desert:

![Desert dune](file:////j/photo/jeremy/2014/2014-03-01_marocco/613.jpg)

I spent every night outside under the wonderfully clear sky, learned to recognise several new (to me) star constellations, and had a view of the waxing moon crescent unlike any I ever previously saw:

![Waxing moon crescent](file:////j/photo/jeremy/2014/2014-03-01_marocco/595.jpg)

This was taken with a completely normal run-of-the mill camera.
I was even able to zoom in with it far enough to recognise some moon surface details in the shaded part of the orb:

![Waxing moon crescent surface](file:////j/photo/jeremy/2014/2014-03-01_marocco/598.jpg)

After the ten-day desert hike, we spent a few days in the
[Atlas Mountains](http://en.wikipedia.org/wiki/Atlas_Mountains).

One of the many friendly people we met was the Koran teacher in a village that will very soon be inundated due to a new water dam:

![Village teacher](file:////j/photo/jeremy/2014/2014-03-01_marocco/769.jpg)

#### A Generic Cloud-based Round-trip Real-time 2D Revit BIM Editor

If this title reminds you of something, you are perfectly correct: the topic of my presentation for the
[Technical Summit 2013](http://thebuildingcoder.typepad.com/blog/2013/03/cloud-mobile-extensible-storage-data-use-in-schedules.html#3) was
cloud-based round-trip 2D Revit model editing on any mobile device using server-side scripting, CouchDB and SVG.

My new proposal is a more generic reimplementation closer targeting real-world application needs:

#### Proposal Abstract

A complete generic cloud-based graphical and non-graphical 2D Revit BIM editor. It stores 2D plan view graphics and non-graphical data from a Revit building information model (BIM) in a NoSQL cloud database, implements a fully generic and completely portable editor for both graphical and non-graphical BIM information on any mobile device and supports full round-trip editing with real-time updates from the mobile device through the cloud database to the BIM. This includes the implementation of a framework for extracting and managing arbitrary data from BIM element properties and parameters, storing it in JSON format in a NoSQL cloud database, displaying it to the user in a flexible manner, providing editing facilities, updating the cloud database from modifications on the mobile device and synchronising the BIM model with the cloud database in real-time. This is a continuation of my TS 2013 project, a cloud-based round-trip real-time of a simplified 2D view with no support for non-graphical metadata. My TS 2014 proposal is to reimplement the whole application from scratch, taking additional real-world requirements from external application developers into account to implement a powerful, flexible, generic tool that will be reused by external developers and significantly expand, enhance and simplify their work to grow the cloud-based Autodesk ecosystem.

Here is a suggested workflow that I may or may not be able to implement in time:

1. In Revit, select the plan views and specific categories to export, e.g., walls, furniture, etc.
2. Export both non-graphical data, e.g. properties and parameters, plus graphical data to 2D SVG.
3. Import the graphical and non-graphical data into a NoSQL cloud database using the Revit unique id as a common key.
4. Display the 2D plans in a graphical viewer in a web browser.
5. Implement picking an element in the viewer, selecting it and displaying a modal window with non-graphical data.
6. The non-graphical data can be edited and the selected element location modified by dragging, updating the cloud database.
7. Update the Revit BIM model from the cloud database, including both graphical and non-graphical data.

This has a the following important advantages over the current simplified 2D room editor:

- There is a real use case and strong need for such functionality; almost any application developer could make use of such a component in one, several, or all typical workflows.
- Non-graphical data could be handled in a generic, customisable, flexible manner, in addition to the current graphical location and rotation functionality.

Wish me luck getting all of this up and running in time.

I look forward to sharing my progress with you.

#### I'm a Grandpa

The second happy item of news is a new little Tammik, my first grandchild, Nora Sophie, daughter of my eldest daughter Lina, 29:

![Nora Sophie](file:////j/photo/jeremy/kids/nora/20140301_113834_cropped.jpg)