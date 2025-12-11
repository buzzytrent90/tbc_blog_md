---
ai_optimized: true
author: Jeremy Tammik
complexity_score: 2.5
content_type: qa
optimization_date: '2025-12-11T11:44:15.427596'
original_url: https://thebuildingcoder.typepad.com/blog/1161_property_dwf_ifc_table.html
post_number: '1161'
reading_time_minutes: 7
series: parameters
slug: property_dwf_ifc_table
source_file: 1161_property_dwf_ifc_table.htm
tags:
- doors
- elements
- family
- parameters
- revit-api
- rooms
- schedules
- views
title: Properties in DWF, IFC, Tables and Extensible Storage
word_count: 1336
---

### Properties in DWF, IFC, Tables and Extensible Storage

Let's discuss a couple of questions that accumulated recently on various aspects of properties in general:

- [Properties in DWF export](#2)
- [Mapping Revit and IFC properties](#3)
- [Storing a table in a project](#4)
- [Updating versus adding new extensible storage schemas](#5)

Before getting to that, I'll briefly mention the happy fact that I yesterday went on my first outdoor climb for a while, up the Balmflue south ridge, a nice long route of 16 rope lengths in the Swiss Jura hills close to Solothurn, just above the
[Balm ruins](https://en.wikipedia.org/wiki/Balm_ruins).

![Balmflue](img/balmflue.jpg)

Back to the Revit API.

#### Properties in DWF Export

**Questions:**

1. The 'Type' properties created through a shared parameter in Revit are not exported in DWF. Only instance properties from a shared parameter are exported. Any idea?

2. Furniture and other family instances have a 'room belonging' property that you can schedule in Revit, but this property can't be found in the DWF. Any idea of how to get this info in the DWF?

3. Surface area and volume are not displayed when exported to DWF in the DWF Room properties. Is it normal? Is there a way to get it?

**Answer:** Since Revit 2013, shared parameters for family instance can be exported to DWF.

So far, though, for family types, only system parameters can be exported:

![DWF properties](img/dwf_property_1.jpeg)

Custom parameters, i.e. family parameters and shared parameters, cannot be exported:

![DWF properties](img/dwf_property_2.jpeg)

For room area and volume, did you check the following DWF export settings?

![DWF properties](img/dwf_property_3.jpeg)

Once properly set, these properties ought to show up:

![DWF properties](img/dwf_property_4.jpeg)

**Response:** Thanks for your help.
I just checked the Area and Volume export options and so it works now!
Fine.
For the other questions, if it's as designed, no worries, we'll find another way to get this.

#### Mapping Revit and IFC Properties

**Question:** We are currently looking at mapping IFC properties against Revit type properties.

I understand from the
[COBie community](http://www.wbdg.org/resources/cobie.php)
(Construction Operations Building Information Exchange) that Autodesk have already gone through this exercise.

Where can I find more information about this?

Essentially we are transferring our shared parameter file into an online database, so any work already undertaken would be hugely beneficial.

**Answer:** The best resources to look at are the notes provided by the open source IFC for Revit project, e.g.:

- [FMHandOverView](http://sourceforge.net/p/ifcexporter/wiki/Notes%20on%20support%20for%20FMHandOverView)
- [Custom parameter mapping](http://sourceforge.net/p/ifcexporter/wiki/Custom%20parameter%20mapping/)

The first item in particular includes a PDF document covering the COBie to IFC mapping that might be of use.

#### Storing a Table in a Project

**Question:** I'm working on a daylighting analysis plug-in and using extensible storage quite heavily in my implementation.

I'm currently saving some data with each Floor object, but I would like to change this to store the data in more of a table format, with one entry per floor, and save this as an object with the model.

I thought I read somewhere on your blog about how to save a singleton data object using ES, but I can't find the post now.

I know how to create array objects in ES, but I'm not sure what to store the data with.

I thought I'd read that ProjectInfo is not the best place, but again, I can't find where I read that, so I may be mistaken.

Can you give me some pointers, or send me the link to the relevant blog post, if I'm remembering correctly that I read it on your blog.

I think I found my answer. I want to use the DataStorage object. I found your entry on it. If you have any other advice for me, please share it, but otherwise, I think I answered my question.

**Answer:** Yes, absolutely right, using a DataStorage element is optimal.

That also keeps your data nicely packaged and tucked away to avoid interfering with others in a workshared environment, as explained in Scott Conover's AU class on
[Revit add-ins that cooperate with worksharing](http://thebuildingcoder.typepad.com/blog/2013/12/au-day-2-worksharing-and-revit-2014-api-roundtables.html#4).

Based on that, I no longer recommend using the ProjectInfo element at all.

#### Updating Versus Adding New Extensible Storage Schemas

**Question:** I have released a version of my software that uses schemas to store data.

I am now working on my next version. I need to expand my schemas to store more data.

When I add data, I can create a new schema and transfer my data to it, or I can create another schema.

Creating another schema seems simplest. Are there any drawbacks to creating another I should look out for? I continue to do this and end up with a large number of schemas, will there eventually be space or time costs?

**Answer:** I assume your issue has to do with extensible storage schemata.

"Create a new schema and transfer my data to it" and "create another schema" sounds almost like the same thing to me.

In any case, and luckily simplifying the issue, I can tell you for sure that you do not really have any choice.

If you need to make any changes whatsoever to an extensible storage schema, the only possibility you have is to define a completely new one.

There is no way to alter an existing one, as explained in the detailed discussion on
[deleting and updating an extensible storage schema](http://thebuildingcoder.typepad.com/blog/2013/08/deleting-and-updating-extensible-storage-schema.html#3).

An example of defining a new schema to replace an existing one and transferring data from the old schema to the new one is provided by the UpgradeSchema sample included in my Autodesk University 2011
[extensible storage class and lab](http://thebuildingcoder.typepad.com/blog/2012/03/melbourne-day-two.html#3).

Finally, I put together a list of all The Building Coder
[extensible storage topics](http://thebuildingcoder.typepad.com/blog/about-the-author.html#5.23) for you.

**Question:** Let me clarify a little further.

I have a schema with ten fields. I need to add one more.

I can create a new schema with eleven fields. That requires me to transfer the data (which as you add more fields, is tedious at best and error-prone at worst). It also results in duplicate data. Not sure if the extra data in old projects would ever become a concern.

Or, I can create a new schema with one field. Now I have two schemas with eleven fields between them. Are there any problems with having lots of active schemas on an element? What if I continue to do this as the program evolves, and end up with five or ten or twenty schemas?

Or, and this is what I came up with after I sent the email, what if I create a map field where I store all my data? Then I have a single schema with a single map. If I have to update one value, I have to pull out the whole map, change one value, then put it back. Is the performance on that significantly different from updating just a string or int field?

**Answer:** Oh, now I see what you mean.

Your idea of a map sounds perfect to me.

The other would certainly also work, but sounds much more complex, with unlimited growing complexity as time goes on.

I like the idea of maintaining one single map and one single schema much better.

The only one who can say anything about the relative performance of the various options is you.

Please
[benchmark it](http://lmgtfy.com/?q=revit+api+building+coder+benchmark),
and let us know what you find out.