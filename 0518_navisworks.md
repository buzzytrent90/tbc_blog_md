---
post_number: "0518"
title: "Create a Navisworks File from Revit"
slug: "navisworks"
author: "Jeremy Tammik"
tags: ['revit-api', 'views']
source_file: "0518_navisworks.htm"
original_url: "https://thebuildingcoder.typepad.com/blog/0518_navisworks.html"
---

### Create a Navisworks File from Revit

In case you do not know Navisworks, it is an Autodesk product which provides a flexible platform for integrating multiple CAD formats to create a compound model from multiple sources and provide specialised viewing and analysis of large projects.
It allows you to combine models from AutoCAD, Revit, Inventor, and many more.
Here is the blurb from the
[Autodesk Navisworks product page](http://www.autodesk.com/navisworks):

Autodesk Navisworks products deliver project review software for 3D coordination, 4D planning, photorealistic visualization, dynamic simulation, and accurate analysis.
Create a whole-project model by integrating design and construction information, including complex building information modelling (BIM), Digital Prototyping (DP), and process plant data.
With Autodesk Navisworks project review software, you can collaborate, coordinate, and communicate more effectively to reduce problems during design and construction.

I recently answered a question on whether you can create a native Navisworks NWD file from within Revit:

**Question:** I want to convert an RVT file into an NWD file with the help of the Revit API.
Is this possible at all?
If so, is there any sample code available?

**Answer:** You certainly can create a Navisworks file from a Revit add-in.
This can be achieved using NWcreate, which is a stand-alone Navisworks library and thus can be integrated into a Revit add-in.

The caveat is that you can only create NWC files using a standalone app, not the native NWD format.
In order to create a native NWD file, you will need Navisworks installed on the machine.
The only limitation of NWC files is that they cannot be loaded it into Navisworks Freedom.

The NWcreate samples that ship with Navisworks include a standalone program that creates an NWC file.

Note that the NWcreate samples are written in C/C++, and the project files are for VC++ 6.0 (Visual Studio 6).
They should not be too hard to port to Visual Studio 2008 or 2010, if you need to do so.

More information, samples, documentation and the Navisworks development tools are available from the Autodesk
[Navisworks Developer Centre](http://www.autodesk.com/developnavisworks).

Thanks to Michael Priestman, Sr Navisworks SW Engineer, for providing most of this information!

#### Programming Quotes

Before closing for today, I will mention that I found some of the entries in and comments on this list of
[top 50 programming quotes of all time](http://www.junauza.com/2010/12/top-50-programming-quotes-of-all-time.html) quite amusing...