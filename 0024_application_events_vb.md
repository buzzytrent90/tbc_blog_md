---
ai_optimized: true
author: Jeremy Tammik
complexity_score: 1.3
content_type: qa
optimization_date: '2025-12-11T11:44:13.246492'
original_url: https://thebuildingcoder.typepad.com/blog/0024_application_events_vb.html
post_number: '0024'
reading_time_minutes: 1
series: general
slug: application_events_vb
source_file: 0024_application_events_vb.htm
tags:
- csharp
- references
- revit-api
title: Application Events in VB
word_count: 179
---

Events, Getting Started, VB

### Application Events in VB

I do not work much in VB, but somebody recently asked how to subscribe to the Revit application events in that language. There is no VB sample for this in the Revit SDK samples, so I created the following little solution to demonstrate it.

The steps to create it are exactly the same as for a C# project, and were actually described in detail for both C# and VB in the post on
[Debugging a Revit Add-In](http://thebuildingcoder.typepad.com/blog/2008/09/debugging-a-rev.html), but here is the really short version:

- Create a new class library project
- Reference RevitAPI.dll
- Set its 'Copy Local' flag to false
- Derive your class from IExternalApplication
- Implement the member methods

I have copied the full Visual Studio solution AppEventsVb here. It is very straight forward to create from scratch, actually, and using Intellisense as much as possible really helps. For instance, the moment I typed 'Implements IExternalApplication.OnShutdown', the skeleton code for two methods is automatically added by Visual Studio.