---
ai_optimized: true
author: Jeremy Tammik
complexity_score: 2.4
content_type: qa
optimization_date: '2025-12-11T11:44:16.161029'
original_url: https://thebuildingcoder.typepad.com/blog/1497_au_devday.html
post_number: '1497'
reading_time_minutes: 6
series: general
slug: au_devday
source_file: 1497_au_devday.md
tags:
- doors
- python
- revit-api
- sheets
- views
title: Au Devday
word_count: 1147
---

### DevDay Conference at Autodesk University
Yesterday afternoon, I checked into the Venetian hotel for AU and the preceding DevDay conference:
- [Desert day and night versus hotel morning](#1)
- [DevDay general session and Forge](#2)
- [AEC breakout](#3)
- [Revit API news, roadmap and idea station](#4)
- [BIM 360](#5)
- [InfraWorks 360 and Civil 3D](#6)
- [Forge's first birthday party](#7)
I am not expecting to emerge again from the hotel and the Sands conference centre into the outside world until the flight taking me back to Europe leaves on Thursday afternoon.
#### Desert Day and Night versus Hotel Morning
At least I spent some time beforehand acclimatising out in nature enjoying fresh air, peace and quiet, and lots of open space in the Calico Basin right next door to the Red Rock canyon.
Here are [some pictures](https://flic.kr/s/aHskNF4GKZ) showing basically nothing special:
[![Nevada Desert](https://c6.staticflickr.com/6/5688/30968024645_9ed6b55519_n.jpg)](https://www.flickr.com/photos/jeremytammik/albums/72157676535234815 "Nevada Desert")
I slept well out in the desert.
Here in the hotel, I woke up at four in the morning. My view looks like this:
![Four AM](/p/2016/2016-11-14_au_devday/034_four_am.jpg)
Unfortunately, the visual effect is severely marred by the noise from a bunch of air conditioning units just down below.
#### DevDay General Session and Forge
DevDays introduction: Jim Quanci explaining what Forge is about and showing an impressive series of demos, both quickly built custom-built code samples and real live customer implementations demonstrating how powerful and useful this is today.
![Jim Quanci](/p/2016/2016-11-14_au_devday/038_jim_quanci.jpg)
Don't miss this boat.
It happens all too easily, convincingly illustrated by Kodak, who patented the first digital camera, worked hard to shift their business from analogue chemical processing to digital technologies, reached the position of number one digital camera manufacturer just before the first iPhone was introduced, missed some important basic understanding of what they were actually doing, and filed for bankruptcy five years later:
![Demise of Kodak](/p/2016/2016-11-14_au_devday/042_demise_of_kodak.jpg)
Jim's pep talk, demos and motivational talk was followed by Shawn Gilmour, director of Forge product management, discussing current status and future plans for specific Forge APIs.
![Shawn Gilmour](/p/2016/2016-11-14_au_devday/045_shawn_gilmour.jpg)
My camera is too weak, the light too low, and the people on the stage wiggle too much.
Here are [some more pictures](https://flic.kr/s/aHskNKxNsk) anyway.
The only nice one I got was of Stephen Preston sitting next to me:
![Stephen Preston](/p/2016/2016-11-14_au_devday/046_stephen_preston.jpg)
Digging deeper into further technical details with Cyrille Fauvel:
Swagger definition of all Forge RESTful APIs, so you can generate a wrapper for any API you need for any programming language you want, e.g., managed C++, Python, Ruby, iOS, Java, etc.
Two- and three-legged OAuth.
Augusto Goncalves on Data Management API.
Jim presented more Forge case stories, followed by a panel discussion with:
- Ben Tytonovich, Project Manager, hooking up Forge with IoT via [Seebo](http://seebo.com/).
- Rob Oud, Inspirator, [Collaborea](http://collaborea.nl/en/), building supermodels, [BLDNG360](https://bldng360.com/). YouTube for buildings. Tried Unity, but not viable; Forge worked. Present models in public, harvest data.
- Sandip Jadhav, [simulationHub](https://www.simulationhub.com/), cloud-based CFD, democratizing computational fluid dynamics.
- Igor Tsinman, [AMCBridge](https://amcbridge.com/) and [openBOM](http://www.openbom.com/).
Kean Walmsley presented an overview of and an experiment using Forge for HoloLens VR and AR.
Cyrille Fauvel demonstrated another strategy and running sample for easily passing in and interacting with Forge model data in HoloLens.
Jim came back and explained the [Forge pricing plans](https://developer.autodesk.com/en/pricing), based on cloud credits or 'cc', currently roughly equivalent to a USD:
- Viewing is free, regardless of numbers
- Model derivative: cc 0.2 for a simple job, 1.5 for a complex one
- Design automation: cc 4 per processing hour
He illustrated with various examples, e.g.:
- 500 DWG files per month: $400
- Create IFC file from 10 RVT projects per day, 300 jobs per month: 300 \* 1.5 = ca. $450 per month
- Extract BOM from 1000 solidworks models per day (5000 jobs per week, 20k jobs per month): ca. 20000 \* 0.2 = $4000 per month
These plans were drawn up based on interviews with existing partners.
Trial includes 500 cc for the first year.
Note that almost every single Forge demonstration presented on stage today included Revit models integrated with CAD data and database properties from other, completely different sources.
Whatever use you are currently making of Revit, you can expect your models to end up being combined with other data and viewed through the Forge platform very soon indeed.
Think about what you do!
As a developer, think ahead.
Forget the noun = app.
Think of a verb = web service!
#### AEC Breakout
I cannot speak openly about everything discussed in the afternoon AEC breakout session, since it deals with unpublished desktop applications.
#### Revit API News, Roadmap and Idea Station
Sasha Crotty presented news about the Revit API, and also pointed out once again
the [public Revit roadmap](http://forums.autodesk.com/t5/revit-roadmaps/the-first-ever-public-revit-roadmap/ba-p/6633199).
We are looking very closely at implementing Forge design automation functionality for Revit.
This is where we are headed when talking about 'headless' Revit.
Very important: [Revit Idea Station](http://forums.autodesk.com/t5/revit-ideas/idb-p/302/).
Extensive Q & A.
The answer to a large number of requests: again and again: [Revit Idea Station](http://forums.autodesk.com/t5/revit-ideas/idb-p/302/).
Will we move away from Revit towards C4R?
No way, C4R is just a way to store the Revit model in the cloud. Future: more access through Forge to the cloud hosted models.
#### BIM 360
Mikako Harada discussed BIM 360 with a special focus on docs.
We had an overwhelming number of requests for APIs.
There is no BIM 360 API; rather, we implemented APIs that we use to build BIM 360 Docs.
This is a good example of Forge being used within Autodesk.
2-legged OAuth B2B...
A360 only has files...
BIM360 also has non-file documents, e.g., sheets within Revit RVT files...
DM sample...
model derivatives...
Brilliant demo of Mikako's new node.js based test harness Forge API navigator. ForgeDbg? ForgeLookup?
Summary:
- Visit [developer.autodesk.com](https://developer.autodesk.com/)
- Come to the [Forge DevCon 2017](https://forge.autodesk.com/DevCon)
- Say hi to us in the Forge booth
- Tweet to [@autodeskforge](https://twitter.com/AutodeskForge)
#### InfraWorks 360 and Civil 3D
The AEC session was rounded off by Augusto Goncalves discussing InfraWorks 360 and Civil 3D news.
#### Forge's First Birthday Party
Tomorrow, Tuesday, we celebrate Forge's first birthday!
Party: 19:00 at the Forge booth.
Cake and bubbly will be provided!