---
post_number: "0861"
title: "Survey and Project Base Point"
slug: "survey_base_pnt"
author: "Jeremy Tammik"
tags: ['elements', 'family', 'filtering', 'parameters', 'references', 'revit-api', 'views']
source_file: "0861_survey_base_pnt.htm"
original_url: "https://thebuildingcoder.typepad.com/blog/0861_survey_base_pnt.html"
---

### Survey and Project Base Point

Happy
[Thanksgiving](http://en.wikipedia.org/wiki/Thanksgiving)!

I am on the road to Las Vegas now, gearing up for last-minute AU preparations.
No actually, I am pretty much set to go.
Other upcoming developer conferences still require some work, though.

Meanwhile, here is some useful information on survey and project base points gathered by my colleague Joe Ye:

**Question:** We need to work with the project base point and the survey point to design platforms in real-world coordinate systems and load survey data from DXF and DWG files containing geo-referenced coordinates.
We therefore wish to use the Revit API to work with the project base point and the survey point.

Could you please explain how to programmatically achieve the following?

- Get and set the location of the project base point and the survey point.- Get and set their parameters (shared coordinates).- Lock and unlock these points.

Additionally, here are a few further questions to improve our understanding of this topic:

1. In Revit we are limited to a distance of 10 kilometres.
   Does this mean that it is necessary to move the survey point together with the shared coordinates of real coordinate systems?
   I successfully placed a platform into a project 25.000 kilometres distant from the project base point (0,0,0) and the survey point (0,0,0).- What is the relation between the shared site (the parameters of the point) and the location of the survey / project base point?- Am I right in my assumption that the survey point changes it's parameters (shared site) when moved depending on whether it is "locked" or not?

**Answer:** The Revit API for shared coordinates is currently still a bit limited.
Here is some of the available functionality:

- Document.ProjectLocations will return all of the ProjectLocations (sites) in a project.
  - These provide Name and ProjectPosition properties that can be used to change the name or position of the site.- The base point and survey point can be retrieved through a FilteredElementCollector for BasePoints.
    - The BasePoint whose IsShared property is true represents the survey point.- Each BasePoint has a Location property that you can set to move the point.- The clipped/unclipped state of the base and survey points cannot be set via the API.
          You can pin them using the Element.Pinned property.- You cannot currently access the shared location of a Revit or DWG instance via the API.- You cannot establish a shared-coordinates relationship with a link instance via the API.
        If the link is already using shared coordinates, then moving the survey point will move the link.

To answer your other questions:

1. The distance limit restricts you from moving the coordinate system origin point more than 10k away from its origin.
   It is still permissible to place Elements far away from the coordinate origin.- There isn't necessarily any relationship.
     Sites are locations you define as you like.
     They can be placed anywhere.- Yes.
       If the survey point's clipped, then it is 'attached' to the world, and when you move it, you move the world with it.
       If you want the survey point to represent some location other than (0,0,0), you can unclip it to move it and nothing else.

Many thanks to Joe for digging this up!

#### BIM for Infrastructure White Paper

Talking about survey points and stuff like that, a new white paper has just been released explaining the use and advantages of BIM for infrastructure:
[BIM for Infrastructure: A vehicle for business transformation](zip/BIM_for_Infrastructure_November_2012.pdf) discusses
BIM as the vehicle by which the business of planning, designing, building, and managing the world's infrastructure will be transformed to deliver higher productivity, quality, and cost-effectiveness.

#### ODBC Export

A recurring topic is programmatic ODBC export.

Here is a typical query on this topic with an answer by Saikat Bhattacharya:

**Question:** Is there a way to export the whole Revit database to an external database such as MSsql via the ODBC link, similarly to the manual process or using DBLink, but avoiding all the dialogues for choosing the ODBC connection etc.?
I would like to set up the connection programmatically in .NET instead.

**Answer:** The Revit API provides direct access to the underlying data.
You can implement your own export functionality to any external database including MSsql.
How to implement this is completely up to you to decide.

The API provides access to the data in the model that might need to be included in the export.
With the data available, the connection can be set up and desired data exported to any external database.

The Revit 2010 SDK still included the RDBLink sample, so that code is available in case of interest.
Due to its size and complexity, it was later promoted to a subscription feature and removed from the SDK.
If you like, you could migrate the old Revit 2010 code to the current version and use it as a basis for your own implementation.

Here are a couple of discussions related to this topic:

- [RDB Link tool on Autodesk Labs](http://thebuildingcoder.typepad.com/blog/2009/06/rfa-version-grey-commands-family-context-and-rdb-link.html#4).- [Integration with a database or ERP system](http://thebuildingcoder.typepad.com/blog/2009/07/integration-with-a-database-or-erp-system.html).- In-depth analysis of
      [integrating with external databases](http://thebuildingcoder.typepad.com/blog/2009/01/database-integration.html) and other data sources.- [Adding a column to RDBLink export](http://thebuildingcoder.typepad.com/blog/2009/11/adding-a-column-to-rdblink-export.html).- [Exporting parameter data to Excel](http://thebuildingcoder.typepad.com/blog/2012/09/exporting-parameter-data-to-excel.html), and re-importing.

Many thanks to Saikat for putting together this overview!