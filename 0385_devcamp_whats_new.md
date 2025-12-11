---
ai_optimized: true
author: Jeremy Tammik
complexity_score: 1.9
content_type: tutorial
optimization_date: '2025-12-11T11:44:13.859023'
original_url: https://thebuildingcoder.typepad.com/blog/0385_devcamp_whats_new.html
post_number: 0385
reading_time_minutes: 4
series: whats_new
slug: devcamp_whats_new
source_file: 0385_devcamp_whats_new.htm
tags:
- elements
- filtering
- levels
- parameters
- references
- revit-api
- schedules
- selection
- sheets
- transactions
- views
- windows
- whats_new
title: DevCamp Session on What's New
word_count: 729
---

### DevCamp Session on What's New

Yesterday we had a great start to the AEC DevCamp here in Boston with lots of great presentations and meetings, and ending with the traditional beer bust at the Autodesk offices in Waltham half an hour's bus ride away from the convention centre here in Boston.

Today is packed with more presentations, obviously, some of which I will hate to miss since there are multiple tracks and I cannot attend all of them simultaneously.
Some sessions will be presented by members of the Revit development team and include unique information and insights not available anywhere else.

I'll also present the first of my own sessions, on what's new in the Revit 2011 API and how to use it.
Several of the areas I cover have already been discussed here in the blog.
My main topics in that session will be:

- [My first Revit 2011 add-in](http://thebuildingcoder.typepad.com/blog/2010/05/pipe-to-conduit-converter.html)- [Migration of Revit 2010 add-ins](http://thebuildingcoder.typepad.com/blog/2010/04/plugin-migration-steps.html)- New Revit SDK sample applications- [Idling event](http://thebuildingcoder.typepad.com/blog/2010/04/idling-event.html) and samples
        - [RevitWebcam](http://thebuildingcoder.typepad.com/blog/2010/06/display-webcam-image-on-building-element-face.html)- RST live link- RME loose connectors- Learning More

I plan to publish details on the new Idling event samples here as well as soon as I can get to it.
Right here and now, let me at least complete the overview of the new SDK samples.

#### New SDK Samples in Revit 2011

I have already mentioned a number of the new SDK samples provided in the Revit 2011 SDK in previous posts, and in some cases described their use and implementation in detail as well, but I notice that a complete overview listing them all is still missing so far.

So here it is.
The following samples are completely new:

- AnalysisVisualizationFramework
  - [DistanceToSurfaces](http://thebuildingcoder.typepad.com/blog/2010/04/element-level-events.html)- [SpatialFieldGradient](http://thebuildingcoder.typepad.com/blog/2010/06/display-webcam-image-on-building-element-face.html)- ConceptualDesign
    - DividedSurfaceByIntersects- ElementFilter
      - ViewFilters- [FindReferencesByDirection](http://thebuildingcoder.typepad.com/blog/2010/01/findreferencesbydirection.html)
        - [FindColumns](http://thebuildingcoder.typepad.com/blog/2010/01/findreferencesbydirection.html)- [MeasureHeight](http://thebuildingcoder.typepad.com/blog/2010/01/findreferencesbydirection.html)- Massing
          - ParameterValuesFromImage- PointCurveCreation- DirectionCalculation- [DocumentChanged](http://thebuildingcoder.typepad.com/blog/2010/04/element-level-events.html)
              - Also known as ChangesMonitor- [DynamicModelUpdate](http://thebuildingcoder.typepad.com/blog/2010/04/element-level-events.html)- [ErrorHandling](http://thebuildingcoder.typepad.com/blog/2010/04/failure-api.html)- [ExternalCommand2011](http://thebuildingcoder.typepad.com/blog/2010/04/revitaddinutility.html)- [MaterialQuantities](http://thebuildingcoder.typepad.com/blog/2010/02/material-quantity-extraction.html)- [PanelSchedule](http://thebuildingcoder.typepad.com/blog/2010/05/the-revit-mep-2011-api.html)- Selections- SolidSolidCut

I added links to existing blog posts on the ones I already discussed.

We also have these three SDK samples which existed in previous incarnations and were rewritten from scratch because the underlying API functionality was completely changed:

- HelloRevit- ImportExport- TransactionControl

#### SDK Sample Spreadsheet

I also updated my
[Revit SDK sample spreadsheet](zip/Revit_2011_SDK_Samples.xlsx) which
provides an overview and classification of all SDK samples into various categories.
I initially created it to keep track of all the external command registration data required for Revit.ini entries to load the samples, and to automatically generate the text file read by the first incarnation of
[RvtSamples](http://thebuildingcoder.typepad.com/blog/2008/09/loading-sdk-sam.html)
to handle all of them in one go.
Nowadays RvtSamples in included in the SDK itself, and the development team creates the text file to drive it, but I still keep my spreadsheet up to date.
I also used this way back for the DE101-3 Revit SDK Sample Smörgåsbord presentation at
[AU 2008](http://thebuildingcoder.typepad.com/blog/2008/10/au-2008.html).
Anyway, here is the updated
[version for Revit 2011](zip/Revit_2011_SDK_Samples.xlsx).

Time for breakfast.
I am on the nineteenth floor looking out at another wonderful day outside the window with a beautiful view northward over the Charles River, Cambridge and the MIT, a huge and completely cloudless blue sky after heavy thunderstorms in the past days.
Painful to spend a day like this cooped up in a convention centre, however exciting the topic may be...