---
post_number: "1797"
title: "Indesign Rvtclipper"
slug: "indesign_rvtclipper"
author: "Jeremy Tammik"
tags: ['csharp', 'elements', 'geometry', 'levels', 'references', 'revit-api', 'sheets', 'views']
source_file: "1797_indesign_rvtclipper.md"
original_url: "https://thebuildingcoder.typepad.com/blog/1797_indesign_rvtclipper.html"
---

### Automated PDF Report and 2D Sketch Booleans
Today, we highlight two contributions by Håvard Leding and Oliver Green:
- [2D Boolean interactive real-time sketch viewer](#2)
- [Single-click automated PDF report via InDesign](#3)
- [Abbreviated table of technical contents](#3.1)
- [Addendum – CodeProject on InDesign](#3.2)
#### 2D Boolean Interactive Real-Time Sketch Viewer
Håvard Leding of [Symetri](https://www.symetri.com) shares a new cool 2D Boolean geometry analysis tool, RvtClipper, saying:
This might be of interest to The Building Coder blog.
I worked up your Revit implementation of the Clipper library presented in the article
on [Boolean operations for 2D polygons](https://thebuildingcoder.typepad.com/blog/2013/09/boolean-operations-for-2d-polygons.html).
I added a UI where you can see the result of the different options in real-time.
For simplicity, it takes sketch-based objects as input rather than just lines.
So, Filled Regions, Floors, Railings, Roofs and what not.
Anything that has a `Sketch` element should work.
I just took what you had there and added a few things.
Source code and short video attached:
- [RvtClipper.zip](zip/hl_RvtClipper.zip)
- [RvtClipper_UI.mp4](zip/hl_RvtClipper_UI.mp4)
I see in the video that RvtClipper can even generate new Revit geometry and database elements based on the 2D Boolean results!
Many thanks to Håvard for this impressive utility, showing how easily you can grab an external library to add really powerful functionality to your Revit add-in!
![RvtClipper](img/RvtClipper.png)
#### Single-Click Automated PDF Report via InDesign
Oliver Green shares the presentation and some utility code from his industry talk \*AULON484 – One-Click PDF Model Reports: Connect Revit to the InDesign API\* at Autodesk University 2019 in London, saying:
I develop for the Revit API, love The Building Coder blog, and wondered if you might find the following talk we gave at AU of interest?
It's about automated publishing from Revit using a bridge we built to [Adobe InDesign](https://www.adobe.com/products/indesign.html).
The talk is aimed at beginners and pitched at the level of tech understanding that your blog covers (C#, OOP and some .NET), so I think it might be a good match for your readers:
- [AULON484 – One-Click PDF Model Reports: Connect Revit to the InDesign API](https://www.autodesk.com/autodesk-university/class/One-Click-PDF-Model-Reports-Connect-Revit-InDesign-API-2019)
We also shared a [AhmmTools/AU2019 GitHub repository](https://github.com/AhmmTools/AU2019) with some useful starter code on it, e.g.:
- [InDesignSampleAU2019.cs](https://github.com/AhmmTools/AU2019/blob/master/InDesignSampleAU2019.cs)
Since model reviews tend to be quite company-specific, we didn't want to dictate how to do it so much as just give useful code examples and principles for others to adapt as suits them.
I'm happy to bring across the several other code examples we showed in our presentation if that would be helpful!
The main challenges weren't really Revit API-specific.
They were more to do with working around the InDesign API.
However, since the talk is aimed at beginners, I've gone into as much detail as possible so others could attempt to recreate what we demoed.
It might make for a nice case study in learning how to connect Revit to other software (in this case using COM).
Many thanks to Ollie for documenting and sharing this!
![InDesign COM API connection](img/og_indesign.png)
I copied his handout [AU2019OneClickModelReports.pdf](zip/AU2019OneClickModelReports.pdf) and extracted the following abbreviated table of technical contents from it, in case you would like to quickly get an idea before diving into the real thing:
#### Abbreviated Table of Technical Contents
\*One-Click Model Reports: Connect Revit to the InDesign API\* by Oliver Green and Aaron Perry, Allford Hall Monaghan Morris
- InDesign SDK
- InDesign API Documentation
- C# Examples
- Simple Proof of Concept Workflow
- Ribbon Buttons in Revit
- Intro to External Commands
- ExternalCommands and ExternalApplications
- Visual Studio Overview
- ExternalCommands
- ExternalApplications
- ExternalCommand or ExternalApplication
- Back to Proof of Concept
- Interprocess Communications
- Revit and the .NET Framework
- COM (Component Object Model)
- Visual Studio Setup
- Back to Visual Studio
- Add Reference to InDesign API
- Fixing COM Error
- Use InDesign's API
- Back to Proof of Concept
- Preparing the Model Report Process
- What Can We Automate?
- Full Proposed Process Diagram
- Pop-Up Dialog for Input Info
- Create New File from Template
- Find and Replace Text
- Update Text Variables
- Target Tables and Input Data
- Normal .NET Operations
- Launch UI Menu Commands
- Live Demonstration
- Conclusion
- Further Possibilities
- Now It's Your Turn
- Resources
#### Addendum – CodeProject on InDesign
Mark Ackerley added a [comment on this](https://thebuildingcoder.typepad.com/blog/2019/11/automated-pdf-report-and-2d-sketch-booleans.html#comment-4682012920):
Re. the Indesign report, here's another link that is mentioned in the presentation and I found very useful:
- [Create an Adobe InDesign Document with C#](https://www.codeproject.com/Tips/124998/Create-an-Adobe-InDesign-Document-with-c)
It demonstrates some functionality which isn't included in the presented project.
Thank you Mark, for pointing that out.