---
ai_optimized: true
author: Jeremy Tammik
complexity_score: 5.0
content_type: tutorial
optimization_date: '2025-12-11T11:44:14.758726'
original_url: https://thebuildingcoder.typepad.com/blog/0865_python_ui_server_api.html
post_number: 0865
reading_time_minutes: 5
series: general
slug: python_ui_server_api
source_file: 0865_python_ui_server_api.htm
tags:
- elements
- python
- references
- revit-api
- transactions
- views
- windows
title: AU Classes on Python, UI, Server and Framework APIs
word_count: 1049
---

### AU Classes on Python, UI, Server and Framework APIs

Thursday was another very full and fruitful day at AU.
I gave my third and last class, the most exciting one in my eyes, on some UI and integration aspects.
I was unable to attend Iffat May's class introducing Python and using it in Revit and Vasari via RevitPythonShell, but I love her material so much I am including it anyway.
Adam Nagy presented a class addressing cloud and mobile development and the Revit Server API.
Finally, Arnošt Löbel provided a peek behind the scenes and into the depths of the fundamental frameworks underlying the Revit API.

Here they are in chronological order:

1. [CP3837-L](https://www.autodeskuniversity2012.com/connect/sessionDetail.ww?SESSION_ID=3837)
   Scripting with [RevitPythonShell in Vasari](#1) by Iffat May.- [CP4107](https://www.autodeskuniversity2012.com/connect/sessionDetail.ww?SESSION_ID=4107)
     Let's Face It: New [Revit 2013 User Interface API](#2) Functionality by Jeremy Tammik.- [CP3093](https://www.autodeskuniversity2012.com/connect/sessionDetail.ww?SESSION_ID=3093)
       My First Cloud/Mobile App with [Revit Server](#3) by Adam Nagy.- [CP3426](https://www.autodeskuniversity2012.com/connect/sessionDetail.ww?SESSION_ID=3426)
         [Core Frameworks](#4) in the Revit API by Arnošt Löbel.

I am including the class materials below, for your and my own convenience, and also to ensure that all this juicy stuff really is picked up and returned by Internet search engines.

#### 1. [CP3837-L](https://www.autodeskuniversity2012.com/connect/sessionDetail.ww?SESSION_ID=3837) RevitPythonShell in Vasari

Iffat provides an entire introductory tutorial to the Python programming language, and then shows how to apply it to solve a number of useful tasks in Revit and Vasari.
She makes use of Daren Thomas'
[Revit Python shell](http://code.google.com/p/revitpythonshell),
originally implemented for Revit 2010, then for
[Vasari](http://thebuildingcoder.typepad.com/blog/2011/07/python-shell-in-revit-and-vasari.html),
[Revit 2012 and Vasari 2.1](http://thebuildingcoder.typepad.com/blog/2011/09/python-shell-for-revit-2012-and-vasari-21.html),
and now available for Revit 2013.

RevitPythonShell introduces interactive scripting ability to Revit and Project Vasari.
Designers now have the ability to interactively design and manipulate Revit elements using algorithms and computational logic.
The class explores the Python structures, variables, data types, and flow control, and shows how to use them to create scripts to control Revit elements dynamically.

- [Handout](file:///a/doc/revit/au/2012/doc2/CP3837/CP3837-L_Scripting_RevitPythonShell_handout.pdf)
- Presentation
- Sample code and models

#### 2. [CP4107](https://www.autodeskuniversity2012.com/connect/sessionDetail.ww?SESSION_ID=4107) Revit 2013 User Interface API

This class takes a deeper look at the new user interface and add-in integration functionality provided by the Autodesk Revit 2013 API.
It covers 2013 features including Options dialogue custom extensions using WPF components, subscription to Revit progress bar notifications, embedding and controlling a Revit view inside an add-in dialogue for preview purposes, the new drag and drop API, and the UIView.
This class is complementary to and expands on CP3272, a Snapshot of the Revit UI API:

- Document management and View API- Revit progress bar notifications- Options dialogue WPF custom extensions- Embedding and controlling a Revit view- UIView and Windows coordinates- Drag and drop

From my point of view, one of the most exciting new features provided by the Revit 2013 UI API is the possibility to correlate between Windows screen device coordinates and Revit model coordinates provided by the UIView class, which I used to examine and implement my
[own Revit tooltip](http://thebuildingcoder.typepad.com/blog/2012/10/uiview-windows-coordinates-referenceintersector-and-my-own-tooltip.html),
which also makes use of the new ReferenceIntersector class and Idling event.
This has never previously been possible.
Now you can do anything you like using the Windows API and connect that intelligently with the underlying Revit model.

- [Handout](file:///a/doc/revit/au/2012/doc/cp4107_revit_2013_ui_api.pdf)
- Presentation
- Sample code

#### 3. [CP3093](https://www.autodeskuniversity2012.com/connect/sessionDetail.ww?SESSION_ID=3093) Cloud/Mobile App with Revit Server

This class has two objectives: to introduce you to the Revit Server REST API and to introduce you to cloud and mobile programming.

Autodesk Revit Server is a server-based file storage system that we can use to store Revit files.
It has a public API that uses simple HTTP known as REST (or Representational State Transfer).
REST is a "style" for designing network applications, and it is used to communicate between the server and a client machine.
REST is simple, yet powerful enough. You can use it from anywhere where HTTP programming is available: from a desktop, a mobile device or a server-side application.
In this class, we discuss what REST is, using the context of Revit Server.

We then go one step further and show you how to create your own REST service in .NET.
We implement it as a notification system in the cloud and demonstrate with an Apple iPad/iPhone application in Apple iOS.
At the end of this class, you will be able to:

- Use HTTP requests to interact with a REST API- Programmatically access and modify data available on Revit Server- Create basic iOS applications in Xcode- Create a basic REST service in .NET- Use Apple Push Notification

Here is the detailed handout document:

- [Handout](file:///a/doc/revit/au/2012/doc2/CP3093/CP3093_My_First_Cloud-Mobile_App_with_Autodesk_Revit_Server.pdf)

#### 4. [CP3426](https://www.autodeskuniversity2012.com/connect/sessionDetail.ww?SESSION_ID=3426) Core Frameworks in the Revit API

A good understanding of core frameworks in the Revit API is a prerequisite for developing well behaved Revit add-ins.
Among the most important ones, the following frameworks play key roles in most applications:

- External applications, commands, and events- Transactions phases- Regeneration and transaction modes- Updaters and other call-backs- Scoped objects and element validity

These concepts have been around for many releases, yet there are still facts about them that may not be completely understood.
This class summarizes the necessary basics, and also sheds some light on the behaviour that is normally hidden 'under the hood'.
Knowledge acquired during this class will help Revit developers to build more efficient, safer, and robust applications.

- Handout
- Presentation
- Sample code

By the time you read this, I will be sitting in the plane heading back to Europe.

Take care, and walk in beauty.