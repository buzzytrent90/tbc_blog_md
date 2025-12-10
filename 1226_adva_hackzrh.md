---
post_number: "1226"
title: "Autodesk View and Data API Notes and Samples"
slug: "adva_hackzrh"
author: "Jeremy Tammik"
tags: ['elements', 'geometry', 'levels', 'revit-api', 'selection', 'sheets', 'views', 'walls', 'windows']
source_file: "1226_adva_hackzrh.htm"
original_url: "https://thebuildingcoder.typepad.com/blog/1226_adva_hackzrh.html"
---

### Autodesk View and Data API Notes and Samples

Here is a summary of my notes from three presentations on the Autodesk View and Data API given by Cyrille Fauvel and Philippe Leefsma, in the two introductory workshops at
[HackZurich](http://thebuildingcoder.typepad.com/blog/2014/10/hackzrh-fluelisee-memento-jobs-and-books.html) on Friday evening, October 10 and at
[HackaBxl](http://www.transformabxl.be/agenda/event/hackathon-open-data-brussels) in
Brussels on October 17.

Philippe also presented a very nice and absolutely minimal
[basic viewer sample in node.js](#3) demonstrating
how to implement a private web service to obtain the authorisation token without exposing your key and secret in your JavaScript source.

I will be making use of this material again in the coming days, for my Autodesk View and Data API class on Thursday at
[AU Germany](https://cluster.ems-secure.de/registrations/autodesk/au.2014/ems.registration.php) in
Darmstadt and at the Berlin Hackathon next weekend.

#### Autodesk View and Data API Presentation Notes

I already presented a bunch of stuff on the
[Autodesk View and Data API](http://thebuildingcoder.typepad.com/blog/about-the-author.html#5.46).

Here is an executive summary of the most important information to get started with it.
We used this for the recent hackathons in
[Zurich](http://thebuildingcoder.typepad.com/blog/2014/10/hackzrh-fluelisee-memento-jobs-and-books.html#2),
[Brussels](http://thebuildingcoder.typepad.com/blog/2014/10/brussels-hackathon-and-determining-pipe-wall-thickness.html#2),
and the upcoming one in
[Berlin](http://www.meetup.com/TechMeetups-Berlin/events/161213342).

The presentation is quick and efficient; the slides are discussed in just a couple of minutes, before diving into live source code demonstrations.

Please read the following notes in the context of the
[accompanying slide deck](http://thebuildingcoder.typepad.com/adva/2014-10/jeremy_tammik_adva.pdf).
By the way, this is the version I will be using for my presentation
*Autodesk View and Data API – Interaktives 3D Modell in beliebige Webseiten einbetten*
at
[Autodesk University Germany](https://cluster.ems-secure.de/registrations/autodesk/au.2014/ems.registration.php) in
Darmstadt on Thursday October 23.

- [Introduction](#2.1)
- [Samples](#2.2)
- [Main highlights](#2.3)
- [More Features](#2.4)
- [Resources](#2.5)
- [Two separate APIs](#2.6)
- [Server API – REST](#2.7)
- [Client API – JavaScript](#2.8)
- [Questions and Answers](#2.9)

#### Introduction

- We are from Autodesk.
- Traditionally, we have implemented model authoring tools.
- Now we are looking at optimal way to share models with others and use the rich data they contain.
- Complexity is growing, we have a huge number of components addressing all imaginable requirements, many of them open source – how to put all the puzzle pieces together optimally?
- We can use WebGL and Three.js to present any graphical data, either 2D or 3D.
- What is WebGL? – Three.js simplifies it.

#### Samples

- Look at the mechanical tractor sample, an Inventor model.
- Rotate, pan, zoom, easy intuitive manipulation with zero footprint.
- This is a pretty big CAD model consisting of numerous files,   > 1 GB.
- It is shrunk and streamlined to be viewed interactively in the browser.
- All the CAD model structure and properties are preserved.
- We understand the internal model structure, metadata, properties, how the model was built.
- The pane on the left side shows the engineering structure, e.g. parts, assemblies and subassemblies.
- A hierarchical tree, objects selectable either on screen or in the tree view.
- The selected element is highlighted and its properties are displayed.
- The smart explode engine understands and dissects subassemblies step by step, e.g. the cabin initially remains intact.
- All interaction performed here in the browser is coded in JavaScript.
- An architectural model, the Autodesk Waltham office.
- We understand and can use the internal model structure to select only a specific level or only HVAC MEP data, i.e. ducts and pipes.
- We can pick an individual element, e.g. a window, isolate it, examine its properties.
- The Morgan steampunk sample shows how other JavaScript UI libraries can be integrated.
- The Morgan model was created by the Morgan car company. We worked with them for the Geneva car expo and they let us have it.
- It also shows how to drive the navigation programmatically, e.g., highlight just the engine or other significant aspects.
- All JavaScript libraries can be used.
- Infraworks model representing infrastructure from GIS data.
- Car seat model demonstrating real time price update from a linked external SAP database.
- Multiple user interaction to share remote control over view and camera in real time.
  Simulate two users in two separate parallel Firefox windows.
  Based on a node.js implementation using socket io.

#### Main Highlights

- Zero install.
- Not only geometry, many other data, metadata.
- Public API, open source, based on three.js.

#### More Features

- Translate heavy CAD model to lightweight JSON streamable web objects.
- Streamed to the browser, you see initial geometry immediately, them metadata, then textures.
- The tractor model was 1 GB and consisted of many files, not useful for web.
- Take all of this, upload to a server, translate to JSON and stream it in little objects.
- Many file formats are supported.
- Long-term objective: support any 3D file format.
- You can still access and manipulate geometry in the viewer.
- You cannot modify original model, just the view.
- Rendering on client side using WebGL.
- Everything is happening through Chrome and JavaScript.
- Everything done interactively can also be driven programmatically.

#### Resources

- [Getting started and documentation](https://developer.autodesk.com).
- Programming examples on [GitHub](https://github.com/developer-autodesk) in all kinds of different programming languages.
- An [API console](https://developer.autodesk.com/api-console) for testing.
- The [cloud and mobile DevBlog](http://adndevblog.typepad.com/cloud_and_mobile) for questions and more info.

#### Two Separate APIs

1. Server side, to upload and translate.
2. Web client, a normal HTML5 web page using CSS and JavaScript.

#### Server API – REST

- The server API requires a few initial one-time steps to upload and translate the model:

- Authorisation
- Upload
- Translation

- You get a secret key pair to protect your data.
- The translation gives you the file id or URN.
- From then on all is on the client side.
- This is well illustrated by the documentation and the [curl shell scripts](https://github.com/Developer-Autodesk/workflow-curl-view.and.data.api) available from GitHub.

#### Client API – JavaScript

- The client side JavaScript API is most likely to be customised, e.g. in hackathon projects.
- It can be used to query model data, examine the model hierarchy, properties, handle model, camera, events, access underlying geometry and textures insert other transient objects, and much more.
- For instance, we can grab the internal id of a model element and use that to attach, display and modify other information coming for separate databases, like in the car seat model displaying prices defined in SAP.
- You can just embed viewer in your web page.
- That is all you need, just an iframe.
- Example on Kean's blog, also Facebook, etc.
- If you have your own website: create HTML5 page, add CSS and JavaScript, create div, initialise, must be div, not canvas.
- The viewer creates the canvas itself internally.
- Show the initialisation function, replace the XXX by your credentials.
- This function can be defined in a plugin, of course.
- Client side API enables all extensions, e.g. to take control of geometry, change properties, get unique id from elements to add more data from external db, etc.

#### Questions and Answers

- Cost, pricing, business model?

  – The viewer is and will remain free forever; the pricing for translation and storage is undefined.
  There may one day be a charge for translation, storage or both.
  If so, however, it will be in the range of cents.
- Access?

  – Viewer is totally open source and JavaScript.
  You have access to all the code.
  Only the translation part is Autodesk private.
  You upload the model to an Autodesk server to convert it to streamable JSON.
- Can I add my own data, metadata?

  – Yes, absolutely.
  It is not saved, this is a viewer only.
  You can add both geometry and properties.
- What is driving the backend server?

  – Background is AWS, C++.
- Is the JSON format published?

  – JSON format is not yet published because still in flux.
  Once it is finished, sometime next year, we might publish the JSON object format.
  It does not really matter.
- Offline access?

  – If you put all JSON on the local hard drive, you can load and view the models, but you lose the streaming aspects.
  If you load a file without streaming it, you have to wait until it completes.
- Notification and interaction?

  – You can get callbacks on selection, explode, camera change, property change, everything.
  All behaviour can be completely customised.
- Is the JSON format published?

  – JSON format is not published. It is still in flux.
  Maybe once it is finished, sometime next year.
  It does not really matter.

#### Philippe's Basic Viewer Sample

Philippe Leefsma presented a sweet client-server sample to demonstrate an easy solution to manage the credentials in a JavaScript viewer application.

The problem is that you do not wish to expose your secret consumer key and consumer secret.

On the other hand, the viewer client needs to use them to access your stored models.

The solution for this is to define a minimal server that provides the required credentials.

The server is called by the client via REST and returns a token without exposing the credentials.

The server can only be called by the client due to the
[same-origin security policy](http://en.wikipedia.org/wiki/Same-origin_policy).

The whole application just consists of four files implementing a node.js application:

- package.json
- server.js
- routes/api.js
- views/index.html

Some points of interest:

- Node.js server app.
- Minimum server.js.
- Generate auth token on server side.
- Serve a static web page.
- Using express and request modules for node.js.
- Code for a super simple REST API called /api.
- Credentials from registering web page.
- Show token – `http://localhost:3001/api/token`.
- Show model – `http://localhost:3001`.

The contents of the node.js application package are defined by package.json:

```
  {
    "name": "AdnViewerBasic",
    "version": "0.0.0",
    "private": true,
    "scripts": {
      "start": "node ./bin/www"
    },
    "dependencies": {
      "express": "~4.8.6",
      "body-parser": "~1.6.6",
      "cookie-parser": "~1.3.2",
      "morgan": "~1.2.3",
      "serve-favicon": "~2.0.1",
      "debug": "~1.0.4",
      "jade": "~1.5.0",
      "request": "*"
    }
  }
```

The server provides one single REST entry point, /api/token, to return an authoristion token, and implementes one view, views/index.html, setting up and displaying the viewer canvas:

```
  var api = require('./routes/api');
  var express = require('express');

  var app = express();

  app.use(express.static(__dirname + '/views'));
  app.use('/api', api);

  app.set('port', process.env.PORT || 3000);

  var server = app.listen(app.get('port'), function() {
    console.log('Server listening on port ' + server.address().port);
  });
```

The REST API call is implemented like this by routes/api.js:

```
  var express = require('express');
  var request = require('request');

  var router = express.Router();

  router.get('/token', function (req, res) {

    var params = {
      client_id: '********',
      client_secret: '********',
      grant_type: 'client_credentials'
    }

    request.post(
      'https://developer.api.autodesk.com' +
        '/authentication/v1/authenticate',

      { form: params },

      function (error, response, body) {
        if (!error && response.statusCode == 200) {
          var authResponse = JSON.parse(body);
          res.send(authResponse.access_token);
        }
      });
  });

  module.exports = router;
```

Cyrille Fauvel used a similar technique to implement the
[PoiPointer node.js REST server](https://github.com/PoiPointer/node.js).

The client view is defined by index.html, a minimal sample of setting up and displaying the viewer:

```
<html>
  <head>
    <title>ADN Viewer Basic</title>
    <script src="http://ajax.googleapis.com/ajax/libs/jquery/2.1.1/jquery.min.js"></script>
    <link type="text/css" rel="stylesheet" href="https://viewing.api.autodesk.com/viewingservice/v1/viewers/style.css"/>
    <script src="https://developer.api.autodesk.com/viewingservice/v1/viewers/viewer3D.min.js"></script>
    <script src="https://rawgit.com/Developer-Autodesk/library-javascript-view.and.data.api/master/js/Autodesk.ADN.Toolkit.Viewer.js"></script>
    <script>
      $(document).ready(function () {

        var adnViewerMng = new Autodesk.ADN.Toolkit.Viewer.AdnViewerManager(
          'http://' + window.location.host + '/api/token',
          document.getElementById('ViewerDiv'));

        adnViewerMng.loadDocument(
          "*X*u*m*k*2*u*2*q*W*0*z*v*y*v*m*l*3*6*W*u*T*w*j*3*j*w*T*t*T*u*D*u*z*v*2*h*C*k*2*=");
      });
    </script>
  </head>

  <body style="margin:0">
    <div id="ViewerDiv">
    </div>
  </body>
</html>
```

You can start it up on the command line and then check the results in the browser via your localhost at the specified port:

```
  $ node server.js
  Server listening on port 3000
```

---

# Cloud and Mobile

### Autodesk View and Data API Notes and Samples

By
[Jeremy](http://adndevblog.typepad.com/cloud_and_mobile/jeremy-tammik.html)
[Tammik](http://thebuildingcoder.typepad.com/blog/about-the-author.html).

I just published a summary of my notes from three
[presentations on the Autodesk View and Data API](http://thebuildingcoder.typepad.com/blog/2014/10/autodesk-view-and-data-api-notes-and-samples.html) given
by Cyrille Fauvel and Philippe Leefsma, in the two introductory workshops at
[HackZurich](http://thebuildingcoder.typepad.com/blog/2014/10/hackzrh-fluelisee-memento-jobs-and-books.html) on Friday evening, October 10 and at
[HackaBxl](http://www.transformabxl.be/agenda/event/hackathon-open-data-brussels) in
Brussels on October 17.

Philippe also presented a very nice and absolutely minimal
[basic viewer sample in node.js](http://thebuildingcoder.typepad.com/blog/2014/10/autodesk-view-and-data-api-notes-and-samples.html#3) demonstrating
how to implement a private web service to obtain the authorisation token without exposing your key and secret in your JavaScript source.