---
ai_optimized: true
author: Jeremy Tammik
complexity_score: 1.8
content_type: qa
optimization_date: '2025-12-11T11:44:15.129798'
original_url: https://thebuildingcoder.typepad.com/blog/1017_location_att_key_sched.html
post_number: '1017'
reading_time_minutes: 4
series: general
slug: location_att_key_sched
source_file: 1017_location_att_key_sched.htm
tags:
- elements
- family
- filtering
- geometry
- levels
- parameters
- revit-api
- rooms
- schedules
- views
title: Determining Location Attributes and Retrieving Key Elements
word_count: 746
---

### Determining Location Attributes and Retrieving Key Elements

Here are two issues that we have visited in the past and seem to be worth recapitulating and summarising, on determining the
[location attributes](#2) of
an element and retrieving all
[key elements in a key schedule](#3).

#### Determining the Location Attributes of a Component Element

**Question:** I have extracted data from a Revit model via DBLink.

Now I would like to establish a relation between a component element and its location-centric attributes, specifically its Level, Space, and the Room it is located in.

For example, for a CAV Terminal Unit component, I can determine the building that it is in and the Level that it is on, but not the Space or Room containing it.

Can you provide any insight on how to properly identify these attributes?

**Answer:** There are a number of different possibilities.

The simplest and most common one is to use the FamilyInstance Room and Space properties.

All elements that can be cast into FamilyInstance can be queried for their Level, Zone, Space, Room Name and Room Number properties.

You need to ensure that your models are correctly constructed, of course, or these properties may be unreliable.
If you find that to be the case, consult with an application engineer and product experts on how to fix it and ensure that this is avoided in the future.

If these properties do not fulfil your needs, or your element is not a family instance, there are several other ways as well.

Basically, all you need is some central point on the element of interest that can be passed in to the Document GetRoomAtPoint or GetSpaceAtPoint methods.

Such a point can be determined from the element Location property, bounding box, or from the element geometry.

We just recently looked at another example of using the GetRoomAtPoint method to
[determine the neighbours of a room](http://thebuildingcoder.typepad.com/blog/2013/09/room-neighbours.html).

For more complex queries, you can also use a filtered element collector in conjunction with the element location or bounding box.

In general, all search for and access to Revit database elements works through filtered element collectors.
It is the most efficient way, and in fact the only way.

I see you also mention DBLink.
I cannot say much about that, since it is a closed subscription product.
You would have to ask product support whether you can customise it or include additional properties in the output it generates.
What you certainly can do is use DBLink on its own as is, run your custom add-in generating additional data in parallel with it, and unite the two data streams in your analysis database.

#### Retrieving the Key Elements Used in a Key Schedule

**Question:** Is it possible to get all key elements used in a key schedule?

For example, I have a room and create a room style schedule.
The parameter that is added to rooms is of storage type ElementId.
I can then get that element from the document and it has parameters for each other parameter used in the key schedule that can be read or set.
This is great and gets me most of what I want.

However, what if there are keys that are not used anywhere in the model at present?
I don't have a room to get the objects from.
The category on the key objects is null so I can't filter that way.
I tried to use an ElementParameterFilter even though it is a slow filter to find the key value parameter (and just made it a numeric greater than -1 filter), but this returned no elements to me at all.

Is it possible to get the unused keys?

**Answer:** To provide some basic understanding on this topic, you can start by looking at the Revit user interface Wikihelp on
[key schedules](http://wikihelp.autodesk.com/Revit/enu/2012/Help/Revit_User%27s_Guide/1394-Document1394/1476-Schedule1476/1479-Key_Sche1479) and
the further explanations provided by this article on
[understanding schedule keys](http://knowingwhatyoudontknow.blogspot.ch/2009/01/understanding-schedule-keys-in-revit.html).
The comments on the latter discussion may be of interest for you and your issue as well.

A surprisingly simple method to access the set of all elements listed in a schedule is to instantiate a filtered element collector providing the schedule view as an initial input argument.
That returns exactly the desired elements.
This works for key schedules as well.