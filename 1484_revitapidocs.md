---
ai_optimized: true
author: Jeremy Tammik
complexity_score: 2.5
content_type: qa
optimization_date: '2025-12-11T11:44:16.132178'
original_url: https://thebuildingcoder.typepad.com/blog/1484_revitapidocs.html
post_number: '1484'
reading_time_minutes: 3
series: general
slug: revitapidocs
source_file: 1484_revitapidocs.md
tags:
- filtering
- revit-api
- sheets
- views
title: The Building Coder
word_count: 552
---

### Token Expiry and Online Revit API Docs
My vacation ended and I am now in Porto, putting the last touches to my presentation material for
the [RTC Revit Technology Conference Europe](http://www.rtcevents.com/rtc2016eur) and
the [ISEPBIM](https://www.facebook.com/ISEPBIM) Forge and BIM workshops at [ISEP](http://www.isep.ipp.pt),
the [Instituto Superior de Engenharia do Porto](http://www.isep.ipp.pt).
![Porto](/p/2016/2016-10-17_porto/634.jpg)
[![Porto](https://c7.staticflickr.com/6/5773/30085212350_47a567224a_n.jpg)](https://www.flickr.com/photos/jeremytammik/albums/72157671812736103 "Porto")
Here are today's Revit API and Forge news items:
- [Updated Online Revit API Docs](#2)
- [Handling Forge token expiry](#3)
#### Updated Online Revit API Docs
Gui Talarico updated the [online Revit API documentation](http://thebuildingcoder.typepad.com/blog/2016/08/online-revit-api-docs-and-convex-hull.html#2)

[www.RevitAPIdocs.com](http://www.revitapidocs.com)
In his own words:
> In case you haven't seen it yet, I just wanted to share with you the latest version of [RevitAPIdocs.com](http://www.revitapidocs.com):
>
- Redesigned home page.
- Consolidated search for all APIs: search results highlight if entry is not part of the active 'Year' and redirect you (i.e., to leaders in Revit 2015).
- Result filtering by type (Class, Method, etc.).
- Built in autocomplete engine to help users find most relevant entries – it will improve further over time!
Here is an animation showing a quick overview of the new features:
![Revit API Docs new features](img/revitapidocs_newfeat.gif)
![Revit API Docs new features](http://thebuildingcoder.typepad.com/revitapidocs_newfeat.gif)
The revamped search result landing page looks like this:
![Revit API Docs search landing page](img/revitapidocs_landing2.gif)
![Revit API Docs search landing page](http://thebuildingcoder.typepad.com/revitapidocs_landing2.gif)
Many thanks to Gui for providing this really important tool!
#### Handling Forge Token Expiry
\*\*Question:\*\* I am working on an app for
the [Forge and AppStore hackathon challenge](http://autodeskforge.devpost.com).
I am using the [models.autodesk.io online tool](https://models.autodesk.io) for token generation.
However, my access token is getting expired.
Please help resolve the issue.
\*\*Answer by Cyrille Fauvel:\*\* [models.autodesk.io](https://models.autodesk.io) is a web site which demonstrates how to easily translate a model using your own consumer and secret key.
It is not intended to be used as an access token generator, because it is highly discouraged to transmit your keys over Internet, even through an encrypted and secured `https` connection.
Instead, you should write your own code to generate the access token yourself, just like `models.autodesk.io` does. The relevant part is located
in [lmv-token.js around line 43](https://github.com/cyrillef/models.autodesk.io/blob/master/server/lmv-token.js#L43).
As you have noticed, the token expires after a while.
This sample does not take care of refreshing it, as this site is to be used manually for a single translation.
In your own implementation, you will need to check the `expires_in` field to know when the token will expire and make sure to refresh it before it does so as shown
in [lmv-token.js around line 29](https://github.com/cyrillef/extract.autodesk.io/blob/master/server/lmv-token.js#L29).
Thank you, Cyrille, for the complete and succinct answer.