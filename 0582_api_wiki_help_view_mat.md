---
post_number: "0582"
title: "Wiki API Help, View Event and Structural Material Type"
slug: "api_wiki_help_view_mat"
author: "Jeremy Tammik"
tags: ['elements', 'parameters', 'revit-api', 'views']
source_file: "0582_api_wiki_help_view_mat.htm"
original_url: "https://thebuildingcoder.typepad.com/blog/0582_api_wiki_help_view_mat.html"
---

### Wiki API Help, View Event and Structural Material Type

I am back again from my trip to Turkey and the attempt to
[climb Mount Ararat](http://thebuildingcoder.typepad.com/blog/2011/05/improved-mep-element-shape-and-mount-ararat.html#2).
Unfortunately, we were a bit unlucky with the weather on the day of our summit bid and forced to turn back at 4900 m, less than 200 m below the top, after spending several days on the mountain.
We established a base camp just below the snow limit at about 2500 m, and a high camp in the snow at around 3500 m, and were not equipped to wait for another day and repeat the attempt.
Also, I got a little bit of frostbite on my nose and cheek.
No lasting damage, though.

Still, we all returned safe and sound and had a wonderful trip through Turkey and a day in Istanbul during the return travel.
I am fascinated by and love the extremely logical Turkish language, which I once dabbled with at university and was very happy to warm up a little bit again.

Now I am completely busy preparing the
[Revit 2012 API webcast](http://thebuildingcoder.typepad.com/blog/2011/05/revit-2012-api-webcast.html) that we will be presenting tomorrow, in just 30 hours' time.

Meanwhile, here are a couple of noteworthy little Revit 2012 API news items:

#### Revit API Wiki Help Online

As I recently mentioned, the
[Revit 2012 product help is available online](http://thebuildingcoder.typepad.com/blog/2011/04/revit-2012-wikihelp-overview.html).

Now the
[Revit API help](http://autodesk.com/revitapi-help) has
been added to the system as well, so that a full community learning and sharing platform is available for the entire Revit add-in developer ecosystem.

Finally, the
[global online Revit API help](http://thebuildingcoder.typepad.com/blog/2009/08/online-revit-api-help.html) initiated
by Rod Howarth can be brought to full fruition.

#### ViewChanging versus ViewActivating

The 'What's New' section in the Revit 2012 API help file mentions the ViewChanging and ViewChanged events.
These do not exist.
It is actually referring to the ViewActivating and ViewActivate events.

#### Structural Material Type in Revit 2012

Here is a question that I ran into myself as well when migrating some of the ADN sample code from the Revit 2011 to the Revit 2012 API, and which Joe Ye has now provided an answer to:

**Question:** In the Revit 2011 API, it was convenient to determine the structural material type using the classes MaterialSteel, MaterialConcrete etc.
In the Revit 2012 API, these have been declared obsolete.
How should one modify this code for Revit 2012, please?

The instructions in the Revit Platform API Changes and Additions document are not very helpful, nor the suggestions in the developer guide.
It suggests looking at the built-in parameter PHY\_MATERIAL\_PARAM\_TYPE, but the actual values returned do not match the table shown.

What is the recommended way to determine structural material type in the Revit 2012 API?

**Answer:** The parameter value and the material mapping table in the Revit 2012 API Developer guide has not updated.
Sorry for this.

In Revit 2012, we suggest using the built-in parameter PHY\_MATERIAL\_PARAMETER\_CLASS to check the material type.

The parameter value of that parameter does match the StructuralMaterialType enumeration values.