---
ai_optimized: true
author: Jeremy Tammik
complexity_score: 2.4
content_type: qa
optimization_date: '2025-12-11T11:44:15.270262'
original_url: https://thebuildingcoder.typepad.com/blog/1082_au_bpa_daylighting.html
post_number: '1082'
reading_time_minutes: 5
series: general
slug: au_bpa_daylighting
source_file: 1082_au_bpa_daylighting.htm
tags:
- elements
- levels
- parameters
- revit-api
- rooms
- views
title: Starting to Clean Up For the Break
word_count: 1031
---

### Starting to Clean Up For the Break

I am starting to clear up some open issues before the break.
Here are a few that I can handle right here and now:

- [Autodesk University classes ready for viewing and download](#2)
- [Building Performance Analysis update](#3)
- [Daylighting Analysis Tool for Revit update](#4)

I will obviously not be able to magically transform the entire backlog of interesting topics I still wish to share and discuss.

Still, I am continuously learning to be active and still take things easy, do what I do, not think too much, just act.

Here is a nice little anecdote to illustrate the principle:

> A new Zen monk has just been admitted to the monastery.
>
> He pays his respects to the master and says: "I'm new to the monastery, please show me the way."
>
> The master asks, "Have you had breakfast?"
>
> The novice replies, "Yes, just a moment ago."
>
> "Then go and wash your eating bowl."

By the way, I wondered whether to use commas or colons in the lines above, and found a conclusive answer in the Grammar Monster discussion of
[quotation marks for quotes](http://www.grammar-monster.com/lessons/quotation_%28speech%29_marks_colon_or_comma.htm):
"use commas for quotations that comprise fewer than 7 words and use colons for longer quotations".
I did not know that before.
Thank you, Grammar Monster.

#### Autodesk University Classes Ready for Viewing and Download

The AU class materials and recordings are now available for viewing and download from
[au.autodesk.com](http://au.autodesk.com).

The search tool returns this year's classes as well as previous ones from 2011 and 2012.

Here are the results of
[searching for my classes](http://au.autodesk.com/au-online/classes-on-demand/search?full-text=tammik):

- 2011 – CP4451 : Extensible Storage in the Revit 2012 API
- 2011 – CP6760-L : Revit 2012 API Extensible Storage Lab
- 2011 – CP4453 : Everything in Place with Revit MEP Programming
- 2012 – CP4107 : Let's Face It: New Revit 2013 User Interface API Functionality
- 2012 – CP4108 : Revit MEP Programming: All Systems Go
- 2012 – CP4109 : Revit API Roundtable: Meet the Champions
- 2013 – DV2010 : Advanced Revit 2014 API Features and Samples
- 2013 – DV1736 : Cloud-Based, Real-Time, Round-Trip, 2D Revit Model Editing on Any Mobile Device
- 2013 – DV1914 : Revit API Expert Roundtable: Open House on the Factory Floor

By the way, talking about AU, and what with the festive season and all, here is a picture of me at AU with a pretty big and festive bird:

![The big festive peacock](file:///j/photo/jeremy/2013/2013-12-03_las_vegas_venetian/jeremy_peacock2.jpeg)

#### Building Performance Analysis Update

I
[mentioned](http://thebuildingcoder.typepad.com/blog/2012/11/building-performance-analysis-and-face-tessellation.html#2) the
Building Performance Analysis blog at
[autodesk.typepad.com/bpa](http://autodesk.typepad.com/bpa) last
year.

The BPA team have completed a number of exciting projects since then.

Thanks to them, Revit 2014 now offers an entirely new way to create an Energy Analysis Model or EAM automatically from Revit building elements, as explained in their following discussions:

- [Revit 2014 release news](http://autodesk.typepad.com/bpa/2013/03/revit-2014-release-news-new-building-performance-analysis-features.html)
- [Energy analysis using Revit building elements](http://autodesk.typepad.com/bpa/2013/04/revit-2014-now-available-for-download-energy-analysis-using-revit-building-elements.html)
- [Revit 2014 update release 1](http://autodesk.typepad.com/bpa/2013/07/revit-2014-update-release-1-now-available-for-download-automatic-energy-analytical-model-creation-from-revit-building-elemen.html)

If you are already working with the Revit Energy Analysis model or are interested in determination of realistic volumes of rooms and spaces for any purpose whatsoever, this is a topic you absolutely must dive into.

For starters, read the detailed explanation
[From BIM to BPA: What is an Energy Analysis Model (EAM)?](http://autodesk.typepad.com/bpa/2013/12/from-bim-to-bpa-what-is-an-energy-analysis-model-eam.html)

Expect more in this area!

#### Daylighting Analysis Tool for Revit Update

One of the BPA tools is the recently announced
[Daylighting Analysis Tool for Revit](https://beta.autodesk.com/callout/?callid=976EB2E05A1D46B5818473BDF2FAABC5),
which was just now updated and released for global distribution.

The Revit 2014 Daylighting Analysis plug-in uses the Autodesk 360 Rendering cloud service to perform very fast and physically accurate daylighting analyses from within Revit.
In the current first release of the plug-in, it provides LEED IEQc8.1 2009 results for most models in less than 15 minutes.

Here is the detailed BPA presentation of
[daylighting as a service](http://autodesk.typepad.com/bpa/2013/04/daylighting-as-a-service-raas-illuminance-radiance.html).

The features provided by the new update include:

- New localized EULA and installer allow global distribution.
- If no Rooms are defined in the model when Run Analysis is selected, the user is given a warning, but analysis will continue.
- A unit option has been added to the AVF results view so lux or foot-candles can be selected.
- The LEED Room parameters now show up as editable in the Room Properties panel when a Room is selected.
- The AVF legend has been improved by clipping the values shown in the AVF to 6000 lux to make the scale visible in the lower ranges. The full raw values are still stored in the Revit Extensible Storage for export.
- User can hide and unhide AVF grids by level to make viewing and navigation easier in 3d section cuts.
- If the user has modified the "LEED 2009…" 3d view, the view is reset to defaults for the analysis run. Best practice is for user to make a copy of that view to create a custom view style.
- The plug-in can now handle multiple projects and multiple concurrent analyses in Revit. It is still required to keep a project open until the analysis is complete and data is downloaded from the cloud service.
- The user will now receive a message if a job fails for any reason, and the successful results will be downloaded.

You can download the technology preview from Autodesk Labs:
[Download Daylighting Analysis Tool for Revit](https://beta.autodesk.com/callout/?callid=976EB2E05A1D46B5818473BDF2FAABC5).