---
ai_optimized: true
author: Jeremy Tammik
complexity_score: 2.5
content_type: qa
optimization_date: '2025-12-11T11:44:14.431847'
original_url: https://thebuildingcoder.typepad.com/blog/0704_benchmarking.html
post_number: '0704'
reading_time_minutes: 3
series: general
slug: benchmarking
source_file: 0704_benchmarking.htm
tags:
- csharp
- elements
- family
- filtering
- levels
- parameters
- revit-api
- schedules
title: Timer Code for Benchmarking
word_count: 535
---

### Timer Code for Benchmarking

Two topics today:

- [Benchmarking add-ins](#1)- [Hands-on training](#2)

#### Benchmarking Add-ins

Just like most programming issues, many Revit API tasks can be achieved in numerous different ways.
The optimal approach often depends on your individual needs.
The only way to find out which one that may be is to test it.
In order to test it effectively, you need to measure the results.
One of the most important aspects is often the time required, the execution performance.
Measuring a short duration of time precisely makes use of a timer.
The .NET framework provides several timer classes for this purpose.
Here is a related question that just came up on this topic:

**Question:** How can I add a timer to my Revit 2012 C# code to find out how much time it takes to execute the whole command?
Is there a way to add a form to show total time taken to run the command?
I am not quite sure how to wrap the timer class around Revit code.

**Answer:** The Building Coder samples include several examples of benchmarking both entire commands and individual function calls or sections of code.

They define the JtTimer class, which is one possible wrapper for the .NET Stopwatch class that you may find useful.
Besides the Stopwatch wrapper, it also implements a TimeRegistry for keeping track of and summing up the results a number of different Stopwatch instances:

- [Performance profiling](http://thebuildingcoder.typepad.com/blog/2010/03/performance-profiling.html)- [Collector benchmark](http://thebuildingcoder.typepad.com/blog/2010/04/collector-benchmark.html)- [Parameter filter benchmark](http://thebuildingcoder.typepad.com/blog/2010/06/element-name-parameter-filter-correction.html)- [Filtered element collectors](http://thebuildingcoder.typepad.com/blog/2010/10/filtered-element-collectors.html)- [Level filter benchmark](http://thebuildingcoder.typepad.com/blog/2010/10/level-filter-benchmark.html)- [Purging text note types](http://thebuildingcoder.typepad.com/blog/2010/11/purge-unused-text-note-types.html)- [The Stopwatch class](http://thebuildingcoder.typepad.com/blog/2010/11/c-and-net-little-wonders.html)- [XML family usage report](http://thebuildingcoder.typepad.com/blog/2010/12/xml-family-usage-report.html)- [Type filter benchmark](http://thebuildingcoder.typepad.com/blog/2011/01/type-filter-benchmark-update.html)

#### Hands-on Revit API Trainings

If you are interested in face-to-face hands-on training in the Revit API, you should take a regular look at the ADN
[Training Course Schedule](http://www.adskconsulting.com/adn/cs/api_course_sched.php) and
enter "Revit API" as the course name.
Here is the short
[course description](http://usa.autodesk.com/adsk/servlet/item?siteID=123112&id=6364883).

The next scheduled class that I will be conducting is taking place in Munich, Germany, starting April 25th
([register](http://usa.autodesk.com/adsk/servlet/item?id=6703509&siteID=123112&cname=Revit%20API,%20Munich,%20Apr%2025%202012,%20201213)).

If you are interested in participating in a class, you will benefit enormously by
[preparing appropriately](http://thebuildingcoder.typepad.com/blog/2012/01/preparing-for-a-hands-on-revit-api-training.html).
The material we provide for that can also be used just as well to efficiently learn the basics of the Revit API for yourself all on your own.