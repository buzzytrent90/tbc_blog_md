---
ai_optimized: true
author: Jeremy Tammik
complexity_score: 1.9
content_type: qa
optimization_date: '2025-12-11T11:44:15.549672'
original_url: https://thebuildingcoder.typepad.com/blog/1225_poipointer_view_depth.html
post_number: '1225'
reading_time_minutes: 5
series: geometry
slug: poipointer_view_depth
source_file: 1225_poipointer_view_depth.htm
tags:
- elements
- python
- references
- revit-api
- schedules
- views
- walls
- geometry
title: PoiPointer, View Depth Override and Destination BIM
word_count: 1028
---

### PoiPointer, View Depth Override and Destination BIM

Three topics for today:

- [Brussels hackathon and PoiPointer](#2)
- [View depth override](#3)
- [Destination BIM contest](#4)

#### Brussels Hackathon and PoiPointer

I returned from the
[Hackathon Open Data Brussels](http://www.transformabxl.be/agenda/event/hackathon-open-data-brussels) that I
[mentioned last Friday](http://thebuildingcoder.typepad.com/blog/2014/10/brussels-hackathon-and-determining-pipe-wall-thickness.html#2),
promoting the use of the huge amounts of open data, cf. this impressive
[list of available data sets](http://www.transformabxl.be/blog/post/hackathon-open-data-brussels-list-of-available-datasets).

As said, I participated in the
[PoiPointer](https://github.com/PoiPointer) project,
with a goal of implementing an app pointing out points of interest of various categories in Brussels, e.g. museums, cultural places, monuments, sculptures, fountains, murals, etc.

We used the schema-less REST driven JSON-based [ElasticSearch](http://www.elasticsearch.org) database and Java for the back end database,
[node.js](http://nodejs.org) and JavaScript for the web server,
Objective-C for the iOS mobile app, Python for database clean-up and verification and HTML with GitHub in-place website hosting for the project home page.

Here is the project home page and all its GitHub repositories:

- [Home page](http://poipointer.github.io)
- [Live demo simulation](https://www.justinmind.com/usernote/tests/12951835/12951839/12951841/index.html)
- [Slide deck](http://fr.slideshare.net/flavienroelandt/poi-pointer-team65)
- [GitHub repositories](https://github.com/PoiPointer)

- [PoiPointer data sources and verification utility](https://github.com/PoiPointer/dataSources) (Python)
- [PoiPointer marketing](https://github.com/PoiPointer/marketing), i.e. images, icons, etc.
- [PoiPointer node.js server](https://github.com/PoiPointer/node.js) (JavaScript)
- [Back end used to sync db](https://github.com/PoiPointer/opendataBrusselsSync) with [opendata.brussels.be](http://opendata.brussels.be) (Java)
- [PoiPointer iOS app](https://github.com/PoiPointer/POIPointer) (Objective-C)
- [Home page sources](https://github.com/PoiPointer/poipointer.github.io) (HTML)

- [HackaBXL](https://github.com/HackaBXL), the official Brussels Open Data hackathon organisation

- [HackaBXL PoiPointer page](https://github.com/HackaBXL/2014_POI-Pointer)

I learned lots of new things and will definitely take a closer look soon at implementing some simple cloud-connected database search engine connected with Revit BIM collection using node.js and ElasticSearch.

I love the GitHub web hosting functionality and will certainly make frequent continued use of that as well.

#### View Depth Override

I received an email asking whether it might be possible to implement a perception of depth on elevation and section views:

**Question:** I am struggling to provide an automatic perception of depth on elevation and section views.

I would like the objects in the front to be displayed in black and white like Revit normally does.
Everything in the background should be a grey of sorts, and the further back, the lighter the grey.

Here is an illustration of what I mean:

![View depth perception](img/view_depth_override.png)

- From the section or elevation line, I would like to define a distance or area where everything would show black & white like Revit normal. This distance or depth can be adjusted.
- Further back from the Revit normal area until you reach the depth clipping line, I would like to make a setting that overrides the normal display and allows the user to set a starting grey and an end grey ranging from dark to light. It would be nice if the element projection line thickness could be overridden in the same way.

Here is a discussion on whether
[Revit elevation views are leaving you flat](http://cad-vs-bim.blogspot.com/2010/10/are-revit-elevation-views-leaving-you.html) that
might help provide a better explanation of the end result that I am looking to achieve.

I have no knowledge of the Revit API or add-in programming. Do you think that this is possible?

Can this be achieved using API or through an add-in?

**Answer:** You are in luck, especially since you say that you are not an experienced add-in programmer yourself.

I once happened to notice an add-in that does exactly what you are asking for.

It was published on an Italian blog, though.

I spent quite a while digging around for it and was initially still not able to find it or any way to pick it out from the mass.

Searching for "italian add-in" obviously did not help, and I could not imagine what it would.

Later...

I spent some more time on this and finally found a suitable search term:
[revit macro distanzia](http://lmgtfy.com/?q=revit+macro+distanzia) led to
[puntorevit.blogspot.com](http://puntorevit.blogspot.com),
and that in turn led to the mention I had in mind, on the
[view depth override macro](http://thebuildingcoder.typepad.com/blog/2013/12/devdayau-chronicle-estorage-view-depth-sound-of-noise.html#11
) by
Paolo Emilio Serra.

I browsed through
[inpuntorevit](http://puntorevit.blogspot.com) a
bit more and even discovered a recent update of the
[view depth override external command](http://puntorevit.blogspot.it/2014/07/view-depth-override-external-command.html) for Revit 2015.

By the way, Paolo switched from Italian to English language blogging now, which may or may not simplify things for you, depending on your language preferences :-)

#### Destination BIM Contest

Are you a BIM champion?

Enter the
[Destination BIM contest](http://www.destinationbim.com) and
win the chance to attend Autodesk University 2014.

Contest Details

- When: Content runs from October 17, 2014 until November 3, 2014.
- Who can enter: Anyone using or interested in BIM! Winner is open to US and Canada only.
- How to enter: Contestants fill out form and tell us how their BIM pilot is helping to change their organization on the dedicated contest landing page.
- How to win: winner selected by Autodesk committee by November 7, 2014.
- The Prize: 4 winners. Each receives one pass to Autodesk University Las Vegas, $600 travel voucher and hotel stay.
- Fine print:

- Contestants grant Autodesk permission to use their submission materials and interview content captured at AU
- Winners must participate in interviews and appear in video at AU per schedule determined by Autodesk

![Destination BIM](img/destination_bim.jpeg)