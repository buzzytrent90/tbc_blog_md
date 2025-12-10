---
post_number: "0504"
title: "Modeless Door Lister and Deleter"
slug: "list_project_doors"
author: "Jeremy Tammik"
tags: ['csharp', 'doors', 'elements', 'revit-api', 'views']
source_file: "0504_list_project_doors.htm"
original_url: "https://thebuildingcoder.typepad.com/blog/0504_list_project_doors.html"
---

### Modeless Door Lister and Deleter

I arrived safely back home after taking a tranquil and uneventful train ride from Milano through Switzerland.
No more airplanes or airports for me this year!
There was actually not all that much snow in sight, and no disturbances whatsoever.
Now I have a couple of days to try to pick up the threads of my normal life again, both work and private, after a full month of nonstop conferences and travelling.

Here is a pretty little sample application by Agnius Vasiliauskas, who also runs a very interesting blog named
[Coding experiments](http://coding-experiments.blogspot.com) covering
a number of programming problems in various languages, with a lot of solutions especially around mathematical Euler problems, image manipulation and analysis.

Agnius made his first excursion into the Revit API, with the following goals:

- Create a Revit add-in with the following features:- Display a modeless form populated with all door instances in the project.- Show a preview image of each door in the form.- Add an option to delete each door in the DataGrid context menu.

Agnius adds:
I want my work to be useful for everybody else who develops for Revit.
Because of this, I send you my work, with the hope that maybe you will find it useful too.

I created this add-in entirely from scratch based completely on the following publicly available information:

- The [DevTV recordings](http://thebuildingcoder.typepad.com/blog/2010/10/revit-2011-devtv.html) provided on the [Revit Developer Center](http://www.autodesk.com/developrevit).- The information provided in the Revit SDK, e.g. the Revit API help file, developer guide and samples.- Various posts on [The Building Coder](http://thebuildingcoder.typepad.com).

Here is a
[screen snapshot of the add-in form](http://oi51.tinypic.com/2gtoop2.jpg) to provide a quick preview:

![List project doors](img/ListProjectDoors.png)

Here is the complete
[Visual Studio 2010 C# project](zip/ListProjectDoors.zip) for
this Revit add-in as well.

Some of the solutions incorporated into the project are the
[modeless dialogue handling](http://thebuildingcoder.typepad.com/blog/2010/07/modeless-loose-connectors.html) using the
[Idling event](http://thebuildingcoder.typepad.com/blog/2010/04/asynchronous-api-calls-and-idling.html) to
communicate with Revit and the
[ElementType preview image](http://thebuildingcoder.typepad.com/blog/2010/05/get-type-id-and-preview-image.html).
I totally agree that it can prove useful for others as well.

Agnius' project door lister is a really impressive example of what somebody with good programming knowledge can achieve using the Revit API in very little time, starting from absolutely zero.
Very many thanks to Agnius for sharing this!