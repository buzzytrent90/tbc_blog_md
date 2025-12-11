---
ai_optimized: true
author: Jeremy Tammik
complexity_score: 1.4
content_type: qa
optimization_date: '2025-12-11T11:44:13.277765'
original_url: https://thebuildingcoder.typepad.com/blog/0043_new_param.html
post_number: '0043'
reading_time_minutes: 2
series: general
slug: new_param
source_file: 0043_new_param.htm
tags:
- elements
- parameters
- revit-api
title: Defining a New Parameter
word_count: 340
---

### Defining a New Parameter

Here are some frequent questions regarding the creation of new parameters:

I want to store a new parameter in a Revit document.

- Is there a way to do this without binding it to a specific category?
- Is it always necessary to create a shared parameter file in order to store new parameters?
- Do I need such a shared parameter file for each Revit file in which I want to store this new parameter?
- How can I store one single instance of a parameter in a Revit file?

We already discussed some aspects of shared parameters in the previous posts on
[adding a shared parameter to a DWG file](http://thebuildingcoder.typepad.com/blog/2008/11/adding-a-shared-parameter-to-a-dwg-file.html)
and
[exploring element parameters](http://thebuildingcoder.typepad.com/blog/2008/11/exploring-element-parameters.html).
Those posts also mentions the Revit SDK FireRating sample and the
[Revit API introduction labs](http://thebuildingcoder.typepad.com/blog/files/rac_labs_20081117.zip),
especially the suite 4-3-1, 4-3-2, 4-3-3 and 4-4-1, which is relevant in this context.

New parameters can only be created as shared parameters, and they are always associated with one or more specific categories. You can create a shared parameter without explicitly setting up or reading or writing to an associated parameter file, but a valid file path still needs to be specified by the application SharedParametersFilename property, and Revit will write information about the newly created parameter into that file. This is demonstrated by the examples mentioned above.

To store one single instance of a parameter in a Revit model, you can choose to attach it to an object of a category that also only allows one instance in the database. An object that exists once and once only in every Revit model is the project information element of the category BuiltInCategory.OST\_ProjectInformation. Using this to store a visible and an invisible per-document parameter is demonstrated by the Lab4\_4\_1\_PerDocParams command.