---
ai_optimized: true
author: Jeremy Tammik
complexity_score: 2.0
content_type: qa
optimization_date: '2025-12-11T11:44:15.341859'
original_url: https://thebuildingcoder.typepad.com/blog/1120_no_family_binding_map.html
post_number: '1120'
reading_time_minutes: 2
series: parameters
slug: no_family_binding_map
source_file: 1120_no_family_binding_map.htm
tags:
- family
- parameters
- revit-api
title: Cannot Get BindingMap of a Family Document
word_count: 391
---

### Cannot Get BindingMap of a Family Document

Here is an issue trying to access the binding map of a family document that just re-arose as a new ADN support case.

It took me a while (thank you,
[Google](http://lmgtfy.com/?q=revit+api+))
to discover that I had actually already answered the very same issue in the past in the Revit API forum discussion thread
[cannot get binding map of a family document error 2014](http://forums.autodesk.com/t5/Revit-API/Cannot-Get-Bindingmap-of-a-family-Document-Error-2014/m-p/4317835).

Therefore, I think it is best for both you and me if I promote it to a blog post here and now to ensure it gets found more easily in future:

**Question:** My add-in adds a shared parameter to a family document.
It has been working perfectly for years now, ever since 2009, in fact.
In Revit 2014, however, it fails and throws an exception saying "Cannot get BindingMap of a family document."

Simply accessing the Document.ParameterBindings property causes this error.

What is the problem, please?

Above all, how can I fix it?

**Answer:** The possibility to add a shared parameter via the binding map was removed for families in Revit 2014.

The reason for this is that you can use an overload of the FamilyManager.AddParameter method instead.

In your code, you are setting up a shared parameter definition, creating a category set, and adding a shared parameter binding using code that only applies to the project environment, not in the family context.

In a family document, you can simply use the appropriate overload of the AddParameter method to do all these things for you in one single fell swoop.

The Revit 2014 API help file RevitAPI.chm entry on the Document.ParameterBindings property explicitly states that it throws an Autodesk.Revit.Exceptions.InvalidOperationException when the document is a family document.

Sorry that this affects your existing code.

The modification to fix the error should be easy.

Just remove the part of the code that accesses the binding map and tries to bind the parameter to the family document, call the AddParameter overload to add a shared parameter and achieve all the required steps for you automatically, including setting up a category set and an instance binding, and all should be well.