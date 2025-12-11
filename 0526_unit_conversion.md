---
ai_optimized: true
author: Jeremy Tammik
complexity_score: 1.6
content_type: qa
optimization_date: '2025-12-11T11:44:14.106141'
original_url: https://thebuildingcoder.typepad.com/blog/0526_unit_conversion.html
post_number: '0526'
reading_time_minutes: 3
series: general
slug: unit_conversion
source_file: 0526_unit_conversion.htm
tags:
- revit-api
- views
title: Unit Conversion and New Blogs
word_count: 631
---

### Unit Conversion and New Blogs

Quite a while back, I published a
[unit conversion utility](http://thebuildingcoder.typepad.com/blog/2009/08/unit-conversion.html) for Revit 2010.
Let's take a fresh look at that, and also at a
[couple of new Revit blogs](#2).

We discussed a couple of other issues related to units since then, such as the
[unit suffix and ProjectUnit SDK sample](http://thebuildingcoder.typepad.com/blog/2009/10/unit-suffix-and-the-projectunit-sdk-sample.html) and a look at
[voltage units](http://thebuildingcoder.typepad.com/blog/2010/06/voltage-units.html).
The latter also includes an overview of some other unit related posts.

Anyway, [Rod Howarth](http://www.rodhowarth.com/) picked up on the unit converter utility and ported it to Revit 2011.
He says:

I was reading your old post on a reader-contributed
[Unit Conversion](http://thebuildingcoder.typepad.com/blog/2009/08/unit-conversion.html) Revit
utility today, and I couldn't find an updated version (and I don't think it made it into the full SDK from what I could tell).
So I decided to quickly convert it to Revit 2011, and send it through to you.

It's a handy utility, especially for us Aussies with our metric system.
I've just gone through and done the basic conversions that are required for the new namespaces, and made a manifest file for it.
It seemed to run fine for me though I am fairly busy so haven't done extensive testing.
Here is
[UnitConversion2011.zip](zip/UnitConversion2011.zip) containing
the complete source code and Visual Studio solution and including the binary executable files in the bin subfolder.

This is what it looks like running in Revit MEP 2011 and gives you an idea of the functionality provided:
![UnitConversion for Revit 2011](img/UnitConversion2011.png)

Many thanks to Rod for converting and sharing this useful utility!

And yes, it didn't seem to make it into the SDK after all, did it?

For more details on the unit conversion tool, please refer back to the
[original post](http://thebuildingcoder.typepad.com/blog/2009/08/unit-conversion.html).

#### New Revit API Blog

Stephen Germano, Project Director, and his colleagues at the
[ibd Resource Group](http://www.bimadvent.com),
have started up a new Revit API blog and say that they hope for a couple blog posts a week; check it out at

- [ibdrg.typepad.com/ibd-resource-group](http://ibdrg.typepad.com/ibd-resource-group) or- [www.bimadvent.com/Blog](http://www.bimadvent.com/Blog).

#### New Polish Revit Blog

Andrzej Samsonowicz, Application Engineer in the Warsaw office of Autodesk, has successfully launched a new blog about Autodesk Revit in Polish:

- [revit-pl.typepad.com/my-blog](http://revit-pl.typepad.com/my-blog).

The new blog promotes BIM, provides tips and tricks, suggests best practices and workflows, answers common problems faced by Revit users, and presents the latest technologies for AEC users.

It is part of an effort from Autodesk in Poland and the International Community team to:

- Further develop the [user community in Polish language](http://communities.autodesk.com/poland).- Support the connection of employees with customers in their particular market.- Promote the creation of relevant content in Poland.

I would encourage you to take a look, read, subscribe, and engage if you are a Polish speaker (or use Google Translate :-).

Actually, you can get an
[English translation](http://translate.google.com/translate?js=n&prev=_t&hl=en&ie=UTF-8&layout=2&eotf=1&sl=auto&tl=en&u=http%3A%2F%2Frevit-pl.typepad.com%2Fmy-blog%2F) directly,
courtesy of
[Scott Sheppard](http://labs.blogs.com/its_alive_in_the_lab/2011/01/new-aec-bim-blog-in-polish.html).
I am pleasantly surprised how readable this machine translation is.