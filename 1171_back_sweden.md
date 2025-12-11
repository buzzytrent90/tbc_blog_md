---
ai_optimized: true
author: Jeremy Tammik
complexity_score: 3.6
content_type: qa
optimization_date: '2025-12-11T11:44:15.448672'
original_url: https://thebuildingcoder.typepad.com/blog/1171_back_sweden.html
post_number: '1171'
reading_time_minutes: 4
series: general
slug: back_sweden
source_file: 1171_back_sweden.htm
tags:
- doors
- elements
- family
- parameters
- python
- revit-api
- sheets
- views
- walls
title: Back from Sweden
word_count: 726
---

### Back from Sweden

I returned from my wonderful relaxing outdoor vacation in the vast nature of Sweden and had numerous simple Revit API questions waiting for me on my return, e.g.

- [Annotation Location property](#2)
- [Placing ElementType instances](#3)
- [Commercial use of the Revit API](#4)

Before looking at those, here is an impression of the huge open space offered by heaven and earth in the islands in the Stockholm
[Skärgården](http://en.wikipedia.org/wiki/Stockholm_archipelago) archipelago,
e.g.
[Utö](http://en.wikipedia.org/wiki/Ut%C3%B6,_Sweden) and
Ålö:

![Ålö strand](file:////j/photo/jeremy/2014/2014-06-21_sverige/20140618_140745_jeremy_ålö_beach.jpg)

For something more tranquil, here is a sunset meditation on some reeds in the inland lake
Kvarnsjön:
![Reeds in Kvarnsjön](file:////j/photo/jeremy/2014/2014-06-21_sverige/20140619_214401_kvarnsjön_reeds.jpg)

Here they are in context:

![Reeds in Kvarnsjön](file:////j/photo/jeremy/2014/2014-06-21_sverige/20140619_214429_kvarnsjön_reeds.jpg)

Here is an
[album](https://www.facebook.com/media/set/?set=a.10203275797548613.1073741831.1019863650) with more pictures.

Back to the Revit API...

#### Annotation Location Property

**Question:** I’ve been following your website for a few weeks now, Great Stuff!

It’s been really helpful in figuring out Dynamo and how to fill in the gaps with Python components.

I’ve run into an issue that I haven’t seen documented anywhere online and wondered if you have any thoughts (maybe your next blog topic).

I’m trying to set up a dynamo script to replace updated images.

I can’t find anything related to the location of an annotation element in a view.

Obviously it doesn’t have a world XYZ, but somehow Revit is tracking where it is located within the view/sheet because it does have a location parameter.

I just can’t find a way to extract it.

**Answer:** Thank you for the appreciation!
I am glad you find it useful.

Every Revit element has a Location property.

You retrieve it and cast it to either a LocationPoint or a LocationCurve instance.

If the cast fails, i.e. returns null, or if the Location property is null to start with, there is no API access to this information.

You can explore the contents of the Location property interactively using RevitLookup or other
[more powerful scripting](http://thebuildingcoder.typepad.com/blog/2013/11/intimate-revit-database-exploration-with-the-python-shell.html) approaches.

#### Placing ElementType Instances

**Question:** I am trying to place new instances of an ElementType via the API.

I can right-click on the type in the Project Browser and select ‘Create Instance’, but need to launch this from the API.

I know I’ve seen in your blog the idea of using shortcut keys like ‘WA’ to begin placing a wall, but was wondering if there had been any new developments allowing directly placing new instances of an ElementType directly instead of having to set it as the default type and launching the wall command via the aforementioned shortcut.

**Answer:** You can place instances in a model using the
[NewFamilyInstance](http://thebuildingcoder.typepad.com/blog/about-the-author.html#5.25) method.

This has been possible for ages.

You can also programmatically prompt the user to place an instance, where your add-in triggers the placement and asks the user to select the location interactively using the UIDocument.PromptForFamilyInstancePlacement method.

Finally, yes, there have indeed been new developments in this area in Revit 2015 API, e.g. the introduction of the default type API and the new PostRequestForElementTypePlacement method:

- [Default Type API](http://thebuildingcoder.typepad.com/blog/2014/04/whats-new-in-the-revit-2015-api.html#3.02)
- [Document API additions](http://thebuildingcoder.typepad.com/blog/2014/04/whats-new-in-the-revit-2015-api.html#4.02) > UIDocument operations and additions

These enhancements improve the usability of both the fully automated and the interactive placement methods.

#### Commercial Use of the Revit API

**Question:** We are building a Revit plugin that exports the model to another software.

Is any license required to use it for commercial purposes?

**Answer:** Of course you may use the Revit API to create custom applications, both for your own use and for commercial purposes.

There is no need for any additional license.

The user of your add-in simply needs to have a standard Revit license to run it.