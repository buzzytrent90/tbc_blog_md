---
post_number: "0932"
title: "Room Editor Project Overview and CouchDB Configuration"
slug: "room_edit_couchdb_conf"
author: "Jeremy Tammik"
tags: ['csharp', 'family', 'python', 'revit-api', 'rooms', 'schedules', 'views']
source_file: "0932_room_edit_couchdb_conf.htm"
original_url: "https://thebuildingcoder.typepad.com/blog/0932_room_edit_couchdb_conf.html"
---

### Room Editor Project Overview and CouchDB Configuration

Continuing the research and development for my
[cloud-based round-trip 2D Revit model editing project](http://thebuildingcoder.typepad.com/blog/2013/03/cloud-mobile-extensible-storage-data-use-in-schedules.html#3),
here are some issues I tackled since my last report:

- [Project Overview](#2)
- [Replicating a CouchDB with Python](#3)
- [Same origin policy snag and resolution](#4)
- [Learning and struggling with CouchApps](#5)
- [Relief moving to Kanso](#6)
- [Next steps](#7)

#### Project Overview

To recapitulate, the grand plan is to grab a 2D floor plan with furniture and equipment layout from the BIM, upload it to the cloud, and make it accessible for viewing on mobile devices:

![Upload from desktop to cloud and view on mobile device](img/desktop_cloud_mobile_2_upload.png)

Furthermore, I am enabling some simple editing operations on the mobile device, e.g. to move the furniture and equipment around in the room a bit, update the cloud-based data repository, and reflect these changes back into the desktop BIM:

![Edit on mobile device, update data repository and BIM](img/desktop_cloud_mobile_3_edit_and_download.png)

All of this with absolutely nil installation on the mobile device, and thus totally ubiquitous, using
[SVG](http://en.wikipedia.org/wiki/Scalable_Vector_Graphics) and
[server-side scripting](http://en.wikipedia.org/wiki/Server-side_scripting).

I already addressed a number of issues in realising this project and am documenting my progress here on the blog, in the three categories
[desktop](http://thebuildingcoder.typepad.com/blog/desktop),
[cloud](http://thebuildingcoder.typepad.com/blog/cloud) and
[mobile](http://thebuildingcoder.typepad.com/blog/mobile).

Here is an overview of the results presented so far, both for this project and some previous related topics, classified into those three categories:

- [Project overview and Tech Summit proposal](http://thebuildingcoder.typepad.com/blog/2013/03/cloud-mobile-extensible-storage-data-use-in-schedules.html)- Desktop
  - [Room plan view boundary loops](http://thebuildingcoder.typepad.com/blog/2013/03/revit-2014-api-and-room-plan-view-boundary-polygon-loops.html)
  - [Sort and orient curves to form a contiguous loop](http://thebuildingcoder.typepad.com/blog/2013/03/sort-and-orient-curves-to-form-a-contiguous-loop.html)
  - [Extrusion analyser and plan view boundaries](http://thebuildingcoder.typepad.com/blog/2013/04/extrusion-analyser-and-plan-view-boundaries.html)
  - [Curve following face and bounding box implementation](http://thebuildingcoder.typepad.com/blog/2013/04/curve-following-face-and-bounding-box-implementation.html)
  - [GeoSnoop boundary curve loop visualisation](http://thebuildingcoder.typepad.com/blog/2013/04/geosnoop-net-boundary-curve-loop-visualisation.html)
  - [Desktop to cloud via DreamSeat CouchDB Client](http://thebuildingcoder.typepad.com/blog/2013/04/desktop-to-cloud-via-dreamseat-couchdb-client.html)- Cloud
  - [Getting Going with the Cloud](http://thebuildingcoder.typepad.com/blog/2012/06/getting-going-with-the-cloud.html)
  - [Apollonian Sphere Packing Web Service and AVF](http://thebuildingcoder.typepad.com/blog/2012/09/apollonian-packing-of-spheres-via-web-service-and-avf.html)
  - [The BIM 360 Glue Viewer and REST API](http://thebuildingcoder.typepad.com/blog/2012/12/the-bim-360-glue-viewer-and-rest-api.html)
  - [BIM 360 Glue REST API Authentication Using Python](http://thebuildingcoder.typepad.com/blog/2012/12/bim-360-glue-rest-api-authentication-using-python.html)
  - [Free Cloud Based Data Repository with NoSQL, CouchDB, and IrisCouch](http://thebuildingcoder.typepad.com/blog/2013/03/relax-simple-free-cloud-based-data-repository-with-nosql-couchdb-and-iriscouch.html)- Mobile
  - [Testing my OBJ exporter on Android](http://thebuildingcoder.typepad.com/blog/2012/08/validate-roof-type-and-view-obj-on-android.html)
  - [Mobile Device Room Location](http://thebuildingcoder.typepad.com/blog/2012/09/mobile-device-room-location.html)
  - [Room Polygon and Furniture Picker in SVG](http://thebuildingcoder.typepad.com/blog/2012/10/room-polygon-and-furniture-picker-in-svg.html)
  - [2D SVG Editing on Mobile Device with Raphael](http://thebuildingcoder.typepad.com/blog/2013/02/2d-svg-editing-on-mobile-device-with-rapha%C3%ABl.html)

#### Replicating a CouchDB with Python

In my last step, I determined the room, furniture and equipment plan view boundary polygons and uploaded them to a CouchDB database in the cloud, hosted on IrisCouch.

To play around interactively with that data, I would prefer to store it locally instead of on the server.

The easiest way to achieve this is to replicate the database.

Replicating a CouchDB repository is trivial, since it has built-in support for that.

All you need to do is specify the source and target database name and URL.

Also, as you may know, I love Python for command-line scripting.

Happily, several
[CouchDB interfaces for Python](http://wiki.apache.org/couchdb/Getting_started_with_Python) are
available.
As you can see on that page, they can be pretty simple.

[Couchdbkit](http://couchdbkit.org) seems
well supported and does the job.

Here is a little Python script that replicates the database 'rooms' created by my Revit add-in in IrisCouch from the web to a new database 'rooms2' my local CouchDB installation:

```csharp
#!/usr/bin/python
import couchdb
server = couchdb.Server()
server.create( 'rooms2' )
server.replicate(
'http://jt.iriscouch.com/rooms',
'rooms2' )
```

You cannot make it much shorter or simpler than that, can you?

#### Same Origin Policy Snag and Resolution

I was originally envisioning a simple architecture composed of three components on the desktop, in the cloud, and on the mobile device, respectively, as illustrated above.

I started looking at how to access the cloud-based data repository from the mobile device and closed my last foray into this area mentioning that I had a nasty surprise discovering that the strict
[same origin policy](http://www.w3.org/Security/wiki/Same_Origin_Policy) prevents
my simple JavaScript application from accessing my IrisCouch domain, so I cannot easily read the data on the mobile device in the manner I had expected.

The discovery cost me a few grey hairs and some sleep, but I found a good and obvious solution for that which you are probably all aware of already:
[server-side scripting](http://en.wikipedia.org/wiki/Server-side_scripting).

In fact, this even simplifies things, in the end, since I really have nothing at all to install or implement on the mobile device, and my list of components to implement is reduced from three to two.
Ok, it adds some complication to the cloud stuff, of course... or actually just additional learning opportunities.

I had already decided that NoSQL and the CouchDB implementation look like good candidates for my data repository requirements, and that I can test and host them in the cloud for free using the IrisCouch server.
Please refer to the link in the overview above for more info on these.

Happily, CouchDB offers full support for server-side scripting.
The same-origin policy effectively requires that the HTML from which the JavaScript is loaded must be served up from CouchDB.
This can be done by attaching an HTML document to a CouchDB document.
You can do this manually, or through the use of CouchApps.

#### Learning and Struggling with CouchApps

CouchApps are CouchDB generation environments and tools for web application developers interested in creating database-driven applications using nothing but HTML, CSS, and JavaScript.

A CouchApp is simply a collection of files that completely defines a CouchDB database to simplify development, deployment and maintenance.
Simple command-line tools are used to synchronise the file collection with the real live CouchDB implementation.

I dived into several of the numerous CouchApp tutorials and was so excited I ended up spending entire nights at it, once with no sleep at all.

Here are some of the things I looked at:

- [Getting started](http://www.couchapp.org/page/getting-started)
- [Build a simple CouchApp](http://blog.edparcell.com/using-jquery-and-couchdb-to-build-a-simple-we)
- [Good IBM CouchApp tutorial](http://www.ibm.com/developerworks/opensource/tutorials/os-couchapp/index.html)
- [A short and effective CouchApp and JavaScript tutorial](http://railsware.com/blog/2012/03/12/couchdb-and-couchapp-part-1)
- [A better introduction and some more modern alternatives](http://couchapp.org/page/index)
- [Documentation for jquery.couch.js](http://daleharvey.github.io/jquery.couch.js-docs/symbols/index.html)
- [A very clear and impressive CouchApp video tutorial](http://vimeo.com/26147136) by Max Ogden
- [Very impressive node CouchApp JavaScript library node.couchapp.js](https://github.com/mikeal/node.couchapp.js)
- [Using node.js for effective asynchronous processing](https://github.com/maxogden/art-of-node)
- [node.js + CouchDB == Crazy Delicious](http://jsconf.eu/2010/speaker/nodejs_couchdb_crazy_delicious.html) video presentation

Unfortunately, I found it pretty confusing.
I learned a huge amount and achieved very little, partly due to the fact that several different CouchApp implementations exist and are used in parallel, and the tutorials do not specify exactly which version they are based on.

Still, I learned enough to continue feeling very enthusiastic about this approach, and then found an easier solution.

#### Relief Moving to Kanso

After struggling with different CouchApp implementations, I discovered
[Kanso](http://kan.so),
toting itself "the best way to create and share apps on CouchDB".

With the little that I know so far, I fully agree with that.
Apparently, kanso is the Japanese phrase for simplicity, or, more specifically, simplifying the complex.

So easy!
Everything works!
The
[documentation](http://kan.so/docs)
includes a short and succinct getting started tutorial that explains a lot.

Later I found
[another tutorial](http://caolan.github.io/kanso/guides/getting_started.html) that
goes further faster than the one included in the documentation pages.

Later still, I found this extremely well documented
[complete application](https://github.com/mauget/opto3-couchapp) that
can be
[tested online](http://mauget.cloudant.com/opto3/_design/opto/index.html) and
demonstrates numerous additional important new features.

After studying these samples and trying to reuse some of the powerful functionality they provide, I am returning more and more often to the
[Definite Guide for CouchDB](http://guide.couchdb.org) and
finally starting to understand the basics of view, show and list functions, etc.

Bit by bit, I am managing to get some SVG graphics to display in my server-side scripts.
Making use of the Raphael library, as I was planning to do, may prove hard, since there is no ready-built Kanso package for it.
I may therefore revert to implementing my room furniture placement editor in pure SVG instead.

#### Next Steps

My next steps will be:

- Separation of symbol and instance data in my add-in and database structure: currently, the furniture loops are placed absolutely, and multiple instances of a symbol duplicate the same loop over and over again at different locations. I will rewrite this to separate the furniture loop definition, defined by the family symbol, and its placement, defined by the instance. This needs to be done anyway to enable editing the placement data through the editor interaction on the mobile device.
- Learn how to use templates in Kanso.
- Implement server-side generated SVG code to display the room and furniture plan in CouchDB using Kanso.
- Implement editing of SVG on the mobile device and reflect changes back to CouchDB (I do not know how yet).
- Implement Idling event handler and polling of CouchDB in the desktop add-in to reflect the changes back to the BIM in real-time.
- Implement an external application wrapper for the add-in providing four commands:

- Upload to cloud
- Refresh from cloud
- Subscribe to cloud
- Unsubscribe from cloud

I know exactly how to address all these points now, except for reflecting back the SVG editor changes to the CouchDB.

I remain excited and optimistic and look forward to hearing your comments and suggestions.

---

# Cloud and Mobile

### Room Editor Project Overview and CouchDB Configuration

By
[Jeremy](http://adndevblog.typepad.com/cloud_and_mobile/jeremy-tammik.html)
[Tammik](http://thebuildingcoder.typepad.com/blog/about-the-author.html).

I am continuing the research and development for my cloud-based round-trip 2D Revit model editing project.

To recapitulate, the grand plan is to grab a 2D floor plan with furniture and equipment layout from the BIM, upload it to the cloud, and make it accessible for viewing on mobile devices:

![Upload from desktop to cloud and view on mobile device](img/desktop_cloud_mobile_2_upload.png)

Furthermore, I am enabling some simple editing operations on the mobile device, e.g. to move the furniture and equipment around in the room a bit, update the cloud-based data repository, and reflect these changes back into the desktop BIM:

![Edit on mobile device, update data repository and BIM](img/desktop_cloud_mobile_3_edit_and_download.png)

All of this with absolutely nil installation on the mobile device, and thus totally ubiquitous, using
[SVG](http://en.wikipedia.org/wiki/Scalable_Vector_Graphics) and
[server-side scripting](http://en.wikipedia.org/wiki/Server-side_scripting).

Here are some issues I tackled since my last report:

- [Project Overview](http://thebuildingcoder.typepad.com/blog/2013/04/room-editor-project-overview-and-couchdb-configuration.html#2)
- [Replicating a CouchDB with Python](http://thebuildingcoder.typepad.com/blog/2013/04/room-editor-project-overview-and-couchdb-configuration.html#3)
- [Same origin policy snag and resolution](http://thebuildingcoder.typepad.com/blog/2013/04/room-editor-project-overview-and-couchdb-configuration.html#4)
- [Learning and struggling with CouchApps](http://thebuildingcoder.typepad.com/blog/2013/04/room-editor-project-overview-and-couchdb-configuration.html#5)
- [Relief moving to Kanso](http://thebuildingcoder.typepad.com/blog/2013/04/room-editor-project-overview-and-couchdb-configuration.html#6)
- [Next steps](http://thebuildingcoder.typepad.com/blog/2013/04/room-editor-project-overview-and-couchdb-configuration.html#7)

Check it out, and please let us know what you think!