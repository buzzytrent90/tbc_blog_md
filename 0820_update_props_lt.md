---
ai_optimized: true
author: Jeremy Tammik
complexity_score: 1.9
content_type: tutorial
optimization_date: '2025-12-11T11:44:14.661908'
original_url: https://thebuildingcoder.typepad.com/blog/0820_update_props_lt.html
post_number: 0820
reading_time_minutes: 3
series: general
slug: update_props_lt
source_file: 0820_update_props_lt.htm
tags:
- elements
- family
- revit-api
- schedules
- windows
title: Updating Properties and Announcing Revit LT
word_count: 641
---

﻿

### Updating Properties and Announcing Revit LT

Today, exceptionally, a second post, partly to highlight an interesting
[comment](http://thebuildingcoder.typepad.com/blog/2010/06/highlight-elements.html?cid=6a00e553e168978833017744869c64970d#comment-6a00e553e168978833017744869c64970d) submitted by
Jinshou on how to programmatically trigger a
[refresh of the Revit element properties window](#2),
and partly to announce a non-API issue, totally non-API, the imminent arrival of
[Autodesk Revit LT](#3).

#### Refreshing Revit Element Properties

If I first invoke the UIApplication OpenAndActivateDocument method, then
[highlight elements](http://thebuildingcoder.typepad.com/blog/2010/06/highlight-elements.html) as
described in a previous post, the 'properties' window is updated.

The trick is that in order to call OpenAndActivateDocument on the document containing the elements to highlight, you have to first ensure that it is not the active document.
To achieve this, I:

- Create an empty RVT file.- Invoke OpenAndActivateDocument to activate that.- Invoke OpenAndActivateDocument again on the document of interest.- [Highlight the elements](http://thebuildingcoder.typepad.com/blog/2010/06/highlight-elements.html).- Don't forget to close the empty RVT file when finished.

Many thanks to Jinshou for this interesting hint.

#### Announcing Autodesk Revit LT

For the nonce, here is an announcement completely unrelated to the Revit API.
In fact, the product announced is specifically designed to not be customisable.

Autodesk today announced Autodesk Revit LT software, a new addition to its award winning family of Revit software products.
Designed for architects, designers and building industry professionals using 2D workflows who are looking to transition to the Building Information Modeling (BIM) process, Autodesk Revit LT software delivers a focused 3D application to help users create higher-quality and more accurate designs and documentation.

"Autodesk Revit LT reflects our commitment to provide architects, designers and building professionals, particularly those in small firms, the tools they need to ease the transition to get started with – and take advantage of – a BIM workflow," said Amar Hanspal, Autodesk senior vice president, information modeling products.
"With Revit LT, customers can deliver higher-quality, more accurate designs and documentation – allowing them to bring projects to market faster and save time and money."

Autodesk Revit LT is built on the Revit platform for BIM and allows users to create designs efficiently with 3D, real-world building objects to produce reliable, coordinated documentation faster.
Revit-based applications help deliver better coordination and quality, and can contribute to higher profitability for architects, design professionals and the rest of the building team.
Some of the benefits of Revit LT include:

- Work more efficiently with a single, coordinated model that allows users to concurrently design and document building projects. Autodesk Revit LT automatically manages iterative changes to building models throughout the documentation process. As a result, a consistent representation of the building is maintained, helping to improve drawing coordination and reducing errors.- Design and visualize in 3D. Revit LT allows users to see their designs virtually, improving their understanding of the building and its spaces, and helping them communicate design ideas to clients more clearly and effectively.- Create photorealistic renderings in the cloud. Users who purchase Autodesk Subscription with Revit LT can render in the cloud directly from the Revit LT interface, enabling them to produce compelling, photorealistic visualizations without tying up their desktop- Exchange designs in the DWG or RVT file formats. Produce designs in the DWG file format, and experience fluid file exchange with project team members using other Autodesk Revit software applications.

##### Availability

Autodesk Revit LT 2013 is scheduled to be available within the month in North America and in select countries worldwide.
The product is available as a standalone version and as part of the new AutoCAD Revit LT Suite 2013, which includes both Autodesk Revit LT 2013 and AutoCAD LT 2013.
Product availability varies by country.
Details and purchasing options are available at
[www.autodesk.com/revitlt](http://www.autodesk.com/revitlt)