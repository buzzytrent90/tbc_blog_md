---
ai_optimized: true
author: Jeremy Tammik
complexity_score: 2.0
content_type: news
optimization_date: '2025-12-11T11:44:14.891564'
original_url: https://thebuildingcoder.typepad.com/blog/0919_restaurant_layout.html
post_number: 0919
reading_time_minutes: 6
series: general
slug: restaurant_layout
source_file: 0919_restaurant_layout.htm
tags:
- family
- filtering
- levels
- revit-api
- rooms
- schedules
- views
title: Cloud-Based Restaurant Seating Arrangement and Cleaning
word_count: 1297
---

### Cloud-Based Restaurant Seating Arrangement and Cleaning

I hope you had a successful
[Easter egg hunt](http://en.wikipedia.org/wiki/Egg_hunt).

![Easter eggs](img/easter_eggs_2013.jpg)

As you probably noticed, I have been focusing on the research and implementation of my
[cloud-based round-trip 2D Revit model editing project](http://thebuildingcoder.typepad.com/blog/2013/03/cloud-mobile-extensible-storage-data-use-in-schedules.html#3) for
the upcoming Autodesk internal tech summit in June as much as my day-to-day tasks will allow.

#### Current Project Overview

To recap, the basic idea is simple: implement an example of round-trip minimal simple editing of a 2D rendering of a Revit model on any mobile device with no need for installation of any additional software whatsoever beyond a browser.

To keep the editing task as minimal and simple as possible, I just envision changing the 2D location and orientation of the furniture and equipment within the boundaries of the selected room.

Such an editor can be realised on any mobile device using JavaScript and [SVG](http://en.wikipedia.org/wiki/Scalable_Vector_Graphics), scalable vector graphics, as I showed in my
[prototype room polygon editor](http://thebuildingcoder.typepad.com/blog/2013/02/2d-svg-editing-on-mobile-device-with-rapha%C3%ABl.html) using
the
[Raphaël](http://raphaeljs.com) JavaScript SVG library.

[![Room layout editor](http://thebuildingcoder.typepad.com/.a/6a00e553e168978833017d40f2e881970c-320wi "Room layout editor")](http://thebuildingcoder.typepad.com/svg/08-button.htm)
[Room layout editor](http://thebuildingcoder.typepad.com/svg/08-button.htm)

Aside from the editing component, this workflow will also require:

- A cloud-based database repository managing the Revit model, level, room, furniture and equipment data.
- A Revit add-in retrieving the data from the Revit project and populating the data repository.

The data repository is fed from the Revit model and queried by the mobile device editor.
It also manages changes to the furniture and equipment location and orientation.

The editor displays the room, furniture and equipment and enables translation and rotation of the latter, which updates the repository data.

The Revit add-in can optionally be expanded to automatically query and update the Revit model based on new furniture and equipment locations, resulting in the following workflow:

- Show a Revit model.
- Export the model to a cloud-based data repository.
- Query and display the repository contents in a 2D rendering on any mobile device.
- Edit the model in the browser on the mobile device, updating the data repository.
- Pick up and display the modified model in Revit.

I already discussed my current candidate for the cloud database using the
[CouchDB NoSQL database implementation](http://thebuildingcoder.typepad.com/blog/2013/03/relax-simple-free-cloud-based-data-repository-with-nosql-couchdb-and-iriscouch.html#4) and
[IrisCouch free hosting](http://thebuildingcoder.typepad.com/blog/2013/03/relax-simple-free-cloud-based-data-repository-with-nosql-couchdb-and-iriscouch.html#5) service.

Regarding the Revit add-in retrieving the room and furniture 2D boundary polygons, I also looked at the retrieval of
[plan view room boundary loops](http://thebuildingcoder.typepad.com/blog/2013/03/revit-2014-api-and-room-plan-view-boundary-polygon-loops.html#3) and
[family instances in a given room](http://thebuildingcoder.typepad.com/blog/2013/03/filter-for-family-instances-in-a-room.html).

Next steps will include enhancing the Revit add-in to extract suitable boundary loops to represent the furniture and equipment in the mobile editor, and above all the data repository implementation.

#### Project Expansion for Cloud-Based Restaurant Seating and Cleaning

As a very happy surprise, I now found a beta site to test the existing functionality and add even more, by hooking up the mobile device furniture layout editing functionality with a restaurant cash register to implement a fully automated restaurant seating layout and floor cleaning system.

The basic idea is to equip the restaurant furniture with unobtrusive mobility support and link the virtual model of the restaurant and its furniture with the real-world objects.

All that is required to provide real-world control over the furniture position are concealed wheels and locking devices inside the table base.
The wheels are normally locked in position, and their existence goes unnoticed.
They can be unlocked and driven by motors for displacement.

The entire restaurant layout can thus be rearranged via the 2D layout editor on the mobile device, with no manual intervention whatsoever required.

Each piece of furniture also includes location detection support.
Besides the automatic rearrangement triggered by updating the virtual model, the inverse is also supported: moving the furniture around manually can trigger the virtual model to be updated instead to reflect the new real-world positions.

Here is a snapshot of the restaurant cash register showing the original layout before editing:

![Restaurant cash register screen](img/restaurant_layout/restaurant_layout_3.jpg)

The cash register lives in the pillar in the centre of the restaurant.
The pillar is not displayed on the cash register screen snapshot above:

![Original restaurant layout](img/restaurant_layout/restaurant_layout_1.jpg)

The bar in the corner is marked in red on the screen snapshot, and its position cannot be edited:

![Unmodifiable bar in red](img/restaurant_layout/restaurant_layout_2.jpg)

This layout is stored in a Revit model and processed fully automatically via the following steps:

- Export to cloud-based data repository using CouchDB and an IrisCouch host, including model, level, room and furniture metadata and room and furniture plan view boundary polygons.
- Display on mobile device using [SVG](http://en.wikipedia.org/wiki/Scalable_Vector_Graphics) and the [Raphaël](http://raphaeljs.com) JavaScript SVG library.
- Edit furniture location and rotation on any mobile device, anywhere.
- Every modification immediately updates the cloud-based data repository furniture transform.
- Optionally, the Revit model is auto-updated as well, making use of the external event functionality to
  [drive Revit through a WCF service](http://thebuildingcoder.typepad.com/blog/2012/11/drive-revit-through-a-wcf-service.html).

Here is the same model in Revit:

![Restaurant layout in Revit](img/restaurant_layout/restaurant_layout_5.jpg)

The SVG editor displays it like this in the browser on any mobile device:

![Restaurant layout in SVG](img/restaurant_layout/restaurant_layout_6.jpg)

Rotating and moving the tables in SVG triggers the synchronisation chain and drives the real-world furniture to its new position:

![Updated real-world restaurant layout](img/restaurant_layout/restaurant_layout_0.jpg)

As you can see, the table base looks completely normal and can easily contain the required motors, wheels, locking and location devices:

![Ample table base housing movement devices](img/restaurant_layout/restaurant_layout_0_zoom.jpg)

#### Fully Automated Restaurant Cleaning

Mobile autonomous furniture obviously also vastly facilitates the restaurant cleaning.

The furniture is programmed to flock together at one end of the room each night, making space for a
[cleaning robot](http://www.irobot.com/en/us/robots/home/Mint.aspx) to
roam freely, and moves over to the other end of the room after the first half has been covered.

This obviously enables much more effective and thorough cleaning than if the furniture is left in place:

It is fascinating to see BIM expanding into new hitherto unexpected areas, and unbelievably exciting to participate in the process!

I trust you will find similar opportunities to embrace this paradigm and successfully use the Revit API to support expanding your applications and ideas into other new niches.

I am looking forward to keeping you posted on the further progress of this ongoing project!

#### JavaScript Physics Sample

By the way, talking about new niches and programming in and for the cloud, here is an utterly cool
[JavaScript physics sample](http://codepen.io/stuffit/pen/KrAwx) simulating ripping a curtain of tearable cloth.

It provides an incredibly compelling and sheer unbelievably compact example use of HTML5, Canvas, CSS and JavaScript for an interactive graphical physical simulation in just three hundred lines of code.