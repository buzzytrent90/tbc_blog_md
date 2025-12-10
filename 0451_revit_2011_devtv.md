---
post_number: "0451"
title: "Revit 2011 DevTV"
slug: "revit_2011_devtv"
author: "Jeremy Tammik"
tags: ['csharp', 'elements', 'filtering', 'parameters', 'revit-api', 'rooms', 'selection', 'views']
source_file: "0451_revit_2011_devtv.htm"
original_url: "https://thebuildingcoder.typepad.com/blog/0451_revit_2011_devtv.html"
---

### Revit 2011 DevTV

The Revit DevTV recordings are probably the absolutely most efficient learning resource for quickly getting a hands-on experience of programming Revit. They include

- Overview of the SDK- Installation of samples- Creating your first add-in- Iterating over database elements- Selecting an element- Extracting element parameter data- ... And much more!

The first version was created for
[Revit 2008](http://thebuildingcoder.typepad.com/blog/2008/08/getting-started.html),
and that was reused for the
[Revit 2009 training material](http://thebuildingcoder.typepad.com/blog/2008/08/getting-started.html) as well.
Augusto Gonçalves of Autodesk Brazil created an updated version for
[Revit 2010](http://thebuildingcoder.typepad.com/blog/2009/12/updated-devtv-introduction-to-revit-programming.html) in
December 2009, and he also created the wonderful
[DevTV add-in templates for Revit 2011](http://thebuildingcoder.typepad.com/blog/2010/07/devtv-addin-templates.html).

Now we have a vastly improved version for the Revit 2011 API.
It is split into two parts to make it more manageable.
Part 1 was again created by Augusto, in English and Portuguese, and Part 2 by Adam Nagy.
They are hosted on the
[Revit Developer Center](http://www.autodesk.com/developrevit) site
and also accessible through the following links which I copied from there:

- DevTV: Introduction to Revit 2011 Programming - Part 1:

  A short video tutorial demonstrating the basic steps to start developing with Revit .NET API.

  [View online](http://download.autodesk.com/media/adn/DevTV_Revit_API_2011_English_Part1/Revit_2011_DevTV_EN.html) –
  [Download](http://download.autodesk.com/media/adn/DevTV_Revit_API_2011_English_Part1.zip)

- DevTV: Introduction to Revit 2011 Programming (Portuguese):

  Introdução à Programação Revit

  Uma videoaula de curta duração que mostra as etapas básicas para começar a programar com a API Revit .NET.

  [View online](http://download.autodesk.com/media/adn/DevTV_Revit_API_2011_Portugues_Parte1/Revit_2011_Portugues.html) –
  [Download](http://download.autodesk.com/media/adn/DevTV_Revit_API_2011_Portugues_Parte1.zip)

- DevTV: Introduction to Revit 2011 Programming Part 2:

  A short video tutorial demonstrating selection and filtering API through a room renumbering application.

  [View online](http://download.autodesk.com/media/adn/DevTv_Revit_NET_English-Part2/Revit%20DevTV%20-%20part%202.html) –
  [Download](http://download.autodesk.com/media/adn/DevTv_Revit_NET_English-Part2.zip)

Here are the detailed tables of contents:

##### Part 1

1. Introduction- Hello World- Add-In Manager- External Command- Selection- Revit Lookup- Database- Filters- Parameters- Learning More

##### Part 2

1. Introduction- Add-In in Action- Add-In Framework- Picking Objects- RevitLookup Tool- Element Filtering- Testing the Add-In- Add-In Manager- Learning More

##### Part 2 Expands on Part 1

There is a certain apparent overlap between the two tables of content, because the two presentations cover related materials using different examples and highlighting different important aspects.
Part 2 is a significant expansion over Part 1, however. As Adam puts it, the difference is that Part 2 discusses how to use what you learned in Part 1:

1. Introduction – Part of the DevTV framework.- Add-In in Action – explains what the add-in will do.- Add-In Framework – refers back to Part 1 without going into details.- Picking Objects – Part 1 talks about iterating the currently selected objects, Part 2 talks about picking.- RevitLookup Tool – Part 1 has a quick look at it but does not 'use' it, whereas Part 2 does.- Element Filtering – Part 1 presented the OfClass and OfCategory shortcuts; Part 2 goes further by showing WherePasses and ParameterFilter.- Testing the Add-In – just a very quick test this time.- Add-In Manager – Part 2 actually shows how to use it, while Part 1 just mentioned it.- Learning More – Part of the DevTV framework.

##### DevTV Template Update

By the way, it is extremely easy to edit the
[DevTV Visual Studio Wizard templates](http://thebuildingcoder.typepad.com/blog/2010/07/devtv-addin-templates.html) included with Part 1.
Simply unzip the files to the hard disk, edit them just as you see fit, and zip them back up again.
For example, here is
[my personalised C# DevTV template](zip/TemplateRevitArchAddinCsJt.zip) which
generates an absolutely minimal C# Revit add-in skeleton with all comments removed and some additional copyright information added to the assembly properties.