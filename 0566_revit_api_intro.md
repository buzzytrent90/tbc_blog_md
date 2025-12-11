---
ai_optimized: true
author: Jeremy Tammik
complexity_score: 2.9
content_type: news
optimization_date: '2025-12-11T11:44:14.186834'
original_url: https://thebuildingcoder.typepad.com/blog/0566_revit_api_intro.html
post_number: '0566'
reading_time_minutes: 2
series: general
slug: revit_api_intro
source_file: 0566_revit_api_intro.htm
tags:
- elements
- geometry
- revit-api
title: Ritchie's Revit API Introduction
word_count: 402
---

### Ritchie's Revit API Introduction

We have already seen a couple of interesting contributions by Ritchie Jackson of the
[Adaptive Architecture and Computation](http://www.aac.bartlett.ucl.ac.uk)
programme at UCL, the
[University College London](http://en.wikipedia.org/wiki/University_College_London):

- [Blends, Hermite splines and derivatives](http://thebuildingcoder.typepad.com/blog/2010/11/blends-hermite-splines-and-derivatives.html)- [Complexity versus constructability](http://thebuildingcoder.typepad.com/blog/2010/11/complexity-versus-constructability.html)- The Autodesk [Project Vasari API](http://thebuildingcoder.typepad.com/blog/2010/11/project-vasari-api.html)- [Flattening a non planar extrusion profile](http://thebuildingcoder.typepad.com/blog/2010/12/flatten-a-non-planar-extrusion-profile.html)

Here is another topic, an introduction to the Revit API for programming novices for the
[London Revit User Group](http://www.lrug.org.uk) LRUG.
Knowing Ritchie, it includes a couple of novel aspects:

- Focus on
  [VSTA](http://thebuildingcoder.typepad.com/blog/2010/12/vsta-to-stay-and-updater-to-go.html#1) macros.- Comparison of manual versus API approach.- Focus on geometry generation.

The source code snippets cover the generation of some extremely simple shapes whilst the sample projects Ritchie presents feature the API workflow in more advanced examples:

- A single line.- A simple box extrusion.- A box with a cut-out.

![A box with a cut-out](img/ritchie-api-intro-box-cutout.png)

- Iteration to generate a series of boxes with cut-outs to represent a boardwalk.

![Boardwalk](img/ritchie-api-intro-boardwalk.png)

- An amphitheatre set-out:

![Amphitheatre](img/ritchie-api-intro-amphitheatre.png)

- A roller-coaster reception with all its elements and materials:

![Roller-coaster reception](img/ritchie-api-intro-reception.png)

- A curved truss with all its elements:

![Curved truss](img/ritchie-api-intro-truss.png)

- A conceptual high-rise model including
  - Façade set-out- Façade panels- Façade materials- Floor plates- Beams- Beam materials

![Highrise](img/ritchie-api-intro-highrise.png)

The presentation includes more details on all of these items, obviously, descriptions of the workflows, and a detailed quick start guide for creating VSTA macros.
Some of these samples were also mentioned in Ritchie's previous contributions.

Here is Ritchie's complete
[Revit API Introduction Presentation](zip/LRUG-01-API-RitchieJackson.pdf) with the accompanying
[Notes](zip/LRUG-01-API-RitchieJackson-Notes.pdf) and
[Source Code samples](zip/LRUG-01-API-Code-RitchieJackson.zip).
Very many thanks to Ritchie for sharing this with us!