---
ai_optimized: true
author: Jeremy Tammik
complexity_score: 3.4
content_type: qa
optimization_date: '2025-12-11T11:44:13.383499'
original_url: https://thebuildingcoder.typepad.com/blog/0112_back_again.html
post_number: '0112'
reading_time_minutes: 5
series: general
slug: back_again
source_file: 0112_back_again.htm
tags:
- csharp
- parameters
- references
- revit-api
- rooms
title: Back Again
word_count: 1008
---

### Back Again

I'm back from a wonderful vacation in southern Italy.
I tried to plan something beforehand and inform myself, but found no time to do so.
I ended up just booking the flights to and from Napoli, with no idea what would happen in between.
My main goal was to practice Italian.
And the result was great!

The main components were:

- I spoke to lots and lots of people, it was really wonderful, they are so nice.
- I saw wonderful places, I never expected it to be so beautiful and fascinating.
- I read a book in Italian, actively using the dictionary from Italian to English.
- I wrote a dairy in Italian, a daily journal, actively using the dictionary from English to Italian.

I completely forgot about work and computers.

I'm really happy to be back and excited to see how I can carry my current enthusiasm with me into everyday working life again.

I have just about gotten through the first look at emails.
Here are some of the questions that were raised.
There are lots more waiting, though:

- Is there any documentation for the Revit 2010 (Beta) API?
- We are integrating our structural analysis application with Revit Structure. Can you please point me towards resources for calling the Revit API from unmanaged C++?
- Is there any way of tagging objects from a linked file into a host file?
- RstLink: I am working in C# and have eliminated the VB projects from the RstLink solution. I assume their functionality is identical? The solution also includes a C++ project which I cannot compile, because it is lacking the file 'arxHeaders.h'. Where can I obtain this file? What is the use of this C++ project?
- MidasLink: I am compiling the 2008 version of the MidasLink source code with the 2009 API. This is causing errors because some parameters are passed by reference. Do you have the 2009 version of the code?

#### Revit 2010 API documentation

**Question:** Is there any documentation for the Revit 2010 (Beta) API?

**Answer:** The Revit API documentation is included in the SDK, as always, and that is included in the Revit beta installation.
When you install Revit 2010 beta, a temporary folder is created.
Search that folder globally and recursively for a file named \*sdk\*.
That is an executable file, named something like RevitSDK.exe.
It is a self extracting archive file containing the SDK documentation and samples.
Obviously, this assumes that you have access to the beta software, for example through an ADN membership.

For example, in the case of Revit MEP, once you installed the Revit 2010 beta to the default location, you will find RevitSDK.exe in the directory 'C:\Program Files\Autodesk\Revit MEP 2010 Beta\support\SDK'.
Executing this and installing to the default path will place the documentation in 'C:\Program Files\Autodesk\Revit MEP 2010 Beta\support\SDK\Software Development Kit'.

#### Calling the Revit API from unmanaged C++

**Question:** We are integrating our structural analysis application with Revit Structure. Can you please point me towards resources for calling the Revit API from unmanaged C++?

**Answer:** If you wish to work with the Revit API in unmanaged C++, you will have to use mixed mode to access it.
The Revit API is managed.
There is no unmanaged access.
Using mixed mode, you can combine your unmanaged code and the managed code to access the Revit API in the same application.
An example is provided in the Revit API
[tips and tricks sample code](zip/rac_tips_20090303.zip).

#### Tagging objects from a linked file

**Question:** Is there any way of tagging objects from a linked file into a host file?

**Answer:** When you say 'tagging objects from a linked file into a host file', I am not completely sure exactly what you mean.
If your intention is to create the tags in the host file for objects in the linked in one, I am afraid that is not possible.
In general, the API will almost never enable you to do something that cannot be achieved through the user interface.
I would expect that the tag and the object it is tagging must reside in the same document.
On the other hand, if your intention is to create tags in the linked in file for objects in the linked in file, then this is probably possible.
The Revit SDK samples AutoTagRooms and TagBeam demonstrate how to create new tags for objects.
This operation can probably be performed through the API in a background document such as a linked in project.

#### RstLink and ObjectARX

**Question:** RstLink: I am working in C# and have eliminated the VB projects from the RstLink solution. I assume their functionality is identical? The solution also includes a C++ project which I cannot compile, because it is lacking the file 'arxHeaders.h'. Where can I obtain this file? What is the use of this C++ project?

**Answer:** The C++ header file arxHeaders.h is part of the AutoCAD ObjectARX API.
This C++ application is part of the Revit analysis link sample application.
It is used to define properties displayed in the AutoCAD Object Property Manager or OPM, and is part of the simulation of an external analysis package using AutoCAD.
You do not need to compile this part of the application to analyse the Revit part of the link application.
If you wish to make use of the simulation of an external analysis package using AutoCAD, you need to have AutoCAD installed, and install and set up Visual Studio to compile an ARX application.

#### MidasLink for Revit 2009

**Question:** MidasLink: I am compiling the 2008 version of the MidasLink source code with the 2009 API. This is causing errors because some parameters are passed by reference. Do you have the 2009 version of the code?

**Answer:** A
[2009 version of MidasLink](http://adn.autodesk.com/adn/servlet/item?siteID=4814862&id=11816840&linkID=4901650)
is available on the ADN web site.

Well, that was it, for a start.
Back to answering emails and further questions now.