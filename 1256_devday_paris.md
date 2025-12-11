---
ai_optimized: true
author: Jeremy Tammik
complexity_score: 1.4
content_type: qa
optimization_date: '2025-12-11T11:44:15.619911'
original_url: https://thebuildingcoder.typepad.com/blog/1256_devday_paris.html
post_number: '1256'
reading_time_minutes: 3
series: general
slug: devday_paris
source_file: 1256_devday_paris.htm
tags:
- elements
- family
- filtering
- geometry
- revit-api
- views
- walls
- windows
title: DevDays Conference and Meetup in Paris
word_count: 671
---

### DevDays Conference and Meetup in Paris

I left Autodesk University, Las Vegas, and travelled to Paris via London.

#### DevDay and Meetup

Today, we held the first West European
[DevDays](https://twitter.com/hashtag/devdays2014) conference
here, followed by a meetup on reality capture, 3D on the web and 3D printing in the
[Ecole des ingenieurs de la Ville de Paris](https://en.wikipedia.org/wiki/Ecole_des_ingenieurs_de_la_Ville_de_Paris) ([eivp-paris.fr](http://www.eivp-paris.fr)).

From here, we continue to
[London, Gothenburg, Munich and Milano](file:///a/doc/revit/blog/1199_devdays_hack_calendar.htm#2) during the coming ten days.

#### Jasper

Unexpectedly, I got to meet Jasper Desmet, whom I previously only knew by email.
Last year, he presented his
[space adjacency for heat load calculation](http://thebuildingcoder.typepad.com/blog/2013/07/football-and-space-adjacency-for-heat-load-calculation.html#3) here
on The Building Coder.
He is now studying at
[ecole 42](http://en.wikipedia.org/wiki/42_(school)) ([42.fr](http://www.42.fr)),
an exciting programming school inspired by new modern learning methods including peer-to-peer pedagogy, adding some programming experience to his construction engineering knowledge to create interesting opportunities in CAD.

#### Vélib'

Once again – like a couple of years ago, in
[December 2010](http://thebuildingcoder.typepad.com/blog/2010/12/filtered-element-collector-sample-overview.html) –
I have been making extensive use of the wonderful Parisian
[Vélib'](http://en.wikipedia.org/wiki/V%C3%A9lib%27) bicycle
rental system, e.g. here at the
[Centre Pompidou](https://en.wikipedia.org/wiki/Centre_Georges_Pompidou):

![Vélib' rental station at Centre Pompidou](file:////j/photo/jeremy/2014/2014-12-06_paris/20141206_130423_cropped.jpg)

#### Aaron Lu's New Posts

While I was underway at Autodesk University and getting here, my new colleague
[Aaron Lu](http://adndevblog.typepad.com/aec/aaron-lu.html) in China has been very productive, publishing 12 articles on the Chinese blog and 3 on the international AEC DevBlog:

- [How to get DWGExportOptions](http://adndevblog.typepad.com/aec/2014/12/revitapi-how-to-get-dwgexportoptions.html)
- [How to get MEP switch system from FamilyInstance](http://adndevblog.typepad.com/aec/2014/12/revitapi-how-to-get-switch-system-from-familyinstance.html)
- [Code formatting for blog publication](http://adndevblog.typepad.com/aec/2014/12/code-formatting-in-typepad-blog.html)

#### Round Window Opening Geometry – Or Not

Pretty obvious, once you think about it: if an element has no solid geometry of its own, you cannot retrieve any solid for it:

**Question:** I have a round window opening in a wall like this:

![Round window opening](img/get_round_window_geometry.jpg)

However, I am unable to obtain any geometry information from this family instance and the round shape window symbol.
I was able to retrieve their solids, but they have are no faces or edges.
You can also see it using RevitLookup and the ElementViewer SDK sample.

Is there any way to get the geometry information for these round shaped windows?

**Answer:** There should be no trouble reading the geometry of this type of window.
It will either contain solids or a GeometryInstance or both.
If you only find empty solids, please provide the sample model for us to explore.

**Response:** I explored this window opening with the ElementViewer sample and it still seems to have an empty solid.

**Answer:** Ahaaa... this window is just a cut-out – there is no glass or other substance here.
Therefore, the family instance does not have any own geometry, just a void.
The only way to determine the shape of the void is to look at what it does to its host.

#### DuckDuckGo

By the way, talking about searching for answers...

My son recommended that I boycott Google and start using the
[DuckDuckGo](https://duckduckgo.com) search engine instead.

They guarantee not to track your information (or any information whatsoever, for that matter), provide more transparent search algorithms, are completely open source and
[easily extensible](http://duckduckhack.com).

So far, I like it very much!

Tomorrow, we'll take off for the next venue in good old Blighty.