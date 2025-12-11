---
ai_optimized: true
author: Jeremy Tammik
complexity_score: 3.8
content_type: qa
optimization_date: '2025-12-11T11:44:15.510694'
original_url: https://thebuildingcoder.typepad.com/blog/1207_adva_webinar.html
post_number: '1207'
reading_time_minutes: 4
series: general
slug: adva_webinar
source_file: 1207_adva_webinar.htm
tags:
- elements
- levels
- parameters
- revit-api
- schedules
- selection
- views
title: Autodesk View and Data API Webinar
word_count: 860
---

### Autodesk View and Data API Webinar

I listed a whole bunch of
[upcoming events](http://thebuildingcoder.typepad.com/blog/2014/08/upcoming-event-calendar.html) last
week.

Now I have yet another one to announce, a [webinar](#3) introducing the Autodesk View and Data API, immediately preceding the [exchange apps hackathon](#2).

#### Autodesk Exchange Apps Hackathon – September 20-21, 2014

There are less than two weeks left before the
[Autodesk Exchange Apps Hackathon](http://thebuildingcoder.typepad.com/blog/2014/08/autodesk-exchange-apps-hackathon.html),
a two-day virtual web based coding 'festival' that will run from September 20 through to September 21, 2014.

![Autodesk Exchange Apps Hackathon 2014](img/exchange_apps_hackathon_2014.jpeg)

This year's event is bigger and better than last year's, with new competitions and great prizes.

We've assembled a great line-up of speakers to give presentations and demos on hot topics that will help your overall app development, and technical experts will be on hand to help and guide you.

For full details and registrations, please visit
[www.exchangeapphack.com](http://exchangeapphack.com).

Please email
[exchangeapphack@autodesk.com](mailto:exchangeapphack@autodesk.com) if
you have any questions on the Hackathon.

#### Autodesk View and Data API Webinar – September 18, 2014

We already looked at several aspects of the Autodesk View and Data API in the past few weeks:

- [Autodesk View and Data API](http://thebuildingcoder.typepad.com/blog/2014/07/autodesk-view-and-data-api.html)
- [Basel.js Meetup View and Data API Demo](http://thebuildingcoder.typepad.com/blog/2014/07/baseljs-meetup-view-and-data-api-demo.html#2)
- [View and Data API Presentation Material](http://thebuildingcoder.typepad.com/blog/2014/07/view-and-data-api-presentation-material.html)
- [Three.js AEC Viewer Progress on Two Fronts](http://thebuildingcoder.typepad.com/blog/2014/08/threejs-aec-viewer-progress-on-two-fronts.html)

The View and Data API
[is now public](http://forums.autodesk.com/t5/view-and-data-api/tell-me-when-this-api-is-public/m-p/5150974#U5150974),
and we invite you to attend a one-hour webinar introducing the View and Data API Beta in all its glory on Thursday September 18, 2014 at 8 am PDT.

In case you are not already aware of it, the View and Data web service API is a zero-client, browser-based viewer and associated (server-side) translation engine that will run in any WebGL-enabled browser. The viewer currently supports over 60 widely accepted design and 3D file formats. One advantage of the View and Data API is that the viewer has access to the rich data embedded in the original design, including properties, parameters, inter-object relationships, materials, etc., and that models are streamed, so you can start interacting with a very large model without waiting for the entire file to be downloaded from the server.

APIs include a server-side REST API for uploading, translating and accessing your 3D models, and a client-side JavaScript API that allows fine control of the viewer itself, including:

- Responding to events, e.g. selection and view changes.
- Searching and displaying the rich data included in the model.
- Manipulating the camera and view.
- Isolating and highlighting model components based on model data search.
- Loading 3D or 2D model representations.
- Linking 3D model objects to your own data sources.

Webinar details:

- Date: Thursday, September 18, 2014
- Time: 8 a.m. PDT – 4 p.m. BST – 5 p.m. CET – 11 a.m. EDT
- Duration: approximately one hour

Registration: To reserve your place, send an email to
[adn-training-worldwide@autodesk.com](mailto:adn-training-worldwide@autodesk.com).

Please make sure you include:

- Your full name
- Company name
- ADN Developer ID, if available
- Email address

We look forward to talking with you in the webcast.

If you would like to dive in right away instead of waiting for the webinar, you are welcome to apply for an API key at
[developer.autodesk.com](https://developer.autodesk.com) and
start experimenting beforehand.
Note that the API keys are still approved manually, so please be patient waiting for your approval.

#### Toggling Annotation Categories per View

Let's address one quick little pure Revit API issue before closing again:

**Question:** What's the right way to tell the difference between a panel and a switchboard using the API?
For example, in the basic MEP sample project, how do I tell the difference between SWB and MDP-1?

**Answer:** The class name and category of the two objects are indeed identical.

Therefore, you have to search for some additional differentiating characteristics.

The RevitLookup database exploration tool provides a useful utility for doing so.

Using that, I noticed that the panel has an INSTANCE\_SCHEDULE\_ONLY\_LEVEL\_PARAM built-in parameter, which is not present on the switchboard.

You might be able to use this to distinguish between the two.

I would always recommend implementing a test suite, though, to ensure that all the various assumptions like that made by your add-in really remain valid for all of the models you work with.

You may discover that different models and different workflows introduce different sets of parameters on the various elements.