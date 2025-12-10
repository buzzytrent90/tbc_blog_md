---
post_number: "1347"
title: "Revit Future and Saving User Configuration Settings"
slug: "user_config"
author: "Jeremy Tammik"
tags: ['elements', 'levels', 'revit-api', 'views']
source_file: "1347_user_config.htm"
original_url: "https://thebuildingcoder.typepad.com/blog/1347_user_config.html"
---

### Revit Future and Saving User Configuration Settings

I encountered the need and implemented solutions to save user configuration data several times over in the past.

Today, prompted by a recent query, I'll point out two of them, and some other exciting and interesting stuff as well:

- [Anthony Hauck on Futures for Revit](#2).
- [The Most Popular Programming Languages 2015](#3).
- [Saving User Configuration Settings](#4).

- [Text Format Configuration File Storage and Parsing](#4.1).
- [.NET XML Configuration File Storage and Parsing](#4.2).

#### Anthony Hauck on Futures for Revit

[BIMThoughts](http://bimthoughts.com) is a podcast platform about BIM technology and techniques.

Listen to the [half-hour BIMThoughts interview with Anthony Hauck](http://bimthoughts.com/s2e10), Director of Product Strategy at Autodesk, talking about what may or may not be coming in Revit’s future:

Your browser does not support the audio element.

#### The Most Popular Programming Languages 2015

[ProgrammableWeb](http://www.programmableweb.com) presents an interesting analysis of
[the most popular programming languages of 2015](http://www.programmableweb.com/news/most-popular-programming-languages-2015/elsewhere-web/2015/08/04):

![Popular programming languages 2015](img/popular_programming_languages.jpg)

Check out the [full article](http://www.programmableweb.com/news/most-popular-programming-languages-2015/elsewhere-web/2015/08/04) for
details on how this ranking was determined.

#### Saving User Configuration Settings

**Question:**
I used the .NET settings file, e.g., xxx.dll.config, to store user and application data.

Unfortunately, it does not work; manual modifications are ignored.

Apparently, it is only active at the application (.exe) level.

The project simply retains the default values for all class library projects.

I still can’t find a workaround up to this moment.

Do you have any suggestions how a Revit add-in can store external configuration data that can be modified by a user?

**Answer:**
Yes, definitely. Thank you for bringing this up.

There are a number of ways to address this, for two of which I can present ready-made implementations on GitHub:

- First, to be clear, let's rule out the usage of the top-level application configuration file revit.exe.config. That would be a very bad idea, for a large number of reasons.
- Implement your own [text format configuration file storage and parsing](#4.1).
- Make use of the [.NET XML configuration file storage and parsing](#4.2) functionality.

#### Text Format Configuration File Storage and Parsing

I already documented this approach when discussing the
[Berlin hackathon results, 3D Viewer and RvtVa3c update](http://thebuildingcoder.typepad.com/blog/2014/10/berlin-hackathon-results-3d-viewer-and-web-news.html),
in the section on
[custom user settings storage](http://thebuildingcoder.typepad.com/blog/2014/10/berlin-hackathon-results-3d-viewer-and-web-news.html#7).

#### .NET XML Configuration File Storage and Parsing

Storing user settings in a config file via the .NET ConfigurationManager and OpenMappedExeConfiguration methods:

Look at my
[DirectShape OBJ import add-in DirectObjLoader](http://thebuildingcoder.typepad.com/blog/2014/12/devdays-github-stl-and-obj-model-import.html#3),
which also defines the kernel for the
[OBJ Mesh Import to DirectShape](http://thebuildingcoder.typepad.com/blog/2015/02/from-hack-to-app-obj-mesh-import-to-directshape.html) AppStore application.

Search the two blog post discussions listed above for the word 'config', and look at the
[Config.cs configuration file](https://github.com/jeremytammik/DirectObjLoader/blob/master/DirectObjLoader/Config.cs) implementation
and usage in the
[DirectObjLoader GitHub repository](https://github.com/jeremytammik/DirectObjLoader).