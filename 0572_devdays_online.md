---
post_number: "0572"
title: "DevDays 2010 Online with Revit 2012 API News"
slug: "devdays_online"
author: "Jeremy Tammik"
tags: ['elements', 'filtering', 'geometry', 'references', 'revit-api', 'views', 'walls']
source_file: "0572_devdays_online.htm"
original_url: "https://thebuildingcoder.typepad.com/blog/0572_devdays_online.html"
---

### DevDays 2010 Online with Revit 2012 API News

Back in December, I mentioned some of the stations of our yearly world-wide
[Developer Days](http://thebuildingcoder.typepad.com/blog/2010/12/snow-and-woe-with-manifest-files.html) or
[DevDays](http://thebuildingcoder.typepad.com/blog/2010/12/filtered-element-collector-sample-overview.html)
[conference](http://thebuildingcoder.typepad.com/blog/2010/12/vsta-to-stay-and-updater-to-go.html)
[tour](http://thebuildingcoder.typepad.com/blog/2010/12/birthdays-and-gaps-in-shells.html).

Now the recordings of the final updated contents of these presentations are available to the general public.
They can be obtained from the Autodesk Developer Network (ADN) DevTech
[API Training & Consulting](http://autodesk.com/apitraining) page, following the link to the
[webcast archives](http://www.adskconsulting.com/adn/cs/api_course_webcast_archive.php).

To find the recording of the new features of the Revit 2012 API, created by my colleague Saikat Bhattacharya, please filter for 'Developer Days Online: Revit' and 'English' and select the
[download link](http://download.autodesk.com/media/adn/DevDay_Online-Revit_2012_API.zip) to obtain the archive file 'DevDays Online - Revit 2012 What's New.zip' containing the following:

- Recording – Developer Days Online - What's New in Revit2012 API.mov (99,581,820)- [Presentation](#1) – DevDays Online - Revit 2012\_Whats New.pptx (3,142,426)- [Sample code](#3) – API samples.zip (10,786,217)

#### Presentation

The presentation briefly discusses the new Revit 2012 product features before diving into the "rice" and "wine" of the API update.

The product features are related to the following Revit 2012 themes:

- Construction Modelling –
  More detailed geometric information to manage construction workflows
  - Wall layers- Subdivision of elements – quantification, construction- Slabs, monolithic concrete objects – split into component pieces- Analysis and Visualisation –
    API feature enhancements to display detailed environmental and element relationship information and more types of analysis results
    - Vector analysis- Non-element solids- Large Team Workflow – Access to worksharing information
      - Who's borrowing? Which elements? What changed?- Visualisation and colour schema – to aid collaboration- Point clouds – Interoperability
        - Field data, renovation- Challenges – large data, recognizing shapes- Multi-year project- MEP – Piping and Ducts
          - Place holder elements – well connected system without specific part info- Structure – Rebar improvements
            - 3D rebars – essential for multiple applications in building and civil

Here is a link to more detailed and in-depth
[Revit 2012 product feature demos](http://www.youtube.com/AutodeskBuilding).

#### Rice and Wine

As in the past, we separated the API enhancements into the rice and wine:
the rice has to do with making sure that your existing application runs in the updated environment, and the wine covers the opportunities to add value and make use of new features and APIs to increase the value of your and our products.

This part pretty closely matches the overview of the
[Revit 2012 API features](http://thebuildingcoder.typepad.com/blog/2011/03/revit-2012-api-features.html) that
I already provided.

Note that we already had a slightly closer look at some of the topics.

#### Revit 2012 API Samples

The samples include the following five Visual Studio solutions:

- AddRibbonTabs
  - Add custom ribbon tabs.- DevDay2010
    - Switch active view.- Geometry filter.- New geometry API.- Geometry2012
      - Updated version of the FindColumns SDK sample, using new geometry and AVF API features instead of FindReferencesByDirection to find and display all intersections between walls and columns.- MepPlaceholders
        - Demonstrate the use of the new MEP placeholder elements.- RevitApiLabs2012
          - Updated version of the Revit API training material, work in progress.

The samples alone include lots of interesting material for us to take a closer look at in the near future.

Meanwhile, I wish you lots of fun and many exciting ideas for new application possibilities while exploring the Revit 2012 API.