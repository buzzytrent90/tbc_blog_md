---
ai_optimized: true
author: Jeremy Tammik
complexity_score: 3.0
content_type: qa
optimization_date: '2025-12-11T11:44:15.780529'
original_url: https://thebuildingcoder.typepad.com/blog/1327_learn_family_api.html
post_number: '1327'
reading_time_minutes: 6
series: family
slug: learn_family_api
source_file: 1327_learn_family_api.htm
tags:
- doors
- elements
- family
- geometry
- parameters
- python
- revit-api
- views
- walls
- windows
title: Getting Started Creating Families and RFA Files
word_count: 1120
---

### Getting Started Creating Families and RFA Files

Here are some more pointers for getting started generating family definitions and RFA files programmatically:

- [Family types and parameters](#2)
- [Imperial internal Revit database length units](#3)
- [Programmatically generating Revit family RFA files](#4)

I am writing this from the Greek island of
[Euboea](https://en.wikipedia.org/wiki/Euboea),
on which I already
[visited](http://thebuildingcoder.typepad.com/blog/2011/08/associative-section-view-fix.html)
[Limni](http://en.wikipedia.org/wiki/Limni,_Euboea) or
[Λιμνι](http://en.wikipedia.org/wiki/Limni,_Euboea) on
the north end a couple of years ago.

This time, I am heading towards its east and south end instead, on my way to the second
[I love 3D – Athens](http://www.meetup.com/I-love-3D-Athens) meetup
on June 5, followed by the
[AngelHack hackathon](http://angelhack.com/hackathon/athens-2015) coming weekend.

For more details, please refer to
[The 3D Web Coder](http://the3dwebcoder.typepad.com/blog/2015/06/athens-angelhack-hackathon-and-nodejs-rest-workshop.html#2).

#### Family Types and Parameters

**Question:**
I am just getting started programmatically creating Revit families.

Where can I find information about the following topics?

- How to use create RFA files using the Revit API
- Which parameters to use

**Answer:**
There is lots of information available about these topics.

Much too much for me to list here, or point out to you individually.

Actually, it would be preferable for you to ask your questions in the public
[Revit API discussion forum](http://forums.autodesk.com/t5/revit-api/bd-p/160) instead of raising individual cases for each.

That way, other developers could chip in and help you as well, and our discussions would be visible for others to share and learn from.

All that I can say about using Revit families programmatically via the Revit API is published by
[The Building Coder](http://thebuildingcoder.typepad.com).

Search for 'family' or 'families' in the
[list of topic groups](http://thebuildingcoder.typepad.com/blog/about-the-author.html#5).

The most effective way to determine which parameters you can use in your specific context is to create a BIM manually and then explore that using RevitLookup, BipChecker, and other, more intimate,
[database exploration tools such as the Revit Python and Ruby shells](http://thebuildingcoder.typepad.com/blog/2013/11/intimate-revit-database-exploration-with-the-python-shell.html).

The advantage of the Python and Ruby shells is that they enable you to navigate interactively through the model using all the available API constructs, including method calls, iteration, etc., whereas an exploration based on RevitLookup only has access to statically compiled properties that allow navigation by clicking buttons in automatically generated data grid view forms.

#### Imperial Internal Revit Database Length Units

**Question:**
I am working in Germany.

For me, one drawing unit is one mm.

If I generate a Revit family definition RFA file with the APIs, 1 mm turns into more than 3000 mm.

Does this have something to do with the American Unit?

What can I do to fix this?

Is there some parameter that I can use to initialise the variable or switch between European and American units?

**Answer:**
Please do confirm that you are in fact aware of the wealth of online information available on Revit before I answer your question.

Have you worked through the
[getting started material](http://thebuildingcoder.typepad.com/blog/about-the-author.html#2)?

Have you performed an Internet search to answer this question of yours?

One of the main entry points for these kind of questions is the
[Revit API discussion forum](http://forums.autodesk.com/t5/Revit-API/bd-p/160).

Aaaw, I can't resist...

Here are some direct pointers for you:

- [Units](http://thebuildingcoder.typepad.com/blog/2008/09/units.html)
- [Unit Conversion](http://thebuildingcoder.typepad.com/blog/2009/08/unit-conversion.html)
- [Unit Conversion and New Blogs](http://thebuildingcoder.typepad.com/blog/2011/01/unit-conversion-and-new-blogs.html)
- [Internal Imperial Units](http://thebuildingcoder.typepad.com/blog/2011/03/internal-imperial-units.html)
- [Revit 2013 Unit Conversion Utility](http://thebuildingcoder.typepad.com/blog/2013/01/revit-2013-unit-conversion-utility.html)

Please search for yourself next time before asking.

Please note that I am perfectly happy to support you in all your questions, but it is a total waste of both your time and ours if you do not do any research yourself at all.

**Response:**
My question is:

How can I scale the Revit object?

If I have an object that is 1 m long and export it to Revit, it shows up as over 3000 m in length.

I think the problem is the difference between the metric and imperial systems.

Do I have to scale the object when exporting, or is some other possibility, e.g., using some property?

**Answer:**
The [internal Revit database unit for length is imperial feet](http://thebuildingcoder.typepad.com/blog/2011/03/internal-imperial-units.html).

This cannot be changed.

Revit does not support scaling of building elements, since it represents a realistic BIM model.

In reality, you cannot take an existing wall, window or door and simply scale it up by a factor of 2.

You have to change the wall, window or door type to something larger.

Therefore, that is not possible to scale the building elements in Revit either.

Here is a more detailed explanation on
[transforming an element](http://thebuildingcoder.typepad.com/blog/2009/05/transform-an-element.html).

Therefore, you have to convert your geometry to feet before you create it.

The Building Coder provides a large number of examples of how this can be achieved.

#### Programmatically Generating Revit Family RFA Files

**Question:**
I am currently producing Revit families at runtime using the APIs of the Revit installation on the client computer.

In the future I would like to create the complete Revit model at runtime on my server.

Are there any Autodesk tools that I can install on my server to generate Revit RFA files without the Revit API?

**Answer:**
The only tools that I am aware of that fall into the categories you are asking about are the sample add-ins provided in the various software developer kits, e.g. the Revit SDK available from the
[Revit Developer Centre](http://www.autodesk.com/developrevit).

They all require the Revit API, i.e., a full Revit installation on the computer and an instance of Revit up and running to provide a valid Revit API context.

Please look at the family creation samples listed in Revit SDK sample readme file `SamplesReadMe.htm` and The Building Coder topic group on the
[Family API](http://thebuildingcoder.typepad.com/blog/about-the-author.html#5.25).