---
post_number: "0912"
title: "Cloud & Mobile, Extensible Storage Data Use in Schedules"
slug: "cm_estorage_schedule"
author: "Jeremy Tammik"
tags: ['doors', 'elements', 'family', 'geometry', 'levels', 'parameters', 'python', 'revit-api', 'rooms', 'schedules', 'views']
source_file: "0912_cm_estorage_schedule.htm"
original_url: "https://thebuildingcoder.typepad.com/blog/0912_cm_estorage_schedule.html"
---

### Cloud & Mobile, Extensible Storage Data Use in Schedules

My topics for today are:

- [Cloud and mobile topics I covered so far](#2)
- [Cloud-based round-trip 2D Revit model editing on any mobile device](#3) using SVG
- [Extensible storage data use in schedules](#4)

I am still meeting with my European DevTech colleagues here in Konk Leon, 'anse du Lèon', the cove of the lion, aka Le Conquet, Brittany.

Yesterday, one of our topics was work we have done so far for cloud and mobile applications.
It was very interesting to hear about my peers' rich and varied experiences.

#### Cloud and Mobile Topics Covered so Far

I documented all my own experiences so far here on the blog, and was able to reuse that for my presentation quite spontaneously with no further preparation:

- [Accessing a REST API web service from a Revit .NET add-in](http://thebuildingcoder.typepad.com/blog/2012/09/apollonian-packing-of-spheres-via-web-service-and-avf.html) – Apollonian sphere packing and AVF.
- Thoughts on
  [mobile indoor positioning](http://thebuildingcoder.typepad.com/blog/2012/09/mobile-device-room-location.html),
  representing room polygons in SVG, the Azure SDK, ASP.NET MVC 4, and implementing a web service.
- An overview of the [BIM 360 Glue viewer and REST API](http://thebuildingcoder.typepad.com/blog/2012/12/the-bim-360-glue-viewer-and-rest-api.html).
- BIM 360 Glue REST API [authentication using Python](http://thebuildingcoder.typepad.com/blog/2012/12/bim-360-glue-rest-api-authentication-using-python.html).
- A room polygon and [furniture picker in SVG](http://thebuildingcoder.typepad.com/blog/2012/10/room-polygon-and-furniture-picker-in-svg.html).
- 2D [SVG editing on mobile devices](http://thebuildingcoder.typepad.com/blog/2013/02/2d-svg-editing-on-mobile-device-with-rapha%C3%ABl.html) using the Raphaël JavaScript SVG library.

As I explained at the time, all of the explorations involving SVG are part of a larger master plan.

This plan has now matured to a certain extent and is forcing me to move ahead quite fast in the immediate future.

The main outcome so far is a presentation proposal that I submitted for the internal Autodesk technical summit taking place in June, and the fact that it was accepted.

My cloud and mobile roadmap is thus now clearly focused on implementing that proposal:

#### Cloud-based Round-trip 2D Revit Model Editing on any Mobile Device using CouchDB and SVG

I described the roadmap I envision in the discussion of my simple
[SVG editor](http://thebuildingcoder.typepad.com/blog/2013/02/2d-svg-editing-on-mobile-device-with-rapha%C3%ABl.html).
The main components include:

- A Revit add-in that determines room boundary and furniture and equipment family instance polygons for a simplified 2D plan view rendering.
- A cloud-based database repository managing the models, levels, rooms, furniture, equipment meta- and polygon data.
- The SVG editor running in a browser on any device, rendering the model and supporting some simple geometric modification.

Here is the actual wording of my technical summit proposal:

##### Proposal Abstract

This presentation demonstrates round-trip editing a 2D rendering of a Revit model on any mobile device with no need for installation of any additional software whatsoever beyond a browser. How can this be achieved? A Revit add-in exports polygon renderings of room boundaries and other elements such as furniture and equipment to a cloud-based repository. On the mobile device, the repository is queried and the data is rendered in a standard browser using JavaScript and SVG. The rendering supports modification, such as translation and rotation of the furniture and equipment, which is saved back to the cloud database. The Revit add-in picks up these changes and updates the Revit model.

##### Intended Audience and Prerequisites

Intended audience is programmers with interest in cloud-based and mobile computing, specifically interest in round-trip editing of a Revit model using a cloud-based data repository and editing a 2D model on an arbitrary mobile device with no additional software beyond a browser installed on it.
 Knowledge of .NET, JavaScript and REST programming is helpful but not required.
Revit add-in experience is not a prerequisite.

##### Presentation Outline

- Show a Revit model.- Export the model to a cloud-based data repository.- Query and display the repository contents as 2D rendering on a mobile device.- Edit the model in the browser on the mobile device, updating the data repository.- Pick up and display the modified model in Revit.

Two of the important components whose implementation I plan to explore next are the polygon generation and data model:

##### Plan View Polygon Determination

The generation of the room boundary polygons is obvious, since the room has a boundary property.

For the furniture and equipment family instance, the solution is a less immediate.
My current idea is to explore making use of the ExtrusionAnalyzer class.
It is supplied a solid geometry, a plane, and a direction.
From those, it calculates the outer boundary of the shadow cast by the solid onto the input plane along the extrusion direction.
As far as I can tell, that should be exactly what I need.

Scott Conover mentioned this class in his session on the
[Geometry API](http://thebuildingcoder.typepad.com/blog/2012/06/devcamp-day-one.html),
a continuation of his AU 2011 presentation
[CP4011](http://au.autodesk.com/?nd=event_class&session_id=9124&jid=1725932) 'Geometric Progression: Further Analysis of Geometry Using the Autodesk Revit 2012 API',
enhanced for the Revit 2013 API.

As soon as I have the polygons determined, I want to implement functionality to render them easily.
This will involve extent calculation, canvas definition, and coordinate system transformation.
I will probably implement a simple .NET geometry viewer before moving on to the SVG rendering.
I've been wanting to do that for a long time...

##### Cloud-based Data Repository

So far, my colleagues have been focusing on using the Amazon and Azure cloud hosting services, and many discussions on the
[cloud and mobile DevBlog](http://adndevblog.typepad.com/cloud_and_mobile) deal
with them.

I searched the Internet for other cloud-based database alternatives, and right now [CouchDB](http://couchdb.apache.org) looks pretty promising, so I might try a completely different approach using that instead.

There is a lot to do here as well: define the data model, set up the web service, define a format for saving the polygon data, etc.
At the very least, I will need to manage:

- Model > metadata, rooms.- Room > metadata, model, furniture, level, polygon.- Furniture > metadata, room, polygon, transformation (initially identity).

Round tripping the information will be interesting, especially if I want the Revit model to update automatically when furniture positions are modified on the mobile device, which I certainly do, if only for the wow effect :-)

Stay tuned...

... Oh, and, yeah, wish me luck!

#### Extensible Storage Data Use in Schedules

Returning to the desktop again, here is a recent question from a developer on using estorage data in a schedule that leads to some general observations on that topic:

**Question:** I would like to display my extensible storage information in a schedule.
I searched high and low, though, and was unable to find any samples or documentation on the subject.
Can you help?

**Answer:** There is a good reason why you have not found anything: there is no such support.

The usage of extensible storage data is utterly and completely up to the add-in itself.
The Revit API offers no support whatsoever for making use of estorage data in any way.
The only support provided is for storage and retrieval, nothing else.

Well, one tiny little detail more: if an element id value is stored, and the element id is later modified by some worksharing operation, its value will be automatically updated.

On the other hand, there is currently no direct support for putting data into a schedule either.

I am no user interface expert, but I think that you can set up a schedule to be populated from certain parameter values via the user interface.
You would have to talk with an application engineer or product support or explore this topic on the Internet to find out how that can be done.

Of course, there is nothing stopping you from extracting your data from extensible storage and transferring it into parameters set up to populate a schedule yourself.

I hope that answers your question and helps you get started successfully.