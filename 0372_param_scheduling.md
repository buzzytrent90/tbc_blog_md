---
post_number: "0372"
title: "Parameter Access and Scheduling"
slug: "param_scheduling"
author: "Jeremy Tammik"
tags: ['parameters', 'revit-api', 'schedules', 'sheets']
source_file: "0372_param_scheduling.htm"
original_url: "https://thebuildingcoder.typepad.com/blog/0372_param_scheduling.html"
---

### Parameter Access and Scheduling

Here is a quite typical general inquiry about API access to Revit parameters which comes up in various incarnations from time to time and seems worthwhile sharing:

**Question:** I would like to pre-populate Revit objects with schedule data, e.g. time to order, time to ship, time to manufacture, time to install, etc.
I then need to be able to extract this schedule information from a model and feed it into some external project management databases.

I need to know:

- Can one manually or programmatically add this kind of data to objects and have it automatically picked up by the schedule?- Is there an API or a product feature which allows schedule data to be extracted and used elsewhere?
    - If so, could you please point me to the relevant classes in the API I should be looking at?

**Answer:** This is definitely doable via the Revit API plus standard shared parameter user interface features.
For instance, one can create a tool that dumps a Revit model into an MSProject file and exports the shared parameters into MSProject user data, or into an Excel spreadsheet.
Examples are provided by several Revit SDK samples such as FireRating and RDBLink.
So it is all definitely doable.

While there is no API for interacting with schedules, based on your description it sounds like what you want to do can be accomplished by creating and extracting parameter data through the API.
The Parameter class is where to look in the API, and Chapter 8 of the Developer's Guide covers these topics pretty well.

Many detailed aspect of this topic are discussed in the
[parameters category](http://thebuildingcoder.typepad.com/blog/parameters).