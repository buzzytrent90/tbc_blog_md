---
ai_optimized: true
author: Jeremy Tammik
complexity_score: 2.5
content_type: tutorial
optimization_date: '2025-12-11T11:44:15.801174'
original_url: https://thebuildingcoder.typepad.com/blog/1338_firerating_cloud.html
post_number: '1338'
reading_time_minutes: 5
series: general
slug: firerating_cloud
source_file: 1338_firerating_cloud.htm
tags:
- csharp
- doors
- elements
- family
- levels
- parameters
- python
- revit-api
- schedules
- sheets
- vbnet
title: FireRating and the Revit Python Shell in the Cloud as Web Servers
word_count: 1006
---

### FireRating and the Revit Python Shell in the Cloud as Web Servers

In the past few days, [The 3D Web Coder](http://the3dwebcoder.typepad.com) documented the first steps of research for a re-implementation of the Revit SDK FireRating sample in the cloud, and Daren Thomas pointed out his new project driving Revit and the Revit Python Shell through a REST API:

- [The FireRating Revit SDK sample](#2)
- [FireRating data structure](#3)
- [FireRating in the cloud](#4)
- [Revit and the Revit Python Shell as a REST API driven web server](#5)

Summer has arrived in Switzerland!
This entire week is expected to be hot and sunny with temperatures up to 36 degrees Celsius.
The Rhine is full of swimmers in the middle of the city of Basle, competing for space with the barges:

![Swimmers and a barge in the Rhine in the middle of Basel](file:////j/photo/jeremy/2015/2015-07-01_basel/619_swimmers_barge.jpg)

#### The FireRating Revit SDK Sample and ADN Xtra Labs

The venerable old FireRating Revit SDK sample has proved extremely useful over the years.

I described it in detail in the discussion on [exporting parameter data to Excel, and re-importing](http://thebuildingcoder.typepad.com/blog/2012/09/exporting-parameter-data-to-excel.html).

It is a VB.NET add-in, implementing the following three steps in three separate external commands:

- Create a shared parameter to hold a fire rating value for all door family instances.
- Export the fire rating for all instances to an Excel spreadsheet.
- Import the (possibly modified) fire rating values back from the selected Excel spreadsheet into the project and update the respective shared parameter values on all instances.

These three also found their way into the
[ADN Xtra labs](http://thebuildingcoder.typepad.com/blog/2015/06/adn-labs-xtra-multi-version-add-ins-and-cnc-direct.html#2),
where they are implemented twice over, in C# as well as VB.NET, in three corresponding external commands:

- Lab4\_3\_1\_CreateAndBindSharedParam
- Lab4\_3\_2\_ExportSharedParamToExcel
- Lab4\_3\_3\_ImportSharedParamFromExcel

#### FireRating Data Structure

The entire FireRating data management consists of storing the following four columns of data in the external spreadsheet, with one row of data populated for each door instance:

- Element id
- Level
- Tag
- Fire rating

The level and tag data are actually never used by the application.

Being rather old and unwary, the sample uses the element id to identify the individual door instances.

That is strongly discouraged, since the element id can be modified by worksharing operations.

When associating external data with an individual Revit element, you should always use its unique id instead.

Furthermore, the element id can only be used to identify elements within one single Revit project, since it is currently more or less just a simple consecutive integer number assigned to reach element in the order of its creation.

The unique id is globally unique though, across all Revit project in the universe – always assuming that you have not made copies of a project, in which case the unique ids of the existing elements will obviously be copied as well.

#### FireRating in the Cloud

I started looking at how to update the FireRating sample to a cloud-based implementation making use of one single global [mongoDB](https://www.mongodb.org) [NoSQL](https://en.wikipedia.org/wiki/NoSQL) database to hold the data for any number of Revit projects.

![mongoDB](img/mongodb.png)

For the sake of compatibility and completeness, I retain the unused level and tag data.

I will obviously replace the element id of the door instances by their unique id; being unique, we can use it across projects.

Here are the explorations I made so far:

- [My First Mongo Database](http://the3dwebcoder.typepad.com/blog/2015/06/my-first-mongo-database.html) – installation, tools, and storing the door instance data.
- [Implementing Mongo Database Relationships](http://the3dwebcoder.typepad.com/blog/2015/07/implementing-mongo-database-relationships.html) – RVT project identification, storing the project data and handling the door-project relationship.

Here are the plans for the next steps, which – actually, hopefully, as usual, as always – will take a lot longer to explain and document than to implement:

- Add a REST API to populate and query the database programmatically.
- Implement a Revit add-in exercising the REST API from C# .NET.
- Re-implement the complete round-trip FireRating Revit SDK sample functionality.

Keep an eye on
[The 3D Web Coder](http://the3dwebcoder.typepad.com) to see how we address these steps.

#### Revit and the Revit Python Shell as a REST API Driven Web Server

Daren Thomas, author of the [Revit Python Shell](https://github.com/architecture-building-systems/revitpythonshell), points out a new and exciting project, providing full web-based REST API access to Revit and the Python Shell:

> I finally finished an article that I spent a lot of time working on, [embedding a web server in Revit](http://darenatwork.blogspot.ch/2015/06/embedding-webserver-in-autodesk-revit_30.html) – the initial idea was born two years ago!
>
> This post (including full code on GitHub) explains how to turn Revit into a web server with a REST interface using the RevitPythonShell!
>
> I used this technique in two projects already and am quite pleased with the results – two papers I wrote are based on the ability to access BIM data from a scientific workflow tool (like
> [Kepler](https://en.wikipedia.org/wiki/Kepler_scientific_workflow_system) or
> [VisTrails](https://en.wikipedia.org/wiki/VisTrails)), and having a REST API to drive the functionality makes this a snap!
>
> I used your idea for
> [exporting a schedule to CSV](http://thebuildingcoder.typepad.com/blog/2012/05/the-schedule-api-and-access-to-schedule-data.html) as an example to discuss the technique.
>
> Also, this can very easily be implemented in C# too, as it uses .NET classes for all the heavy lifting.

Read [the article](http://darenatwork.blogspot.ch/2015/06/embedding-webserver-in-autodesk-revit_30.html) for the full story and all the links.

Thank you very much, Daren, for this exciting and powerful project!