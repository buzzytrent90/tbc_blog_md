---
ai_optimized: true
author: Jeremy Tammik
complexity_score: 1.5
content_type: qa
optimization_date: '2025-12-11T11:44:15.419403'
original_url: https://thebuildingcoder.typepad.com/blog/1157_rvtva3c.html
post_number: '1157'
reading_time_minutes: 3
series: general
slug: rvtva3c
source_file: 1157_rvtva3c.htm
tags:
- doors
- elements
- family
- revit-api
- views
- windows
title: "RvtVa3c \u2013 Revit Va3c Generic AEC Viewer JSON Export"
word_count: 689
---

### RvtVa3c – Revit Va3c Generic AEC Viewer JSON Export

When you read this, I will already be sitting on the plane back from New York to Switzerland.

Before leaving, though, I wanted to add some more explanations on the extremely fruitful work we accomplished this weekend.

I am very happy and grateful that we spontaneously formed such a wonderful team and worked together so enthusiastically, pleasantly and effectively.

Most of all, I enjoyed the companionship, competence and professionality of my closest teammate Matt Mason of
[Imaginit Technologies](http://www.imaginit.com).

I am really looking forward to seeing where all the other exciting projects that we discussed will go, and of course most of all how this one will fare in the long run.

In the short
[AEC Hackathon](http://thebuildingcoder.typepad.com/blog/2014/05/aec-hackathon-from-the-midst-of-the-fray.html)
project description that I published yesterday, I mentioned our new three.js based open source AEC visualisation project
[**va3c** to view 3D building models](https://github.com/va3c) in
any web browser.

The entire
[va3c team](http://va3c.github.io/#team) ended
up proudly winning the second place in the Hackathon competition, and each project participant was awarded an
[Arduino](http://www.arduino.cc) as a prize.

Here are [more details](#2) on the
[RvtVa3c](https://github.com/va3c/RvtVa3c) Revit
va3c exporter project that I completed together with Matt in
a total effort of 2 \* 26 hours = 52 man hours.

The entire source code, Visual Studio solution and add-in manifest is provided in the
[RvtVa3c GitHub repository](https://github.com/va3c/RvtVa3c).

The other projects, especially the central viewer component fed by the Revit add-in component, can be copied or cloned from their respective own
[repositories](#5) listed
below.

#### Short RvtVa3c Project Description

Implement a Revit add-in external application, external command and custom exporter extracting information straight from the Revit graphics output pipeline and streaming it to a three.js scene version 4 JSON model file for consumption and display in the va3c AEC viewer, including support for meta-data and Internet hyperlinks.

#### Task List and Features

- Done:

- Properly handle instance transformation, e.g. doors and windows
- Properly handle colour and material
- Support transparency, e.g. window panes
- Get completely fed up with the buggy Microsoft System.Runtime.Serialization.Json.DataContractJsonSerializer class and switch to the more reliable [Json.NET](http://james.newtonking.com/json) component
- Add scaling to common viewer size, e.g. [(0,0),(20000,20000)]
- Implement the external application ribbon panel and button
- Implement element properties, i.e. metadata support
- Eliminate multiple non-element materials
- Prompt user for output file name and location
- Eliminate null element properties, i.e. useless JSON userData entries
- Fix the GitHub repository corrupted by adding an excessively large file exceeding the 100 MB GitHub size limit

- Todo:

- More optimisations to reduce file size
- More intelligent family instance and type reuse

I could spend hours discussing each one of the steps listed above, more hours than Matt and I spent implementing them.

I can also save some time and breath and let you explore them for yourself.

Suffice to say that the Revit add-in is up and running, reliably producing version 4 three.js rendering of both small and large Revit models, and that the va3c viewer and all the other different va3c exporters work reliably as well.

#### Main Challenge

The target JSON format was initially a moving target, oscillating between version 3 and 4, both of which are nowhere completely reliably defined, requiring hard-core reverse engineering of the viewer by extensive JavaScript debugging.

We settled for version 4 in the end, since 3 is announced soon to be deprecated.

#### Links to Related Va3c Projects

- [va3c project landing page](http://va3c.github.io/)
- [viewer](https://github.com/va3c/viewer) – HTML viewer
- [GHva3c](https://github.com/va3c/GHva3c) – Grasshopper va3c exporter
- [maxscriptVa3c](https://github.com/va3c/maxscriptVa3c) – 3DS Max va3e JSON Exporter
- [json](https://github.com/va3c/json) – JSON sample files
- [va3c](https://github.com/va3c/va3c.github.io) – web viewer for AEC models organisation
- [va3c viewer demo](http://va3c.github.io/viewer)