---
ai_optimized: true
author: Jeremy Tammik
complexity_score: 1.6
content_type: news
optimization_date: '2025-12-11T11:44:13.822219'
original_url: https://thebuildingcoder.typepad.com/blog/0365_solar_radiation.html
post_number: '0365'
reading_time_minutes: 1
series: general
slug: solar_radiation
source_file: 0365_solar_radiation.htm
tags:
- revit-api
- views
title: Solar Radiation Technology Preview
word_count: 178
---

### Solar Radiation Technology Preview

The
[solar radiation technology preview](http://labs.autodesk.com/utilities/ecotect) for Revit 2011
is now available.
It helps understand and quantify solar radiation on the surfaces of a conceptual building model.
This can help make informed design decisions about building shape, orientation, and shading strategies early on when changes are least expensive.

A new version was just released, including many improvements and closer inntegrated in the Revit user interface:

![Solar radiation technology preview](img/solar_radiation1.png)

One aspect of interest to application developers is that this plug-in is written entirely using the public Revit API, in addition to the Solar Radiation logic from Ecotect wrapped in a library that is also used from the plug-in code.
While there is no direct API access to call the functionality programmatically, it does showcase the new Revit 2011 APIs:

- Dynamic Update API- Analysis Visualization Framework- [Idling Event](http://thebuildingcoder.typepad.com/blog/2010/04/idling-event.html)- Sun Path API

We will be discussing these in greater depth anon.

![Solar radiation](img/solar_radiation2.png)