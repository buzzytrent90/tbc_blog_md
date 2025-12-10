---
post_number: "1262"
title: "Back from The Conference Tour and Wrapping Up"
slug: "wrapping_up"
author: "Jeremy Tammik"
tags: ['elements', 'family', 'levels', 'revit-api', 'views', 'windows']
source_file: "1262_wrapping_up.htm"
original_url: "https://thebuildingcoder.typepad.com/blog/1262_wrapping_up.html"
---

### Back from The Conference Tour and Wrapping Up

Things have been a bit hectic the last couple of weeks, just like every December, what with Autodesk University and the DevDays Conferences.

I am still cleaning up various open issues and looking forward to settling down into Christmas real soon now.

Here are a couple of items to share from the last few days:

- [AU 2014 classes online](#2)
- [Family type preview images](#3)
- [InfraWorks 360 REST API](#4)
- [Moscow DevDays, Meetups and Hackathon](#5)
- [Milano JavaScript Meetup](#6)
- [I Love 3D](#7)
- [More Wrapping Up to Come](#8)

#### AU 2014 Classes Online

The classes from Autodesk University 2014 are now accessible online.

Here are links to the
[classes on demand](http://au.autodesk.com/au-online/classes-on-demand) and the
[search entry point](http://au.autodesk.com/au-online/classes-on-demand/search).

#### Family Type Preview Images

**Question:** I am writing a Revit add-in app.
Part of its functionality is to display images of families, furniture, windows, etc.

Can you point me to some code examples for this functionality?

**Answer:** You can simply call the Revit API ElementType.GetPreviewImage method.

Here is a really old example from May 2010 of
[reading a preview image](http://thebuildingcoder.typepad.com/blog/2010/05/get-type-id-and-preview-image.html).

It introduced a new external command CmdPreviewImage in The Building Coder samples, which nowadays live in the
[The Building Coder samples GitHub repository](https://github.com/jeremytammik/the_building_coder_samples) and
therefore have been continually updated for the current versions of Revit.

#### InfraWorks 360 REST API

I don't normally deal much with InfraWorks.

However, it was on my plate during the DevDays conferences as part of the AEC and infrastructure sessions, so here is a query on it that I happened to be involved in:

**Question:** We have been using InfraWorks since the first release and see a lot of potential in the product. We are very curious about the REST API and have a couple of questions about it:

- Is the REST API read-only or can the users edit data through the REST API? For example, what if a municipality has a model for a city plan and wants to gather comments about the plan from the citizens. It would then be great if we could, through the REST API, grab an object from the model, for example a surface, publish it, and make the attributes on the object editable to the user. We have been working with mapguide-rest and the GeoREST API for this type of editing and publishing.
- Will we be able to publish the model through a server or is it limited to Autodesk 360?

**Answer:** Thanks for reaching out. We are indeed releasing an InfraWorks 360 REST API, which is currently available as part of our Beta program. The API is used to read model data, such as lists and detailed element information. The models are published using the desktop client of InfraWorks 360, like any other model, by syncing it to the cloud, from where it becomes accessible through the API.

Here are some links with more information:

- [The InfraWorks 360 REST API at Autodesk University](http://adndevblog.typepad.com/infrastructure/2014/11/infraworks-360-rest-api-at-autodesk-university.html)
- [Importing alignments from InfraWorks 360 to Civil 3D](http://adndevblog.typepad.com/infrastructure/2014/10/importing-alignment-from-infraworks-360-to-civil-3d.html)
- [InfraWorks 360 REST API Tech Preview](http://adndevblog.typepad.com/infrastructure/2014/05/infraworks-360-rest-api-tech-preview-available.html)

Please get in touch with Augusto if you would like to use the InfraWorks 360 API so we can discuss and ensure we provide all you need.

#### Moscow DevDays, Meetups and Hackathon

In between wrapping up the past, I am also looking toward the future, planning the beginning of next year.

I will be going to Moscow for the
[Russian DevDays conference](http://autodesk-press.livejournal.com/169202.html) and
meetup on January 29 next year.
In a way, that is wrapping up as well, isn't it?
That will be the last conference in the worldwide series.

If you happen to be around during that period, I would love to see you there!

#### *I Love 3D* at the Next Milano JavaScript Meetup

I had a great time at the
[Milano DevDays conference](http://thebuildingcoder.typepad.com/blog/2014/12/last-western-european-devdays-links-textures-ur4-vs-r2.html) and
[meetup event](http://thebuildingcoder.typepad.com/blog/2014/12/milano-meetups-and-my-new-nfc-business-card.html) last week.

The other meetup participants were pretty thrilled as well, so we went right ahead and organized another meetup there at
[Milano JS – il DOM di Milan!](http://milanojs.com) on
February 10 next year.

Here is the latest version of my blurb for that:

#### I Love 3D

For web designers and software developers, we will show how to make 3D models on the web much more immersive and engaging. We'll also check out 'VR on the cheap', real virtual reality using Google Cardboard.

You may have created fantastic 3D designs. Isn't it frustrating that your website is all in 2D? Now you can also easily embed interactive, intelligent, truly stunning 3D models into your web page, web application, or desktop application.

WebGL is a powerful technology for enriching the web with interactive 3D graphics. How do we get it into the hands of the people, provide easy general access to everyone? I'll walk through all aspects of publishing and delivering 3D models on the web through a simple easy pipeline, providing a powerful 3D web graphics tool for programmers, designers and artists alike. Based on
[WebGL](https://www.khronos.org/webgl) and
[three.js](http://threejs.org),
it enables you, your friends and your customers to interact with the 3D model and query all the element metadata.

I also demonstrate how this engine can be elevated to the next level by delivering a stereoscopic interface to 3D models through a simple web page and
[Google Cardboard](https://www.google.com/get/cardboard) VR with no need to invest lots of money in complex equipment or hours of time preparing and publishing your model.

For an entry point to get started and learn more about the Autodesk 3D web technology, check out
[developer.autodesk.com](http://developer.autodesk.com).

See a full demo app at [autode.sk/m3w](http://autode.sk/m3w).

For more details on this topic, please refer to the
[Autodesk View and Data API](http://thebuildingcoder.typepad.com/blog/about-the-author.html#5.46) topic group.

By the way, if you are not aware of Kean's series of posts on the Google Cardboard stereoscopic 3D interface, you should definitely take a look right now:

- [Gearing up](http://through-the-interface.typepad.com/through_the_interface/2014/10/gearing-up-for-the-vr-hackathon.html)
- [Part 1](http://through-the-interface.typepad.com/through_the_interface/2014/10/creating-a-stereoscopic-viewer-for-google-cardboard-using-the-autodesk-360-viewer-part-1.html)
- [Part 2](http://through-the-interface.typepad.com/through_the_interface/2014/10/creating-a-stereoscopic-viewer-for-google-cardboard-using-the-autodesk-360-viewer-part-2.html)
- [Part 3](http://through-the-interface.typepad.com/through_the_interface/2014/10/creating-a-stereoscopic-viewer-for-google-cardboard-using-the-autodesk-360-viewer-part-3.html)
- [Speech recognition](http://through-the-interface.typepad.com/through_the_interface/2014/10/adding-speech-recognition-to-our-stereoscopic-google-cardboard-viewer.html)

#### More Wrapping Up to Come

Tomorrow is my last day for wrapping up before the break.
Still lots to do...

Oh yes, and after wrapping up the work related items, I have to get going on Christmas presents, too!