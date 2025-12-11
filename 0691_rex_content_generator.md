---
ai_optimized: true
author: Jeremy Tammik
complexity_score: 2.7
content_type: tutorial
optimization_date: '2025-12-11T11:44:14.404636'
original_url: https://thebuildingcoder.typepad.com/blog/0691_rex_content_generator.html
post_number: 0691
reading_time_minutes: 6
series: general
slug: rex_content_generator
source_file: 0691_rex_content_generator.htm
tags:
- csharp
- elements
- family
- revit-api
- views
title: REX Content Generator
word_count: 1202
---

### REX Content Generator

I had an absolutely magical experience in Israel before leaving Tel Aviv Thursday afternoon.
Among many others at the developer conference on Wednesday, we met with
[Dima Brickmann](http://www.dimabrickman.com) of
[Omnitech](http://www.omnitech.co.il),
and he suggested going for a short visit to Jerusalem before returning to the airport, which is located between Tel Aviv and Jerusalem.
Partha and I decided to join him.
Dima has a very special relationship with the city and a deep philosophical and photographical view of it, so he was a unique and wonderful guide.
His tour focused more on feeling and experiencing the magic of this deeply religious place, which brings out so many extremes in all the different religions, and yet simultaneously enables them to coexist in an extremely confined area.

By the way, Dima has more recent photos with some short philosophical texts on the Russian photoreporting news web site
[newsru.co.il](http://www.newsru.co.il),
e.g. this beautiful photo series on
[Jerusalem between black and white](http://www.newsru.co.il/photo/21nov2011/jerusalem2.html):

![Jerusalem between black and white](http://newsru.co.il/pict/id/480783_20111121113039.jpg)

I went and installed a translation add-on for my browser to be able to read the Russian text.
It works very well, and the photos are really and truly deep and beautiful.

Click on the link at the bottom saying 'ПРОДОЛЖЕНИЕ ФОТОРЕПОРТАЖА' to proceed to the rest of the photos in the series.

Thank you, Dima!

I would never have believed you could experience so much in such a short time, both in the three hours that we spent with Dima in Jerusalem and in the two days we spent in Israel.

On Friday we held another developer conference in Paris, and today we are already en route to Gothenburg in Sweden.

And my backlog of interesting blog posts is growing far faster than I can publish them.

#### REX Content Generator

One really astounding revelation at the DevLab at AU was the REX content generator sample.

It is provided with the REX framework and its other samples as part of the standard Revit SDK.
I had not looked at it previously in any depth, since I have not used REX myself yet.
I tend to avoid any additional frills, and the plain vanilla Revit API keeps me fully occupied all on its own.
But this sample really bowled me over!

A DevLab participant showed me some of its features, and it looks like an astonishingly powerful tool.
Here are a few notes I made from his demo:

The content generator provides the following functionality:

- Read existing families into a data structure.- Create families from the data structure.- Determine the section of a beam or column and display a shaded 3D preview of its solid extrusion.- Interface to external structural databases to generate sections and families with these objects.- Recognition which tries to match the current element to an existing database entry. If the user accepts the match, the content generator can switch the types by creating a new family based on the database record and replace all of the old instances by the new family.

Impressed?

I was!

The family API enables us to create new families step by step.
In contrast, the content generator allows you to load an existing family, tweak and change it, and immediately generate a new family.

There is no need to install the Revit extensions to ensure that the basic REX framework is in place and the REX API is available.
The content generator can be used even without Revit Extensions installed, since the framework is part of the regular Revit installation and is installed in a way transparent for the user.

To run the content generator sample, simply select a structural element, e.g. a column, and launch the command.
It takes a little while to start up, because it reads the three databases: standard, steel and structural.

I asked Emmanuel Weyermann for a short description, and he provided the following brief yet full picture of Content Generator and this sample application:

#### What is Content Generator?

The Content Generator is a component which is part of the Revit Extensions Framework installed with Revit 2012.

It provides tools for content management. This component is strongly connected with standard databases shipped with Revit (over 130 databases).

Advantages of the Content Generator:

- Generation of Revit families: database sections, parametric sections, materials, user generic types.- Interpretation of Revit families: database sections, parametric sections, materials.- Getting data from standard databases.- Searching elements in standard databases.- Standard database browser dialogue.- A mapping system.

More information may be found in the "User Manual for Extensions SDK.pdf".

#### Content Generator basic types

The idea of Content Generator is very simple.
It provides its own representations of specific types (e.g. sections) and two converters which communicate with each other via these types:

- RVT Converter – responsible for communication with Revit- REX Converter – responsible for communication with standard databases

![REX Converter Pipeline](img/rex_converter_pipeline.png)

Content Generator provides 4 types:

- Database section- Parametric section- Material- Generic – defined by the user.

#### Code sample

The code below presents how to generate a particular section taken from the Rcatpro database:
```csharp
  // Initializing the rex converter.

  REXFamilyConverter rex = new REXFamilyConverter();

  // Database identification data.

  string alias = "HEA 100";

  string directory = rex.GetDatabaseDirectory(
    ECategoryType.SECTION\_DB );

  string dbName = "Rcatpro";

  // Getting the REXFamilyType from the database.

  REXFamilyType\_DBSection famDBSection
    = rex.GetFamilyDBSection( EElementType.BEAM,
      EMaterial.STEEL, alias, dbName, directory );

  // Initializing the RVT converter.

  RVTFamilyConverter rvt = new RVTFamilyConverter(
    m\_CommandData, true );

  // Getting Revit element.

  Autodesk.Revit.Element el = rvt.GetElement(
    famDBSection );

  FamilySymbol famSymbol = el as FamilySymbol;
```

Over twenty further samples are provided in the "User Manual for Extensions SDK.pdf".

#### ContentGeneratorWPF SDK sample

In Extensions SDK may be found the ContentGeneratorWPF sample.
It allows examining a selected beam or column and changing its type to a new one (generated by Content Generator).

The sample demonstrates some of Content Generator key features:

- Revit FamilySymbol interpretation as:
  - Parametric section – shape is recognized and dimensions are calculated.- Database section – the element is recognized and match with appropriate section from database.- Revit FamilySymbol generation:
    - Parametric section – there are three types available (in CG there are over 22): rectangle, angle and I section.- Database section – the user is able to point section from available databases.- Database browsing.

The sample runs database browser which allows exploring available databases:

![REX Converter Sample Form](img/rex_converter_sample.png)

#### Where is Content Generator used?

- As a standalone application which generates different types of Revit content (part of Revit Extension for RAC and RST - Extensions ribbon->Tools).- In different integration tools:
    Revit – Cloud computation ([Storm](http://labs.autodesk.com/utilities/revit_storm)),
    links Revit – RSA, Revit – ASD, and for some files import/export extensions Revit – CIS, Revit – SDNF.- In different Revit Extensions: steel connections, reinforcements, timber framing,
      [structure generator](http://labs.autodesk.com/utilities/revit_structure_generator)
      ([recently mentioned](http://thebuildingcoder.typepad.com/blog/2011/11/project-structure-generator-on-autodesk-labs.html)) etc.

Many thanks to Emmanuel for this overview!

If you are interested in Revit Structure and structural databases, I strongly encourage you to take a deep look at this if you have not already done so.