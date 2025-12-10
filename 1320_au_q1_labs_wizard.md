---
post_number: "1320"
title: "Autodesk University, Q1 Report, ADN Labs and Wizard Update"
slug: "au_q1_labs_wizard"
author: "Jeremy Tammik"
tags: ['csharp', 'elements', 'family', 'geometry', 'references', 'revit-api', 'vbnet', 'views']
source_file: "1320_au_q1_labs_wizard.htm"
original_url: "https://thebuildingcoder.typepad.com/blog/1320_au_q1_labs_wizard.html"
---

### Autodesk University, Q1 Report, ADN Labs and Wizard Update

I submitted my yearly Autodesk University proposal for the Revit API expert panel.

Time for you to submit yours as well.

The call for proposals closes in one week – the deadline is May 26th.

Besides that, here are some other things I want to point out:

- [Titbits from the Q1 report](#2)
- [Autodesk University proposals](#3)
- [ADN Revit API labs training material for Revit 2016](#4)
- [Updated Visual Studio add-in wizards for Revit 2016](#5)

#### Titbits from the Q1 Report

Autodesk [posted a solid Q1 despite the strong US Dollar](http://gfxspeak.com/2015/05/19/autodesk-despite-strong). It was another solid quarter for the AEC business. Continued adoption of BI​M in the building and infrastructure industries drove growth of the Building Design and Infrastructure Design Suites. Multiple deals across geographies were closed with important and strategic customers, including Nabholz, TBI Holdings, KHIDI, AMEC and Japan Central Rail, showing the strength of the portfolio in every segment.

While The Building Coder is normally not interested in any such nitty-gritty financial details, I thought I would still pick out a couple of AEC and Revit related highlights for you:

1. Revit 2016’s scalability and performance improvements made a difference for our customers. But don’t just take my word for it, this quote is from one very happy customer, "I have our 1.2 GB Architecture Model of a hospital opened in Revit 2016, and all I can say is 2016 is reDONKulously fast. Insanely. And our models are intensely detailed in terms of modelling geometry. 2016 refreshes views (now multi-core) in a stupid fast way." ReDONKulously, his word, not mine!
2. Solar analysis for Revit downloads more than doubled in the last month, and Lighting Analysis also continues to grow at 30% average per month. According to Elizabeth Ratner of Little (Architects), "...As I keep telling you, it’s been a game-changer for our firm."
3. BIM 360 continued to deliver solid billings growth with 19% Y/Y net new billings growth. And BIM 360 Plan (formally Project Falcon) is now being used by Suffolk Construction (pilot part of their recent EBA) who plan to use it on a mixed-use high rise in Boston. This project will construct one of the tallest buildings in Boston. DPR also began live pilots on several projects including Apple HQ and Disney World. The Apple project is a joint venture with Skanska and the Disney project is the first IPD (Integrated Project Delivery) project in the hospitality sector. BIM 360 Field added Daily Reports that gives users mobile daily project reports.

#### Autodesk University Proposals

As said, the AU 2015 call for proposals ends May 26.
Don't miss your chance to join the AU community of experts.
Contribute to industry excellence, creative design, and the future of making things around the world.
Submit your class proposal at [autode.sk/CFPAU2015](http://au.autodesk.com/speaker-resource-center/call-for-proposals).

By the way, in case you need any help with it,
[Kean Walmsley is offering free AU proposal advice](http://through-the-interface.typepad.com/through_the_interface/2015/05/autodesk-university-2015-class-proposals.html)   :-)

![Autodesk University 2015](img/au2015.png)

Today I submitted my proposal for the annual 90-minute Revit API expert panel:

> **Revit API Expert Roundtable: Open House on the Factory Floor**
>
> Interact with a panel of Revit API experts from Autodesk to answer your questions and discuss all relevant topics of your choice. For anyone writing add-ins for Revit, this is the perfect forum to get to know the people who shape the APIs you work with better and explain your views, ideas and problems directly face to face. Please note that prior .NET programming and Revit add-in development experience is required and that this class is not suitable for beginners.

Here are the notes from
[the corresponding session last year](http://thebuildingcoder.typepad.com/blog/2014/12/the-revit-api-panel-at-autodesk-university.html#2).

#### ADN Revit API Labs Training Material for Revit 2016

The
[Revit Developer Centre](http://www.autodesk.com/developrevit) provides
the ADN Revit API training material, aka Revit API labs, updated for Revit 2016.

This is the material we use for our two-day hands-on Revit API training classes.

It includes both the labs themselves, consisting of sample source code exercises in both C# and VB for you to fill in, corresponding instruction documents for both languages, and an accompanying slide deck.

The labs cover three main areas:

- Introduction for getting started with the Revit API, database, elements and properties
- User interface programming to create an external application and custom ribbon
- Family API for programmatic family generation

The entire package is hosted in the [Revit API Training GitHub repository](https://github.com/ADN-DevTech/RevitTrainingMaterial).

#### Updated Visual Studio Add-in Wizards for Revit 2016

I recently published the
[Visual Studio add-in generator wizard for Revit 2016](http://thebuildingcoder.typepad.com/blog/2015/04/add-in-migration-to-revit-2016-and-updated-wizards.html#3).

![Visual Studio Revit add-in wizard for Revit 2016](img/Revit2016AddinWizardCs0.png)

At the time, I had not yet installed the final version of Revit 2016, so some of the paths still referred to its development codename Copernicus.

As said, you should be prepared to
[adapt these wizards to your own specific needs and preferences](http://thebuildingcoder.typepad.com/blog/2015/04/add-in-migration-to-revit-2016-and-updated-wizards.html#4) anyway, and that is easy.

Here are the updated wizards, now referring to the official install location; to use, simply copy the zip file of your choice to the corresponding Visual Studio project template folder in your local file system:

- C# – copy [Revit2016AddinWizardCs1.zip](zip/Revit2016AddinWizardCs1.zip) to

  [My Documents]\Visual Studio 2012\Templates\ProjectTemplates\Visual C#- Visual Basic – copy [Revit2016AddinWizardVb1.zip](zip/Revit2016AddinWizardVb1.zip) to

    [My Documents]\Visual Studio 2012\Templates\ProjectTemplates\Visual Basic

For further details, please refer to the recent
[more complete discussion of this topic](http://thebuildingcoder.typepad.com/blog/2015/04/add-in-migration-to-revit-2016-and-updated-wizards.html#3).