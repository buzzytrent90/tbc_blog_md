---
ai_optimized: true
author: Jeremy Tammik
complexity_score: 1.7
content_type: qa
optimization_date: '2025-12-11T11:44:13.868752'
original_url: https://thebuildingcoder.typepad.com/blog/0391_open_ole_storage.html
post_number: 0391
reading_time_minutes: 3
series: general
slug: open_ole_storage
source_file: 0391_open_ole_storage.htm
tags:
- levels
- python
- references
- revit-api
- views
title: Open Revit OLE Storage
word_count: 665
---

### Open Revit OLE Storage

One of the valuable spoils of the AEC DevCamp last week in Waltham is the following stand-alone utility created by David S. Echols of
[H&A Architects & Engineers](ha-inc.com) for
opening and analysing the contents of the Revit structured OLE storage file streams.

We initially discussed this internal RVT and RFA file structure when presenting the Python utility to read the
[RVT or RFA file version](http://thebuildingcoder.typepad.com/blog/2008/10/rvt-file-version.html) without
starting up and loading the file into Revit.
It is also of interest to read the
[RVT and RFA thumbnail image](http://thebuildingcoder.typepad.com/blog/2009/06/rvt-and-rfa-thumbnail-image.html) stored
in the file.

Now Dave has submitted this great little utility that goes much further in reading and parsing the information available in the structured file data, providing access to the thumbnail image and items such as the following from the sample file rac\_advanced\_sample\_project.rvt provided with Revit Architecture 2011:

- DocType: Project- WorkSharing: NotEnabled- IsCentralFile: False- UserName: tammikj- CentralFilePath:- RevitBuild: Autodesk Revit Architecture 2011 (Build: 20100208\_2115)- Product: Architecture- Platform: x86- BuildTimeStamp: 20100208\_2115- LastSavedpath: C:\My Documents\BIM\_UX\_Team\_Projects\_2011\First Exp FT\Advanced Projects\RTM Files\rac\_advanced\_sample\_project.rvt- OpenWorksetDefault: 3

Here is an example of the result of parsing the RAC sample file rac\_basic\_sample\_project.rvt:

![OpenRevitOleStorage utility showing structured file storage data from rac_basic_sample_project.rvt](img/OpenRevitOleStorage.png)

Here is Dave's description of this utility and the associated source code:

Attached is a zip file of my OLE Storage test project.
It does not have any references to the Revit API assemblies, so you should be able to extract the files, open the project and run it.
Select the file you want to open and the BasicFileInfo will be displayed in the Text area.
If there is a preview image, it will be displayed to the right of the text area.
This is course provided as-is.

Here is
[OpenRevitOleStorage.zip](zip/OpenRevitOleStorage.zip)
containing the complete source code and Visual Studio solution for this stand-alone utility.

Obviously, as Dave points out, you make use of this utility or the provided code at your own risk.
Its use is completely unsupported, has nothing to do with the Revit API, and can change at any time.

#### Addendum

Here is a question that came up just a couple of days later, so I thought I would add it to this post, for completeness' sake:

**Question:** Here is a longstanding issue I have had with Revit – how to test externally for file version?

We automate some processes and start Revit to do it, therefore it would be very helpful to understand what version the file is that we are attempting to open!

One problem with Revit is that there is no way to determine the version of a Revit project file outside (or inside) of Revit.
On opening the file, Revit immediately upgrades an older file with no way to stop the process from happening.
We even thought of storing information in an external SQL database about what version of Revit each file is utilizing. We moved away from a similar method with AutoCAD projects, though, because of the issues it presented.

**Answer:** A Revit project is stored as a compound storage file, which consists of streams and storages. Storages are like folders, and can contain their own substreams and substorages.

At the top level, as of a few releases ago, one stream is BasicFileInfo. If interpreted as Unicode, it is roughly human-readable and lists some information about the model. This includes the build of Revit used to save the model.

This information was added so that out own support and development teams could quickly take a peek at a model and see what build it was saved in, and also whether it has worksharing enabled, since that information is also stored.