---
ai_optimized: true
author: Jeremy Tammik
complexity_score: 2.9
content_type: qa
optimization_date: '2025-12-11T11:44:14.567814'
original_url: https://thebuildingcoder.typepad.com/blog/0775_revit_api_webcast.html
post_number: '0775'
reading_time_minutes: 3
series: general
slug: revit_api_webcast
source_file: 0775_revit_api_webcast.htm
tags:
- csharp
- doors
- elements
- filtering
- parameters
- revit-api
- views
- walls
title: Revit 2013 API Webcast Recording
word_count: 568
---

### Revit 2013 API Webcast Recording

The recording of the
[Revit 2013 API webcast](http://thebuildingcoder.typepad.com/blog/2012/05/revit-api-webcast-tomorrow.html) that
we held on May 17 is now available from the ADN DevTech
[Webcast Recordings Archive](http://www.adskconsulting.com/adn/cs/api_course_webcast_archive.php),
entering "Revit API" as the course topic, or
[directly from here](http://download.autodesk.com/media/adn/Revit2013APIWebcast_May17_2012.zip).

The 34 MB zip file includes:

- ReadMe.txt- Slide deck- Recording- Code Samples

The webcast addresses two main areas: an introduction to Revit programming for beginners, including a quick walkthrough of basic concepts, and a discussion of some of the new functionality provided by the Revit 2013 API, which will be of interest to both beginners and experienced developers.
The two parts are roughly equal in length.

The new API topics discussed have been selected to avoid too much overlap with the
[DevDays Online presentation on What's New in Revit 2013](http://thebuildingcoder.typepad.com/blog/2012/05/devdays-online-on-whats-new-in-revit-2013.html),
from which a recording is also available, so these two complement each other and should do quite a good job together to cover all important aspects of the Revit 2013 API.
This is the agenda of the second half of the webcast:

- New feature overview and DevDays Online recording- Add-in integration features in depth, Idling and external events- Analysis, simulation, MEP and Structure- New SDK samples- More News

For your convenience, here is the content of the readme file describing the sample code provided:

#### Webcast on Revit 2013 API

This folder includes the following materials:

- Recording – recording of the webcast and the Q&A collection.
  Click on the WMV file to view the recording.
  Answers to questions asked using the Q&A Console can be found in document file.- Webcast – powerpoint slides used in the recording.- Code Samples – code samples used for the demo. Please see below for notes on each sample.

#### About Code Samples Folder

This folder includes samples demonstrated in the webcast
presentation. Some samples that were not demonstrated during
the webcast but used in the presentation slides are also
included.

The sample code is available in both C# and VB in the directories SourceCS and SourceVB, respectively.
Both of these contain a Visual Studio solution file WebcastDemo.sln to compile and run it.

A sample Revit model Demo.rvt containing a few simple elements that can be used to
test the functionality demonstrated by the sample code is also included.
If you prefer, you may use any other Revit file.

#### Functionality Demonstrated by the Solutions

1. HelloWorldCmd – a bare-bones example of an external command.- HelloWorldApp – a bare-bones example of an external application.- CommandData – parameters of the execute method of the IExternalCommand interface.- SelectedElements – method to retrieve selected elements.- FilteredElements – use of FilteredElementCollector to identify all Walls that exceed a certain length.- AllElements – use of FilteredElementCollector to retrieve all elements in the model.- ElementIdentification – method to identify the type of an element.- AllWalls – use of FilteredElementCollector to retrieve all walls in the model.- AllDoors – use of FilteredElementCollector to retrieve all doors in the model.- ElementCreation – use of the Creation namespace and the static creation method to create a wall.- ElementModification – use of ElementTransformUtils to rotate a wall.- BuiltInParameters – method to retrieve built-in parameters.- SharedParameter – creation of a shared parameter and binding it to the Doors category.