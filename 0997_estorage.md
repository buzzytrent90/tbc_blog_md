---
ai_optimized: true
author: Jeremy Tammik
complexity_score: 1.6
content_type: qa
optimization_date: '2025-12-11T11:44:15.084812'
original_url: https://thebuildingcoder.typepad.com/blog/0997_estorage.html
post_number: 0997
reading_time_minutes: 3
series: general
slug: estorage
source_file: 0997_estorage.htm
tags:
- elements
- revit-api
title: Deleting and Updating Extensible Storage Schema
word_count: 612
---

### Deleting and Updating Extensible Storage Schema

We discussed a number of extensible storage schema topics in the past.

Once you've started using it, you may run into the need to [update](#3) or [delete](#2) something.

Discussing that also provides an opportunity to summarise the other results we already have:

#### Deleting an Extensible Storage Schema

**Question:** I am unable to delete my extensible storage schema if the RVT project contains a linked project with the same schema.

First I thought I was doing something wrong, but the ExtensibleStorageUtility SDK sample project DeleteStorage command has the same problem.

**Answer:** Please be aware that all extensible storage schemata are held in memory and shared between all open projects in the current Revit session.

Therefore, if any of the currently open projects is making use of a specific schema, that will prevent it from being deleted.

I hope this is clear and explains the behaviour you are observing.

#### Updating an Extensible Storage Schema

**Question:** I need to add some additional fields to an existing extensible storage schema.
How can I achieve this?

Right now, I am deleting the schema and creating a new one with all the required fields.

My current workaround to delete successfully is to unload the Revit links before deleting the schema.

Another solution I have been considering is to just leave the existing schema and create a new one. That seems a bit messy, though, so I have avoided that so far.

What is the best practice if you have an existing extensible storage scheme and you want to add more fields to it?
A far as I can see, it is impossible to change existing schema fields.

**Answer:** Yes, indeed, schema fields cannot be modified, nor can new fields be added.

Once you publish a schema it is there, immutable, for the rest of the life of the universe.

If you wish to add new fields or modify existing ones, you have to create a new schema.

The old one remains untouched.

You can easily implement an update handler in your add-in that automatically updates all instances of the old schema that it encounters to the new one.

This is demonstrated by the UpgradeSchema sample provided by my extensible storage
[class](http://au.autodesk.com/?nd=event_class&session_id=9263&jid=1725932) and
[lab](http://au.autodesk.com/?nd=event_class&session_id=9726&jid=1725932) at
Autodesk University 2011, which include the complete background information on the topic.

The full
[handouts and sample material](http://thebuildingcoder.typepad.com/blog/2012/03/melbourne-day-two.html#3) are
also available directly from The Building Coder.

Some additional topics that we covered since then include:

- [Project wide data storage](http://thebuildingcoder.typepad.com/blog/2012/05/devblog-devcamp-element-and-project-wide-data.html#4)
- [The DataStorage element](http://thebuildingcoder.typepad.com/blog/2012/05/devblog-devcamp-element-and-project-wide-data.html#5)
- [Extensible storage extension](http://thebuildingcoder.typepad.com/blog/2013/05/effortless-extensible-storage.html)
- [Custom tagging and extensible storage](http://thebuildingcoder.typepad.com/blog/2013/07/sydney-revit-api-training-and-vacation.html#8)

**Response:** I might change my mind on my current solution unloading and loading Revit links to be able to delete the schema, because it might still not solve problem in all cases.
For instance, if users have two Revit projects open and both use the schema, I guess it will not work. All projects have the schema, because I add it on project open.

**Answer:** Yes, I think you can and certainly sooner or later **will** run into problems modifying an existing schema.

Creating a new one is the only safe way to go.