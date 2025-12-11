---
ai_optimized: true
author: Jeremy Tammik
complexity_score: 1.6
content_type: qa
optimization_date: '2025-12-11T11:44:13.442594'
original_url: https://thebuildingcoder.typepad.com/blog/0147_shared_param_rfa.html
post_number: '0147'
reading_time_minutes: 2
series: family
slug: shared_param_rfa
source_file: 0147_shared_param_rfa.htm
tags:
- family
- parameters
- revit-api
- windows
title: Adding a Shared Parameter to an RFA File
word_count: 499
---

### Adding a Shared Parameter to an RFA File

Back again from my vacation in Avignon.
We had a wonderful drive back up again through the gorges of the
[Buëch river](http://en.wikipedia.org/wiki/Buech)
with a beautiful camp near Rosans.
I knew the place from last year, and here are two photos from that visit.
Here is Cornelius at the camp fire in the evening:

![Cornelius at the camp fire near Rosans](img/rosans_cornelius_campfire.jpg)

Here is our camp site in the early morning dew:

![Our camp site in the morning dew](img/rosans_morning_dew.jpg)

We were lucky with the weather, sleeping outside with no roof, preferring the open sky to a tent, and it started raining just after we left but not before.

Looking at old and new subjects to write about, I just discovered a rather overdue issue, a question raised quite a while ago by Arun Shankar of
[Reed Business](http://www.reedbusiness.com)
in a comment on the discussion on
[adding a shared parameter to a DWG file](http://thebuildingcoder.typepad.com/blog/2008/11/adding-a-shared-parameter-to-a-dwg-file.html):

**Question:**
Is it possible to add a shared parameter to an RFA file using the DotNet Revit API?
The example for adding shared parameters to a DWG file does not work for RFA files.
Can you please post some sample code for adding shared parameters to an RFA file?

**Answer:**
As pointed out in a private note by
[Matt Mason](http://cadappdev.blogspot.com),
adding a shared parameter to an RFA file is a completely different issue from adding it to a DWG file instance in a project.
In the latter case, we work in the project file and bind a shared parameter to a certain category specific to the inserted DWG file.
For an RFA file, you wish to define the shared parameter in the content, and the first step is to add it to the family itself.

You can add a shared parameter to an RFA file as a type or instance parameter by using the new Family API provided in Revit 2010.
It provides the new method FamilyManager.AddParameter().
Its use is demonstrated by the Revit SDK samples AutoParameter, DWGFamilyCreation and WindowWizard, which are located in the FamilyCreation subdirectory of the Samples folder.
It provides the following overloads:

- AddParameter(ExternalDefinition, BuiltInParameterGroup, Boolean): Add a new shared parameter to the family.- AddParameter(String, BuiltInParameterGroup, Category, Boolean): Add a new family type parameter to control the type of a nested family within another family.- AddParameter(String, BuiltInParameterGroup, ParameterType, Boolean): Add a new family parameter with a given name.

To add a shared parameter, you would use the overload taking an ExternalDefinition argument.
DWGFamilyCreation and WindowWizard both only add standard type parameters using the third overload with the fourth isInstance argument set to false, whereas AutoParameter uses both the first and third overloads to create instance, type and shared parameters.

Very many thanks to Matt for pointing this out!