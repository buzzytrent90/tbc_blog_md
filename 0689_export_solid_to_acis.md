---
post_number: "0689"
title: "Export Solid to ACIS"
slug: "export_solid_to_acis"
author: "Jeremy Tammik"
tags: ['elements', 'revit-api', 'schedules', 'transactions', 'views']
source_file: "0689_export_solid_to_acis.htm"
original_url: "https://thebuildingcoder.typepad.com/blog/0689_export_solid_to_acis.html"
---

﻿

### Export Solid to ACIS

Today we are en route to Tel Aviv in Israel after the successful conference held in Moscow yesterday.
In Tel Aviv I even have a personal activity lined up, besides the official reason for going there to hold another conference on Wednesday: a
[5Rhythms](http://en.wikipedia.org/wiki/5Rhythms)
[City Wave](http://www.5rhythms-rivi.co.il/Preview.asp?Page=7594&WebsiteID=3751) is
offered very Tuesday evening by Rivi Diamond and fits my schedule perfectly.
That will provide another wonderful change from airplanes, taxis, hotels, and conferences.

Meanwhile, here is a sweet little question on exporting a Revit solid to ACIS raised yesterday by Ishwar Nagwani and answered by Miroslav Schonauer and Emmanuel Weyermann:

**Question:** Is there any way to programmatically create an ACIS file from an individual solid?

For example, AutoCAD ObjectARX provides the method AcDbBody::acisout to achieve this.

As far as I can tell, the Revit API only supports exporting entire views to ACIS.

**Answer:** You can programmatically create a 3D view, loop through all the visible elements in the view to set the individual element visibility ON for your desired solid element and OFF for all others, and export the view to the SAT file format.

The whole operation can be encapsulated in a transaction that is never committed and aborted afterwards so the database is not affected by this operation.

I can confirm that this works, because we have actually used this exact strategy ourselves.

Many thanks to Ishwar, Miro and Emmanuel for raising and resolving this issue!