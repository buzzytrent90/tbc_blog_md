---
ai_optimized: true
author: Jeremy Tammik
complexity_score: 5.4
content_type: qa
optimization_date: '2025-12-11T11:44:14.528802'
original_url: https://thebuildingcoder.typepad.com/blog/0756_xtra_adn_labs.html
post_number: '0756'
reading_time_minutes: 4
series: general
slug: xtra_adn_labs
source_file: 0756_xtra_adn_labs.htm
tags:
- csharp
- elements
- geometry
- levels
- revit-api
- selection
- walls
title: Xtra ADN Revit 2013 API Training Labs
word_count: 886
---

### Xtra ADN Revit 2013 API Training Labs

I finished the Munich Revit API training yesterday and celebrated with a picnic and a snooze in the sunshine on the on the bank of the river Isar followed by a
[wave with Maja Mühlbauer](http://tanz-als-weg.de) in the evening.

![The river Isar in Munich](http://www.best-of-munich.com/fremdenverkehr-muenchen/img/muenchen-fremdenverkehr.jpg)

Today I am heading back home again.

Before leaving, let me post the updated ADN Revit API Training material that I used.

#### Lab Projects

Most of it is a copy of the migration of the official
[ADN Revit 2012 API Labs](http://images.autodesk.com/adsk/files/revit_2012_api_training.zip)
provided on the
[Revit Developer Center](http://www.autodesk.com/developrevit).
In my trainings, however, I also include the old labs predating the new stream-lined material, packaged in the two 'Xtra' projects.

It thus contains the following eight projects, four each for C# and VB, and an RvtSamples include file AdnSamples.txt:
![ADN Revit API Training labs including Xtra](img/adn_labs_2013_project_explorer.png)

I add the ADN and Building Coder include files to the RvtSamples input file by appending two lines at the end:

```
#include C:\a\lib\revit\2013\adn\src\AdnSamples.txt
#include C:\a\lib\revit\2013\bc\BcSamples.txt
```

With those in place, I have an amply populated selection of labs, tests and sample code at my disposal:
![RvtSamples ribbon panel](img/RvtSamples_2013_ribbon_panel.png)

#### Lab Migration

The migration of the standard ADN Revit API labs from Revit 2012 to 2013 was performed by Balaji,
who joined ADN DevTech team a couple of years ago, very actively supporting the AutoCAD APIs, and recently started providing support for the Revit API as well, e.g. for a
[wall compound layer structure issue](http://thebuildingcoder.typepad.com/blog/2012/03/updating-wall-compound-layer-structure.html).
Balaji will also be the main presenter of the upcoming
[Revit API webcast](http://thebuildingcoder.typepad.com/blog/2012/04/failure-rollback.html).
Many thanks to Balaji for diving in so fast and deep and all his hard and productive Revit API related work!

Balaji thus took care of the standard ADN lab migration, and I did the old Xtra labs.

The migration was trivial, and the initial compilation only generated [six warnings](zip/adn_xtra_2013_warnings.txt) about obsolete API usage:

```
------ Rebuild All started: Project: XtraVb, Configuration: Debug Any CPU ------
XtraVb\Labs2.vb(97) : warning BC40000:
Public Function NewWall(curve As Curve, level As Level, structural As Boolean) As Wall is obsolete:
This method is obsolete in Revit 2013. Please call a static creation method of Wall class instead.

XtraVb\Labs7.vb(153) : warning BC40000:
Public ReadOnly Property Objects As GeometryObjectArray is obsolete:
This property will be obsolete from 2013; Call GetEnumerator() instead.

  XtraVb -> C:\a\lib\revit\2013\adn\src\lab\XtraVb\bin\Debug\XtraVb.dll

------ Rebuild All started: Project: XtraCs, Configuration: Debug Any CPU ------
XtraCs\Labs2.cs(198,23): warning CS0618:
Autodesk.Revit.Creation.Document.NewWall(Curve, Level, bool) is obsolete:
This method is obsolete in Revit 2013. Please call a static creation method of Wall class instead.

XtraCs\Labs2.cs(466,19): warning CS0618:
Element.PhaseCreated is obsolete:
This property is obsolete in 2013. Use CreatedPhaseId instead.

XtraCs\Labs2.cs(545,26): warning CS0618:
Element.PhaseCreated is obsolete:
This property is obsolete in 2013. Use CreatedPhaseId instead.

XtraCs\Labs7.cs(151,48): warning CS0618:
GeometryElement.Objects is obsolete:
This property will be obsolete from 2013; Call GetEnumerator() instead.

Compile complete -- 0 errors, 4 warnings
  XtraCs -> C:\a\lib\revit\2013\adn\src\lab\XtraCs\bin\Debug\XtraCs.dll
```

These warnings concern:

- The NewWall method on the creation document, which is replaced by a static Create method on the Wall class. The creation document is being gradually phased out.- The obsolete GeometryElement Objects property, which can be removed, since the class itself is now enumerable, as we
    [recently discussed](http://thebuildingcoder.typepad.com/blog/2012/04/migrate-building-coder-samples-to-revit-2013.html).- The Element.PhaseCreated property, which can be replaced by using CreatedPhaseId instead.

I fixed all these issues, which is pretty trivial, and marked the old and new lines of code with comments saying '2012' and '2013', respectively.

The entire project now compiles with zero errors and warnings.

Here is
[adn\_labs\_2013.zip](zip/adn_labs_2013.zip)
containing the complete Visual Studio solution, source code, and RvtSamples include file for displaying and launching the commands in Revit.

The standard ADN training labs also include hands-on instruction documentation and their own independent add-in manifests, in case you prefer to use those to load them.

**Addendum:** Especially for Zack, here is
[AdnSamples.txt](zip/AdnSamples_2012-06-23.txt),
which you can use as an include file in RvtSamples.txt to load all the standard and Xtra ADN Revit API training labs in one fell swoop. Enjoy :-)

#### Built-in Category OST Prefix

Here is a useless little bit of information for you people who, like me, prefer to now what every single acronym they encounter stands for:

Almost all the built-in category enum values come equipped with an "OST\_" prefix.
I asked the development team what these three letters stand for and learned that it is short for **o**bject **st**yle.
Just in case you ever wondered.