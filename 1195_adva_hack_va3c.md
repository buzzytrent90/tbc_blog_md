---
ai_optimized: true
author: Jeremy Tammik
complexity_score: 2.4
content_type: tutorial
optimization_date: '2025-12-11T11:44:15.489469'
original_url: https://thebuildingcoder.typepad.com/blog/1195_adva_hack_va3c.html
post_number: '1195'
reading_time_minutes: 5
series: general
slug: adva_hack_va3c
source_file: 1195_adva_hack_va3c.htm
tags:
- elements
- geometry
- levels
- revit-api
- views
title: Three.js AEC Viewer Progress on Two Fronts
word_count: 1032
---

### Three.js AEC Viewer Progress on Two Fronts

Along with our everyday workload, my colleagues and I are busy playing with the View and Data API and the powerful
[WebGL based viewer](#2) that is on the verge of release.

Meanwhile, very significant progress has also been made on the public domain and open source
[vA3C three.js based AEC viewer](#4).

#### The Autodesk View and Data API WebGL Based Viewer

I mentioned that I presented the Autodesk View and Data API and viewer technology at the
[July basel.js meetup](http://thebuildingcoder.typepad.com/blog/2014/07/baseljs-meetup-view-and-data-api-demo.html).
My colleagues have been showing it off in many other places all over the world.

It is still not fully public.
You can request an API key today.
The request currently still needs to be manually confirmed.
If you are really serious about getting into this and gathering some experience right away, please let us know.
We can accommodate dozens of pilot testers, but not hundreds.

The fully public release and automated confirmation of API key requests is expected no later than September 8-10, the date of the upcoming
[apigee conference](http://iloveapis2014.com) in
San Francisco.

Another important milestone for me personally will be the
[hackzurich](http://hackzurich.com) hackathon,
'the largest hackathon Switzerland has ever seen', held October 10-12, which
[Philippe Leefsma](http://adndevblog.typepad.com/cloud_and_mobile/philippe-leefsma.html) and I will be attending.

Here is a blurb about our topic by
[Cyrille Fauvel](http://adndevblog.typepad.com/cloud_and_mobile/cyrille-fauvel.html):

#### Bring Complex 3D Models to WebGL in a Simple Way

WebGL is a very powerful technology for enriching the web with graphics. How can we get it into the hands of the creators? We discuss some aspects of building 3D models and how to bring them to life on the web through a smooth streamlined pipeline, providing coders, designers and artists alike a strong and powerful web graphics tool. In addition to easily bringing your 3D models to the web, the viewer (based on three.js) includes an API allowing close interaction with the 3D model and querying element metadata. We demonstrate how WebGL experiences created with the Autodesk View and Data API engine can be elevated to the next level by making 3D models even more immersive and engaging with practical hands-on demonstrations and samples.

So much for the exciting (near) future perspective from the corporate CAD world.

Now, for a look at the other end of the spectrum:

#### Significant Progress on the vA3C Project

As said, significant progress has also been made on the grass-roots, public domain and open source
[vA3C three.js based AEC viewer](https://va3c.github.io) that
was initiated at the
[AEC hackathon](http://core.thorntontomasetti.com/aec-technology-symposium-2014) in
New York in May.

vA3C is an open source, browser-based 3D model viewer for AEC models that uses three.js to render 3D geometry in the browser.
vA3C enables authors in the AEC industry to easily publish their 3D design work on the web, for free.

Here are my previous posts on the topic:

- [AEC Hackathon](http://thebuildingcoder.typepad.com/blog/2014/05/aec-hackathon-in-new-york-and-new-developer-guide-url.html)
- [AEC Hackathon – From the Midst of the Fray](http://thebuildingcoder.typepad.com/blog/2014/05/aec-hackathon-from-the-midst-of-the-fray.html)
- [RvtVa3c – Revit Va3c Generic AEC Viewer JSON Export](http://thebuildingcoder.typepad.com/blog/2014/05/rvtva3c-revit-va3c-generic-aec-viewer-json-export.html)
- [RvtVa3c Assembly Resolver](http://thebuildingcoder.typepad.com/blog/2014/05/rvtva3c-assembly-resolver.html)

The vA3C viewer has made huge progress since its initial implementation, mainly propelled by the inexhaustible Theo Armour, who also originally initiated the whole project.

The main new enhancements include:

- Huge collection of new sample models
- Editing geometry, lights, camera, etc.
- Adding models
- Export

The new capabilities mean that this project enables a poor man's Navisworks and more, for free, today: export 3D BIM models from any authoring software of your choice, combine them all into one view, edit the composite model as you please, access all metadata you like, link to any additional data sources of your choice, and export the result for further use.

Pretty cool, huh, for such a small team – mainly Theo, as far as I can tell – in such a short time – twelve weeks.

This goes to show many things, among others the power and flexibility of JavaScript, HTML5, and all the new cloud and web technologies.
You can achieve things in a matter of weeks that previously took years to accomplish.

For more information, please refer to the following documentation and samples:

- [Work-in-progress update: digging deep DOM into 3D models](http://www.jaanga.com/2014/07/va3c-viewer-work-in-progress-update.html)
- [HTML5 Read Me](http://va3c.github.io/viewer/va3c-viewer-html5/readme-reader.html)
- [Viewer Live Demo](http://va3c.github.io/viewer/va3c-viewer-html5/r3/va3c-viewer-html5-r3.html)
- [Source Code](https://github.com/va3c/viewer/tree/gh-pages/va3c-viewer-html5)

The future is looking bright, interesting and exciting, at least on this technical front.

Sadly, other areas are full of suffering.

![Death suffering from burnout](img/wenn-der-tod-burn-out-hat.jpg)

Syria, Iraq, Islamic Nation, Gaza, Libya, Ukraine, Boko Haram, Ebola...! I can't stand it anymore! – Typical burnout! You should take a break! – [zeitimblick](http://zeitimblick.info/wenn-der-tot-an-burnout-leidet)

#### Addendum – Letting Go of Suffering and Striving for Peace

The most important place I know of to start striving for peace is within my own self.

First of all, I recognize my own suffering and aggression.

As the Buddhists say, the root cause of suffering is
[clinging](http://en.wikipedia.org/wiki/Up%C4%81d%C4%81na) and rejection.

My main hope for transforming my own pain is self-observation and acceptance of what is.

I pray: [Lokah samastah sukhino bhavantu](http://en.wikipedia.org/wiki/Lokaksema_(Hindu_prayer)) – may all beings everywhere be happy and free.

[Continued...](http://thebuildingcoder.typepad.com/blog/2014/08/striving-for-personal-peace-continued.html).