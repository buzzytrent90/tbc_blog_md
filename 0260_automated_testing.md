---
ai_optimized: true
author: Jeremy Tammik
complexity_score: 1.9
content_type: qa
optimization_date: '2025-12-11T11:44:13.637316'
original_url: https://thebuildingcoder.typepad.com/blog/0260_automated_testing.html
post_number: '0260'
reading_time_minutes: 2
series: general
slug: automated_testing
source_file: 0260_automated_testing.htm
tags:
- geometry
- revit-api
- windows
title: AU and Automated Testing
word_count: 423
---

### AU and Automated Testing

AU has been as exciting as ever, with lots of fascinating news and good meetings and discussions with developers and colleagues that I never get to meet except here. Here are some
[quick facts and photos](http://www.solidsmack.com/autodesk-university-2009-quick-fact-photo-smack-up-au2009/2009-12-02).

I visited two great classes yesterday on the Revit API by Harry Mattison and Scott Conover,
who both work with the internal development of the Revit API and really know what they are talking about:

- [CP214-2](http://au.autodesk.com/?nd=class&session_id=5252) Creating and Analyzing New Conceptual Massing Geometry With the Autodesk Revit API
- [CP222-3](http://au.autodesk.com/?nd=class&session_id=5256) Analyze Geometry of Buildings Using the Autodesk Revit API

I am very much looking forward to presenting some of the contents of these classes in upcoming posts.

Meanwhile, here is a great idea from Saikat Bhattacharya on how to automatically trigger automated testing, or perform any other action on a regular basis or at a specified time.

**Question:** Is there a way to call an add-in application from the command line?
We would like to create and run an automated test for our export tool.
This should run during the night, to provide the results the next morning.

**Answer:** We have looked at this and related issues several times in the past:

- [Driving Revit from outside](http://thebuildingcoder.typepad.com/blog/2008/12/driving-revit-from-outside.html)
- [Accessing Revit from a modeless dialogue](http://thebuildingcoder.typepad.com/blog/2009/02/revit-window-handle-and-modeless-dialogues.html)
- [Programmatically triggering a command](http://thebuildingcoder.typepad.com/blog/2009/05/vb-samples-and-other-questions.html#1)
- [Journal file replay](http://thebuildingcoder.typepad.com/blog/2009/07/journal-file-replay.html)

Besides this, another workaround that comes to mind is to make use of the Revit auto-save mechanism.
You could set up an external application which registers a DocumentSaving event handler.
In this event handler, you could check to see if the system time is beyond midnight.
In that case you can implement the functionality to export the data from your active Revit model.
Revit can perform an auto-save of the model based on any interval you choose.
Every time it does so, this mechanism will check the system time and run your custom export logic accordingly.
I hope this helps.

Thank you Saikat for this brilliant idea to add to our growing arsenal of Revit automation tools!