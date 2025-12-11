---
ai_optimized: true
author: Jeremy Tammik
complexity_score: 1.7
content_type: tutorial
optimization_date: '2025-12-11T11:44:15.810193'
original_url: https://thebuildingcoder.typepad.com/blog/1342_firerating_cloud.html
post_number: '1342'
reading_time_minutes: 3
series: general
slug: firerating_cloud
source_file: 1342_firerating_cloud.htm
tags:
- csharp
- doors
- geometry
- parameters
- revit-api
- views
- walls
title: Grevit, FireRating in the Cloud Demo, Deployment, Vacation
word_count: 609
---

### Grevit, FireRating in the Cloud Demo, Deployment, Vacation

I have been extremely busy the past few days implementing my
[FireRating in the Cloud](https://github.com/jeremytammik/FireRatingCloud) sample,
a migration of the standard Revit SDK FireRating sample to a cloud-based multi-project implementation – reflected in this week's GitHub contributions:

![GitHub contributions](img/2015-07_github_contribution.png)

Also, I heard from Max Thumfart about his very interesting Grevit project:

- [FireRating in the cloud demo and deployment](#2)
- [Grevit](#3)
- [Vacation time soon](#4)

#### FireRating in the Cloud Demo and Deployment

I'm just about done with my FireRating in the Cloud project.

I published a
[demo run log](http://the3dwebcoder.typepad.com/blog/2015/07/fireratingcloud-round-trip-and-on-mongolab.html#4) yesterday
showing the details of the four workflow steps:

- Create and bind a shared parameter
- Export door instance fire rating data from Revit
- Modify the values externally
- Import updates back into the BIM

This is followed by an
[82-second video](http://the3dwebcoder.typepad.com/blog/2015/07/fireratingcloud-round-trip-and-on-mongolab.html#5)
showing the addition of a few more doors and the full round trip data flow live:

After completing that initial running both the mongo database and the node.js web server locally, I continued to implement and test the real live
[fully deployed cloud version of FireRatingCloud](http://the3dwebcoder.typepad.com/blog/2015/07/fireratingcloud-fully-deployed-on-heroku-and-mongolab.html) with
the node.js web server hosted on
[Heroku](http://www.herokuapp.com) and the database on
[mongolab.com](https://mongolab.com).

The
[fireratingdb node.js mongo database web server](https://github.com/jeremytammik/firerating) and
[FireRatingCloud Revit add-in](https://github.com/jeremytammik/FireRatingCloud) GitHub
repositories provide an overview of the complete project analysis, exploration and implementation.

#### Grevit

Everybody is excited about Grexit... I find Grevit far more interesting!

Max Thumfart, Senior Engineer at [Thornton Tomasetti](http://www.thorntontomasetti.com) in the UK, kindly pointed me to
[Grevit](http://grevit.net), a
[Rhino](http://www.rhino3d.com) +
[Grasshopper](http://www.grasshopper3d.com) app
that enables you to assemble a BIM in Grasshopper, send it to Revit or AutoCAD Architecture, and dynamically update existing models with geometry changes:

- [Grasshopper project](http://www.food4rhino.com/project/grevit)
- [Full documentation](http://grevit.net)

![Grevit](img/grevit.png)

Grevit is now open source and lives in its own [Grevit GitHub repository](https://github.com/moethu/Grevit).

It currently drives Revit 2015 with support for Revit 2016 coming soon, and is used by numerous architects, engineers and Revit consultants to transfer geometry to Revit,
including Arup, Henn, LAVA, Pilbrow & Partners, GehryTech, Pattern Architects, SOM, A3D in Spain, etc.

Max also started to blog about Revit API interoperability on the
[Grevit blog](https://grevit.wordpress.com), e.g., on how to
[write a C++ wrapper for the Boost library for use in C#](https://grevit.wordpress.com/2015/07/03/boost-c-library-in-c-revit-api) and thus within the Revit API.
As an example, he picks the use of the [Boost Voronoi diagram implementation](http://www.boost.org/doc/libs/1_54_0/libs/polygon/doc/voronoi_main.htm) to
drive a BIM and create Model Lines showing optimal separation of Revit walls, columns and slabs.

Congratulations to Max on these super cool projects, and many thanks for pointing them out!

#### Vacation Time Soon

That's it from me for this week.

That's almost it from me for quite a while, actually.

I'll be going on holiday quite a lot in the next couple of weeks.

I will say bye-bye properly before I leave for good sometime next week, though.