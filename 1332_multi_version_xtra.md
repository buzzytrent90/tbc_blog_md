---
ai_optimized: true
author: Jeremy Tammik
complexity_score: 3.0
content_type: qa
optimization_date: '2025-12-11T11:44:15.789681'
original_url: https://thebuildingcoder.typepad.com/blog/1332_multi_version_xtra.html
post_number: '1332'
reading_time_minutes: 4
series: general
slug: multi_version_xtra
source_file: 1332_multi_version_xtra.htm
tags:
- csharp
- elements
- family
- geometry
- revit-api
- walls
title: ADN Labs Xtra, Multi-Version Add-Ins and CNC Direct
word_count: 711
---

### ADN Labs Xtra, Multi-Version Add-Ins and CNC Direct

Let's discuss some more Revit add-in migration aspects and yet another completed migration task:

- [ADN Revit API training labs Xtra migration](#2)
- [Multi-version add-ins and CNC Direct](#3)

#### ADN Revit API Training Labs Xtra Migration

I recently mentioned that the official
[ADN Revit API Labs Training Material for Revit 2016](http://thebuildingcoder.typepad.com/blog/2015/05/autodesk-university-q1-adn-labs-and-wizard-update.html#4) is
available from the
[Revit Developer Centre](http://www.autodesk.com/developrevit) and the
[Revit API Training GitHub repository](https://github.com/ADN-DevTech/RevitTrainingMaterial).

They are used for our standard two- or three-day hands-on Revit API introduction training courses.
They are also suitable for self-learning and include full step-by-step instructions, separate for both C# and VB add-ins.

As you maybe know, I also maintain an extended version of these including the precursor versions, repackaged in the ADN Revit API Training Labs Xtra modules.

The Xtra version Visual Studio solution implements the following projects:

![ADN Revit API Training Labs Xtra Visual Studio solution](img/AdnXtra_solution.png)

The first six are the official, standard, ADN training labs to introduce the Revit API basics, UI programming and Family API.

Obviously, the Xtra labs have advantages and disadvantages over the standard labs, so you should check out both and decide which you prefer for yourself.

They also include a couple of additional utilities that I frequently find useful, especially a simplified version of the
[BipChecker](http://thebuildingcoder.typepad.com/blog/2015/05/geometry-creation-and-line-intersection-exceptions.html#5)
([2015](http://thebuildingcoder.typepad.com/blog/2014/05/bipchecker-for-revit-2015-on-github.html#3)) and the
[element lister](http://thebuildingcoder.typepad.com/blog/2014/09/debugging-and-maintaining-the-image-relationship.html#2).

I now migrated the Xtra labs to Revit 2016 as well and resynchronised them with the official ADN Revit API Training Labs.

The migration was pretty straightforward.

Here is my external application RvtSamples listing entry points to launch all the Revit SDK, ADN Xtra lab and The Building Coder samples:

![RvtSamples listing SDK, Adn Xtra and The Building Coder samples](img/AdnXtra_2016.png)

As always, the most up-to-date version of the ADN Revit API Training Labs Xtra is provided in the
[AdnRevitApiLabsXtra GitHub repository](https://github.com/jeremytammik/AdnRevitApiLabsXtra),
and the current version right now is
[release 2016.0.0.6](https://github.com/jeremytammik/AdnRevitApiLabsXtra/releases/tag/2016.0.0.6).

#### Multi-Version Add-Ins and CNC Direct

Talking about migrating between major releases of the Revit API, how about avoiding that issue altogether?

Well, it is definitely achievable, and can even be very easy or come for free in certain simple cases.

For instance, I just discussed the detailed steps required to
[migrate the CNC Direct add-in from Revit 2014 to 2015 and 2016](http://thebuildingcoder.typepad.com/blog/2015/06/cnc-direct-export-wall-parts-to-dxf-and-sat.html).

I was surprised that William never asked me for an add-in update and asked him whether he had done anything himself to maintain or improve it in any way in the meantime, to which he replies, "Glad you liked the YouTube video. I didn't change or compile anything. All I did was add the add-in to the appropriate folder for Revit 2015 and 2016."

In other words, this particular add-in just happens to be upwards compatible across three major releases of Revit.

Of course, this only works if you make no use of any Revit API functionality that was modified between releases.

I recently discussed a
[compatibility helper](http://thebuildingcoder.typepad.com/blog/2015/05/compatibilizar-entre-vers%C3%B5es-api-compatibility-helper.html) that
is useful in case do you want implement an single multi-version add-in that does access Revit API functionality that changed across versions.

Here are two other approaches that we discussed to support multiple API versions:

- [Multi-Version Add-in](http://thebuildingcoder.typepad.com/blog/2012/07/multi-version-add-in.html)
- [Multi-Version Visual Studio Revit Add-In Wizard](http://thebuildingcoder.typepad.com/blog/2013/11/multi-version-visual-studio-revit-add-in-wizard.html)