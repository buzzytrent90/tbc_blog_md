---
ai_optimized: true
author: Jeremy Tammik
complexity_score: 3.2
content_type: news
optimization_date: '2025-12-11T11:44:15.292991'
original_url: https://thebuildingcoder.typepad.com/blog/1094_clouds.html
post_number: '1094'
reading_time_minutes: 8
series: general
slug: clouds
source_file: 1094_clouds.htm
tags:
- elements
- geometry
- levels
- revit-api
- rooms
- views
title: Lots of Clouds
word_count: 1578
---

### Lots of Clouds

Let me recapitulate my cloud related explorations for my simplified 2D BIM editor, since I will be discussing it again at a web-based BIM workshop later this week.

Before getting to that, though, talking about cloud related topics reminds me of some others of completely different nature that are near and dear to me:

- [Clouds in La Palma](#2)
- [Cloud Atlas](#3)
- [Cloud-based simplified BIM editor](#4)
- [Autodesk cloud service overview](#5)
- [Updated RevitLookup on GitHub](#6)

#### Above, in and under the Clouds of la Palma

My hiking trip on La Palma was definitely on interesting study of clouds, from close and afar, inside and out.

Happily, I was mostly above them, e.g. wandering the sunny ridge around the
[Caldera de Taburiente National Park](http://en.wikipedia.org/wiki/Parque_Nacional_de_la_Caldera_de_Taburiente),
reaching its highest point at the
[Roque de los Muchachos](http://en.wikipedia.org/wiki/Roque_de_los_Muchachos),
looking back over the roiling clouds and the ridge in the east that we walked along for three days, with the
[Mount Teide](http://en.wikipedia.org/wiki/Teide) of
Teneriffa in the background far behind it, 90 km distant:

![Roque de los Muchachos and the eastern ridge surrounded by clouds](file:////j/photo/jeremy/2014/2014-01-24_la_palma/272_jeremy_los_muchachos_teide_cropped.jpg)

From there we descended into the depths of the Caldera itself and admired the same ridge from below, assailed by clouds as ever:

![La Caldera and the eastern ridge in clouds from below](file:////j/photo/jeremy/2014/2014-01-24_la_palma/404_la_caldera.jpg)

We emerged from the Caldera to cross the impressive volcanoes in the parque natural de
[Cumbre Vieja](http://en.wikipedia.org/wiki/Cumbre_Vieja) in the south, again above the clouds, looking back northwards at the ever cloud-filled and overspilling Caldera:

![The volcanoes of the Cumbre Vieja and la caldera in clouds](file:////j/photo/jeremy/2014/2014-01-24_la_palma/516_volcanoes_cumbre_vieja_cropped.jpg)

After spending the night up on the volcanoes, we were decisively chased away by more aggressive clouds backed up by strong and cold storm winds:

![Storm and clouds over the Cumbre Vieja](file:////j/photo/jeremy/2014/2014-01-24_la_palma/629_storm_and_clouds_over_cumbre_vieja_cropped.jpg)

Happily, we made it down to a warm and sunny beach to dry off and recuperate a bit before starting the long journey back home again.

#### Cloud Atlas by David Mitchell

Another highly recommended cloud related topic is the extremely impressive and wonderful book
[Cloud Atlas](http://en.wikipedia.org/wiki/Cloud_Atlas_%28novel%29)
by David Mitchell, recently and almost equally impressively converted to
[a film](http://en.wikipedia.org/wiki/Cloud_Atlas_%28film%29) by
Lana Wachowski, Andrew Wachowski and Tom Tykwer.
I absolutely loved them both.
I found the book revolutionary.
I never read anything like it.
The film is revolutionary as well, and an
[unbelievable feat](http://www.theguardian.com/film/2013/feb/24/cloud-atlas-review-philip-french).
Tykwer and the Wachowskis met for a week in isolation to hatch ideas, then visited the book author Mitchell to hear what he thought of them.
It helps to read the book first, though, to understand anything at all about what is going on in there.

#### Cloud-based Real-time Round-trip Simplified 2D BIM Model Editor

I'm attending a cloud workshop in the next few days, so I dug out the recordings from my Autodesk Tech Summit and Autodesk University presentation
[DV1736](https://events.au.autodesk.com/connect/sessionDetail.ww?SESSION_ID=1736) –
Cloud-Based, Real-Time, Round-Trip, 2D Revit Model Editing on Any Mobile Device.
Here is the class summary:

This class demonstrates real-time, round-trip editing of a simplified 2D rendering of an Autodesk Revit intelligent model on any mobile device with no need to install any additional software whatsoever beyond a web browser. How can this be achieved? A Revit software add-in exports polygon renderings of room boundaries and other elements such as furniture and equipment to a cloud-based repository that is implemented using an Apache CouchDB NoSQL database. On the mobile device, the repository is queried and the data rendered in a standard browser using server-side generated JavaScript and SVG. The rendering supports graphical editing, specifically translation and rotation of the furniture and equipment. Modified transformations are saved back to the cloud database. The Revit add-in picks up these changes and updates the Revit intelligent model in real-time. All of the components used are completely open source, except for Revit itself.

Here are direct download links to all the class materials:

- [Handout](file:////a/doc/revit/au/2013/doc/dv1736_2d_revit_model_editor_handout.pdf)
- [Slide deck](file:////a/doc/revit/au/2013/doc/dv1736_2d_revit_model_editor_slides.pdf)
- [AU recording](http://au.autodesk.com/au-online/classes-on-demand/class-catalog/2013/building-design-suite/dv1736)
- [Live recording](http://thebuildingcoder.typepad.com/room_editor_live/index.html)
- [Preview recording](http://thebuildingcoder.typepad.com/room_editor_preview/index.html)
- [RoomEditorApp – Revit add-in desktop component](https://github.com/jeremytammik/RoomEditorApp)
- [roomedit – CouchDB cloud database source code](https://github.com/jeremytammik/roomedit)

I already discussed all the details of all the different components here on The Building Coder, and that just about covers what I personally have to contribute right now to cloud and mobile applications interacting directly with Revit.

#### Autodesk Cloud Service Overview

Obviously, Revit itself is making more and more use of cloud-based services, and the offerings provided by Autodesk are growing in all directions as well;
here is a good overview website for all
[currently available Autodesk services](http://www.autodesk.com/360-cloud):
[autodesk.com/360-cloud](http://www.autodesk.com/360-cloud).

For a quick text-based overview, here is my list from the
[cloud & mobile platform web service API](http://thebuildingcoder.typepad.com/blog/2013/12/devdayau-chronicle-estorage-view-depth-sound-of-noise.html#4) presentation
given at AU:

- Identity

- Autodesk Single Sign-On Services

- OAuth – i
- Federated Sign-On

- Viewing

- Viewing Services
- Translation Services

- Creation

- AutoCAD 360 Services
- Fusion 360 Services
- InfraWorks Services
- ReCap Photo Services – ii
- AutoCAD Core Engine Services (ACES)

- Simulation

- Render Services
- Building Performance, Green Building Studio Services

- Collaboration (Social)

- Autodesk 360 (Pro) Services
- BIM 360 (Glue + Field) Services – ii
- PLM 360 Services

- Business

- Licensing Services
- Exchange

As always, Autodesk is interested in making APIs available for these services, just like all other products, to provide a strong platform for automation of the related tasks.
I added an 'i' to the ones already providing a deep and broad API, and an 'ii' to the ones already equipped with a maturing API.
The rest are still under consideration.

The existing APIs are not accessible yet, though, either, since we are still working on
[pilot programs](http://adndevblog.typepad.com/aec/2013/10/bim-360-glue-api-pilot-and-updated-samples.html) to
get them ready for public consumption.

#### Updated RevitLookup on GitHub

Finally, as you hopefully already know,
[RevitLookup now lives on GitHub](http://thebuildingcoder.typepad.com/blog/2013/10/revitlookup-on-github-and-invitation-to-collaborate.html),
also up in the cloud somewhere, so to speak, accessible to all.

This vastly simplifies collaboration, and you are heartily invited to participate in improving it.

One friendly supporter is Florian Schmid of
[SOFiSTiK AG](http://www.sofistik.com).

He future-proofed RevitLookup by removing compiler warnings, and significantly extended the functionality for snooping of geometry, FormatOptions and RevitLinkInstances.

Another step of future-proofing that I performed was to bump the years listed in the copyright notices from 2013 to 2014, and, for the first time, incrementing the build version number.

The current version right now is thus
[release 2014.0.1.0](https://github.com/jeremytammik/RevitLookup/releases/tag/2014.0.1.0).

You should probably always go for the most up-to-date
[master branch](https://github.com/jeremytammik/RevitLookup) when
downloading.

---

#### Autodesk 360 Platform Web Service API Overview

Here is an overview of the API coverage level currently provided by the different Autodesk 360 Platform Web Services, with the level API support ranging from a to d according to the following scale:

1. Deep and broad
2. Maturing
3. Starter
4. Medium to Long Term Plans

None of these are accessible, yet, though, since we are still working on pilot programs to get them ready for public consumption.

- Identity

- Autodesk Single Sign-On Services

- OAuth – i
- Federated Sign-On – iv

- Viewing

- Viewing Services – iii
- Translation Services – iv

- Creation

- AutoCAD 360 Services – iii
- Fusion 360 Services – iv
- InfraWorks Services – iii
- ReCap Photo Services – ii
- AutoCAD Core Engine Services (ACES) – iv

- Simulation

- Render Services – iii
- Building Performance, Green Building Studio Services – iii

- Collaboration (Social)

- Autodesk 360 (Pro) Services – iv
- BIM 360 (Glue + Field) Services – ii
- PLM 360 Services – iii

- Business

- Licensing Services – iv
- Exchange (sales + mktg) – iii

---

1. Deep and broad
2. Maturing
3. Starter
4. Medium to Long Term Plans

Most of these are not accessible yet, though, since we are still working on
[pilot programs](http://adndevblog.typepad.com/aec/2013/10/bim-360-glue-api-pilot-and-updated-samples.html) to
get them ready for public consumption.

- Identity

- Autodesk Single Sign-On Services

- OAuth – i
- Federated Sign-On – iv

- Viewing

- Viewing Services – iii
- Translation Services – iv

- Creation

- AutoCAD 360 Services – iii
- Fusion 360 Services – iv
- InfraWorks Services – iii
- ReCap Photo Services – ii
- AutoCAD Core Engine Services (ACES) – iv

- Simulation

- Render Services – iii
- Building Performance, Green Building Studio Services – iii

- Collaboration (Social)

- Autodesk 360 (Pro) Services – iv
- BIM 360 (Glue+Field) Services – ii
- PLM 360 Services – iii

- Business

- Licensing Services – iv
- Exchange (sales + mktg) – iii