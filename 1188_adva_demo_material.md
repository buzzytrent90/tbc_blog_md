---
post_number: "1188"
title: "View and Data API Presentation Material"
slug: "adva_demo_material"
author: "Jeremy Tammik"
tags: ['doors', 'geometry', 'levels', 'references', 'revit-api', 'selection', 'sheets', 'views']
source_file: "1188_adva_demo_material.htm"
original_url: "https://thebuildingcoder.typepad.com/blog/1188_adva_demo_material.html"
---

### View and Data API Presentation Material

I presented the
[Autodesk View and Data API](http://thebuildingcoder.typepad.com/blog/2014/07/autodesk-view-and-data-api.html) at the
[Basel.js meetup](http://www.meetup.com/basel-js/events/192651262) yesterday evening.

Now I would like to share the material that I showed with you as well:

- [Appetiser demos](#2)
- [Introduction](#3)
- [Slide deck and notes](#4)
- [Curl shell scripts for authorisation, model upload and translation](#5)
- [Demo script](#6)
- [Resources](#7)
- [Holiday and vacation](#8)

This should provide a handy snapshot and point of reference for my colleagues as well :-)

#### Appetiser Demos

Before we even start talking about this, let's take a really quick look at a couple of samples that show what it is all about:

- [Frontloader tractor](https://s3.amazonaws.com/FastViewer/index.html?file=frontloader/0.svf) –
  Zoom, rotate, isolate, focus, explode, embedded data.- [Waltham office building](https://s3.amazonaws.com/FastViewer/index.html?file=Waltham/0.svf) –
    Large building model, explore structure.- [SAP](http://54.191.41.170/sapdemo) –
      Link to external database, double click headrest, look at the linked SAP database data displayed in a separate property palette, edit the price in the database view at the bottom, and watch the properties update immediately.- [Morgan steampunk](http://safe-reef-1847.herokuapp.com) –
        Huge model with highly customised UI.

#### Introduction – aka Stephen's Spiel

Note that the viewer is currently still in preview mode.

The general public will be able to request developer keys and make complete use it within the next few weeks.

There was a time when everybody ***knew*** the earth was flat. They ***knew*** that if you sailed far enough, you'd fall off the edge of the world. But things change, and by the fifteenth century it was generally accepted that the earth was a sphere. In 1492, Christopher Columbus tried to take advantage of this 3D world to find a westerly trade route to the East Indies. Instead he discovered the Americas. European settlers quickly followed him, and a few hundred years and several genocides later we ended up here – in Silicon Valley.

And then we programmers invented another world – the worldwide web. And just like our ancestors, we all thought it was flat. If you moved your mouse far enough it would fall off the edge of the screen. A few people thought this new world shouldn't be flat, but 3D on the web was hard – you had to download a huge client-side executable, or you had to write a lot of code – so most web developers were happy with their two-dimensional webpages. Just think for a moment how a seller on Amazon or eBay shows you what their product looks like – you have to click through a set of photos taken from different angles. Why can't I see that product in 3D – and zoom it and pan it and rotate it? And maybe even explode it to see what it looks like on the inside?

But the world is changing. Today, all the important browsers support WebGL natively (so there's no need for customers to download a nasty plugin), and we have robust libraries like THREE.js that greatly simplify creating 3D web apps. For the first time, it’s not only possible for us to create 3D content – but we can be confident that our customers can actually view it.

But 3D still isn't that easy. You have to design your model somewhere and then you have to translate it into a format that your WebGL-enabled app can understand. And that's where Autodesk comes in. Our goal is to make 3D on the web so easy that it’s a no-brainer. We are launching our new 3D viewer API, which consists of a state-of-the-art zero-client, WebGL viewer and a REST API that allows you to translate and view almost any 3D file format. Let’s see what it can do.

#### Slide Deck and Notes

Here is the Autodesk Web Services View and Data API presentation
[slide deck](http://thebuildingcoder.typepad.com/adva/2014-07/adva_jeremy.pdf),
shown at the Basel.js meetup on July 30, 2014, based on work by Daniel and Cyrille with some additional notes by Jeremy:

The Challenge – Big Data, 2D and/or 3D: Autodesk Large Model Viewer.

Add interactive 3D viewing to your web application.

Single pipeline for integrated interactive viewing, search and data: extend it, find it, see it.

The same pipeline feeds search and viewing.

Two APIs available:

- **REST** Server and Management API

- Authenticate using oAuth 2.0
- Upload and translate files
- Manage access rights

- **JavaScript** Web Client API

- Viewing technology based on Three.js
- Embed and control viewer in HTML5 applications
- Implement user interaction, access documents, manipulate objects, camera...

Empowered by
[WEBGL](https://developer.mozilla.org/en-US/docs/Web/WebGL) and
[Three.js](http://threejs.org).

3D First – focus is currently on getting the 3D functionality out there first.

- 3D Functionality

- Select, view properties, zoom, pan, orbit, isolate, focus, highlight
- Access to underlying 3D model, e.g. meshes and materials

- 2D Functionality

- Raster image – zoom and pan only
- Later: Vector graphics soon – select, view properties, zoom, pan, isolate, focus, highlight

Currently supported formats:

- dwg
- dwt
- dwf
- dwfx
- rvt
- iam
- ipt
- nwc
- nwd
- f3d
- fbx
- 3ds
- dae
- obj
- zip
- stl
- ifc
- ige
- iges
- igs
- 3dm
- asm
- catpart
- catproduct
- cgr
- dlv3
- exp
- g
- jt
- model
- neu
- prt
- sab
- sat
- session
- skp
- sldasm
- sldprt
- smb
- smt
- ste
- step
- stla
- stlb
- stp
- wire
- x\_b
- x\_t
- xas
- xpr
- cam360
- sim
- sim360

Getting started – server side REST server and management workflow:

- Register an account and create an application.
- Use the authentication API to get an access token. The access token is required for all other API calls.
- Create a bucket in which to store your objects.
- Upload an object to the bucket and request that it be translated into a "bubble" for the viewing service.
- At this point you have a uniform resource name (URN) for your object, which you pass to the viewing client.

Here are the steps in more detail:

- Step 1: Register and Create an Application
- List of Registered Applications
- Step 2: Obtain an Access Token
- Step 3: Create a Bucket
- Bucket Policy:

- Transient: persists for 24 hours
- Temporary: persists for 30 days
- Persistent: persists until deleted

- Step 4: Upload a Model File
- Response to Upload Request
- Determine the URN from the upload response: the URN is the Base64 encoded id.
- Step 5: Register the Model for Viewing
- Check Progress: you can start viewing the object as soon as some parts have a 'complete' status
- Retrieve Thumbnail Image

This whole sequence is demonstrated most effectively by the [curl shell scripts](#5) available from GitHub.

Getting Started – Client Side JavaScript

Compatibility Requirements: the A360 Viewer requires a WebGL canvas compatible browser, such as:

- Internet Explorer 11.0+
- Chrome 18.0+
- Opera 15.0+
- Firefox 4.0+
- Chrome on Android

Load URN in JavaScript Viewer.

Create a html5 page or web application.

Add references to CSS style sheet and JavaScript library:

```
  <link rel="stylesheet"
    href="https://developer.api.autodesk.com/viewingservice/v1/viewers/style.css"
    type="text/css">

  <script
    src="https://developer.api.autodesk.com/viewingservice/v1/viewers/viewer3D.min.js">
  </script>
```

JavaScript Client Side Extension

- Model hierarchy
- Metadata and properties
- Events
- Camera / Zoom / Navigation
- Access to geometry, textures...
- etc.

Questions and Answers:

- [Rendering optimisations](http://thebuildingcoder.typepad.com/blog/2014/06/technical-summit-day-1-and-removing-rvt-references.html#2)
- [JavaScript optimisations](http://thebuildingcoder.typepad.com/blog/2014/06/technical-summit-day-1-and-removing-rvt-references.html#3)
- **[Q]** Is this for anyone and everyone? – **[A]** Yes, sure, not just large complex models; everybody's models, cf. 3D printing, SAT, anything.
- **[Q]** Other use cases? – **[A]** Any 3D model; every situation where you can replace a bunch of stupid 2D photos or flat plans by something more real and interactive.

#### Curl Shell Scripts for Authorisation, Model Upload and Translation

Look at Cyrille's
[curl shell scripts](https://github.com/Developer-Autodesk/workflow-curl-view.and.data.api) that
provide a very practical, efficient, interactive and direct method to step through the entire authorisation, upload and translation process.

Here is an example of uploading, translating and viewing a Revit model test.rvt:

```
./viewerAPI auth
./viewerAPI bucketCreate my_bucket_name
./viewerAPI upload samples/test.rvt
./viewerAPI register test.rvt
./viewerAPI registerProgress test.rvt
./viewerAPI registerProgress test.rvt
./viewerAPI thumbnail test.rvt
sudo ./viewerAPI html test.rvt
```

Please refer to the GitHub
[readme](https://github.com/Developer-Autodesk/workflow-curl-view.and.data.api) for
a more detailed description.

#### Demo Script and Demos Embedded Everywhere

The viewer can be embedded in almost any web application.

The sample applications presented here include embedding in blogs, Facebook, TypePad, Sharepoint, mechanical models, AEC (architectural, engineering, construction), HVAC (mechanical equipment in buildings), Infraworks (infrastructure and GIS), database integration (SAP), etc.

Here are some notes from a demo held by Jim Quanci in the end of June:

Remember ahead of time that you need decent Internet access.

Attached is a recording where I walked through a typical View and Data web service internal demo including the URLs I used.

Note I have all these set-up as default tabs in chrome, as some of these take a minute to load.

These are also in the order I show them, starting with simple and getting gradually more complex.

And remember:

- Zero client
- HTML5 and JavaScript based (open source) – which means runs on most browsers (IE coming very soon) – and is easy to customize/extend/hack
- WebGL (and three.js for the 3D web fluent)
- Industry standard architecture – REST and JSON
- Streaming – so big models
- Integrates easily with external data
- 60+ file formats supported – so developers and customers avoid the messiness of having to worry about file formats – including support for lots of general industry formats like OBJ, FBX, DWG, SketchUp – and competitive formats like SWX, CATIA, NX, etc.

Here are the demo links:

- [Blog](http://through-the-interface.typepad.com/through_the_interface/2014/05/a-sneak-peek-at-the-new-autodesk-360-viewer.html
  )
- [Facebook](https://www.facebook.com/a360viewer
  )
- [TypePad](http://adndevblog.typepad.com/cloud_and_mobile/stephens-test-page.html
  )
- [Sharepoint](https://share.autodesk.com/IPG/CloudPlatforms/SitePages/Test%20Page.aspx
  )
- [Model](https://s3.amazonaws.com/FastViewer/index.html?file=frontloader/0.svf
  )
- [Architectural, Engineering, Construction, HVAC, Mechanical Equipment in Buildings](https://s3.amazonaws.com/FastViewer/index.html?file=Revit_Kitchen/0.svf
  )
- [Waltham – large office building](https://s3.amazonaws.com/FastViewer/index.html?file=Waltham/0.svf
  )
- [Infraworks model – city planning](https://s3.amazonaws.com/autodesk.viewingservice.viewers.prod/0.1.68/viewer3d.html?&file=https://s3.amazonaws.com/temporary-model-artifact-storage/11044/LMVGeneratorPlugin/proposals/master/model.svf
  )
- [SAP Database Integration](http://54.191.41.170/sapdemo2
  )

Finally, here is Philippe's new slick UI viewer
[nodeview/bootstrap](http://54.191.41.170:3003/nodeview/bootstrap) sample:

- Full JavaScript client/server
- Integration of multiple js UI libs
- Bootstrap, jquery layout, slickgrid, jsTree
- Load/upload of 2D and 3D models
- Save/load of named views in MongoDB

Transcript of the Demo:

1. startup chrome and shrink to get tabs
2. all tabs are pre-setup in the same order as the email
3. depends on audience which demos are shown
4. some of the tabs do not come up right away
5. figure out what is working beforehand, refresh and wait
6. plm360 tab has not worked for the last week – kill it
7. refresh some tabs now and then
8. sap demo one needs several refreshes
9. all tabs set up, check them all quickly, remove ones that don't work
10. often the demo is used for a MeetUp and you want to set up quickly
11. start with the tractor, not embedded, full screen canvas, zero client, html5, pure chrome, api is js, standard, mostly open source, pretty big model, source model is 2 GB
12. tractor has structure from mechanical design, metadata, often equally important as graphics
13. everything is built on the api and can be changed
14. go back to the full loader
15. sometimes escape-escape to clear selection
16. explode, mech design, assemblies, subassemblies, etc., and explode follows same order
17. next demo is car with metadata to demo search, find batteries, wheels
18. look at 2D drawings, not just 3D models
19. move on to architectural samples, e.g. autodesk waltham office
20. very large model, go to structure, look at plumbing, electrical
21. this is a very large model, we use streaming to show something right away in viewer
22. right click on an air handler, look at the details
23. the id is included, and guid, for linking to the model and other databases for permanent links with external data
24. go to office, see textures, chairs, focus on a table, round, third floor, data, guid
25. good combination of quality and performance
26. give me all the doors, for example
27. next: embedding
28. mostly you are building some kind of app
29. here is an example of embedding in SharePoint
30. explode, people like that
31. here is a Revit house, look at the materials and textures
32. you don't explode building, that is for mechanical, but let's do that here too
33. we saw SolidWorks, inventor, Revit
34. we support 50 file formats, both adsk and non-proprietary such as OBJ
35. here is a Typepad blog embedding, and this can be achieved in ten minutes
36. Facebook sample: this will be popular for students
37. here is a Facebook page with the viewer embedded, the shaver
38. the model is actually in DropBox, the viewer in Facebook
39. we are looking at federated data with different sources
40. currently mostly adsk servers
41. look at the external info presented here coming from a db on azure
42. here is other data from the model itself
43. this is powerful stuff: help access and relate to metadata
44. explode this too
45. so far the samples were adsk
46. here is a sample by a partner called coins in the uk, pilot partner
47. this is just a test, a hospital
48. show level 2, level 5, navigate, all columns, simply testing how to drive the view based on bits of metadata
49. search all the glazing
50. some underlying technology is three.js, open source library
51. this viewer is very hackable, you can do interesting things
52. revit ids for all glazing, to connect with external db
53. oauth is our security system, used widely in the web
54. here is another 2D sample
55. we can select vectors, relate metadata
56. I cannot yet select entities, we are working on that
57. you can flip the 2D to 3D also
58. this is all zero client
59. I am showing on chrome, it works on Firefox, currently not on ie11
60. we want to enable you to deliver this viewing everywhere you need it
61. protected by password or open, however you need it
62. on the mobile side, we are building sdks for iOS and android
63. this stuff works on mobile devices, but still a bit clunky lacking a mouse
64. here is a code sample showing how to hook it up with SAP
65. the chair is from inventor
66. double click to isolate
67. here is the sap data, straight out of an sap enterprise database online
68. change the pricing here to $500 in the sap data
69. return to the graphics pane, back out, go back to 5468, and here is the new updated pricing, $500
70. this demo was put together by Philippe in two days
71. he does have prior experience, but it was one of his first viewer samples
72. large scale urban planning, InfraWorks
73. here is a pretty big model of an apartment block
74. how do you get involved?
75. url
76. get started
77. request access key
78. documentation is partially there
79. rest and js api
80. supported file formats
81. details on each api call
82. security mechanism
83. how to register and process a model
84. here is everything you need to get started
85. I hope this motivates you to do so
86. last thing: here are the different file formats:
87. url, spreadsheet?
88. as a user, you do not have to worry about the format
89. you push it up and we convert it automatically
90. extra complications with compound models
91. also if you have files with supporting files
92. you need to know which file to push up
93. federated data coming oauth2 is security mechanism
94. source files are only stored temporarily, we do not keep the source file
95. you should feel comfortable making the viewable public
96. the source seed binary is destroyed
97. easy
98. zero client, rest based, js, html 5, use your favorite ui tools and libraries
99. one question: business model
100. we are currently thinking a certain number of free file uploads per month, e.g. x MB
101. if you exceed the limit, it may cost, e.g. $100 for a couple of GB
102. zero charge for viewing, view as much as you want, unlimited
103. encourage people to stick 3D models everywhere, Facebook, Typepad, anywhere
104. the web is pretty cool these days
105. html 5 gibes some flashiness and interactivity, but it is still rather flat
106. young folks are used to 3D games and are not impressed by a flat 2D world
107. time to change that

#### Resources

Here is an overview of the main resources:

- [Getting started material, registration and full documentation](http://developer.autodesk.com)
- [autode.sk/viewerapisamples](http://autode.sk/viewerapisamples) – demo code and [viewer sample applications](https://github.com/developer-autodesk) on GitHub
- [API Console](https://developer.autodesk.com/api-console) – test the View and Data API services online
- Autodesk [Cloud and Mobile DevBlog](http://adndevblog.typepad.com/cloud_and_mobile)
- [Autodesk Community](http://forums.autodesk.com)
  > [Web Services API](http://forums.autodesk.com/t5/Web-Services-API/ct-p/94)
  > [View and Data API discussion forum](http://forums.autodesk.com/t5/View-and-Data-API/bd-p/95)

As said, please note that the View and Data API is still in pilot program mode right now, so you cannot yet apply for an API key.
We expect that to come within the next week or two.

There you are.

That just about covers what we have right now.

I look forward to seeing wat you can do with all this once you have access to the developer keys.

Good luck, much success, and have fun!

#### Holiday and Vacation

By the way, tomorrow is the first of August, a public holiday in Switzerland, and I am on vacation next week, so I hope this will help keep you entertained in the meantime.

The public developer key access may even be live by the time I return.

I'll keep my fingers crossed for that.