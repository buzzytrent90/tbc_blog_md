---
ai_optimized: true
author: Jeremy Tammik
complexity_score: 2.2
content_type: qa
optimization_date: '2025-12-11T11:44:13.709858'
original_url: https://thebuildingcoder.typepad.com/blog/0302_ifamilyloadoptions.html
post_number: '0302'
reading_time_minutes: 3
series: structural
slug: ifamilyloadoptions
source_file: 0302_ifamilyloadoptions.htm
tags:
- csharp
- family
- parameters
- revit-api
- structural
title: IFamilyLoadOptions and GEMini
word_count: 616
---

### IFamilyLoadOptions and GEMini

First of all, a new lunar year is beginning, and with it the year of the tiger. Happy Chinese New Year to all!

![Year of the tiger](img/year_of_the_tiger_3.png)

We discussed the new
[Revit Family API](http://thebuildingcoder.typepad.com/blog/2009/08/the-revit-family-api.html) in
some depth and presented
[labs](http://thebuildingcoder.typepad.com/blog/2009/10/revit-family-creation-api-labs.html) to
help start making efficient use of it.
One issue in the context of families and the loading of families remained unresolved, however, and we have some questions on this from
[Prasad](http://thebuildingcoder.typepad.com/blog/2009/10/revit-2010-subscription-pack.html?cid=6a00e553e1689788330120a67a6b82970c#comment-6a00e553e1689788330120a67a6b82970c) and
[Lars Rådman](http://thebuildingcoder.typepad.com/blog/2009/08/the-revit-family-api.html?cid=6a00e553e16897883301287655e694970c#comment-6a00e553e16897883301287655e694970c) that
still remain unanswered, namely on how to use the IFamilyLoadOptions interface.
So far nobody was actually able to provide a working example.

Now Gamal Kira of
[GEM Team Solutions](http://www.team-solutions.de)
solved this problem and very friendlily provided a sample application illustrating the solution.

The IFamilyLoadOptions interface defines call-back functionality for specific family load situations through the two events OnFamilyFound and OnSharedFamilyFound, called when a family or a shared family being loaded was already present in the target document, respectively.

They both take an output argument in which the event handler has the opportunity to specify whether the existing family's parameter values should be overwritten.

Unfortunately, this does not currently work the way one might expect. Once a family has been loaded into the project, it will not be reloaded later, even if it was modified, and attempting to use this event handler throws an exception.

One would mostly want a family to be reloaded if it has been edited, even if it is already loaded.

The trick to this that Gamal has uncovered is to call the LoadFamily method outside of a command context.
Normally, the Revit API is only usable within the command context, although read-only access sometimes also works outside of it, e.g. when called from a modeless dialogue.
This is the first example that I know of where some functionality actually requires a non-command context.

Here is what Gamal has to say about his example solution:

I isolated the ILoadOptions solution in a separate project
[RevitTest](zip/RevitTest.zip).

#### Installation and GEMini

To test, register the command in Revit.ini in the normal fashion.
If you like, you can use our GEMini.exe registration tool, which is included.
It works through an XML configuration file.
Simply adapt the path specified in the batch file Setup-Debug-RevitIni.bat and run it.
If you are interested in the source for GEMini, please feel free to ask.
It works with all versions of Revit, including 64-bit and 32-bit ones running on a 64-bit system.

#### IFamilyLoadOptions and RevitTest

The external command GEM RevitTest displays a dialogue box to initiate the LoadFamily call.
This dialogue can be displayed either as a modal or modeless form.
From the form, LoadFamily can be called either with or without the IFamilyLoadOptions call-backs installed.
When modal, the LoadFamily operation is executed within the command context, whereas the modeless dialogue executes it outside of the command context.
The call to LoadFamily always throws an exception when called within the command context, regardless of whether the IFamilyLoadOptions call-backs are installed or not.
From the modeless version, the IFamilyLoadOptions call-back works.

Very many thanks to Gamal for all his research and for providing this valuable sample!

![Year of the tiger](img/year_of_the_tiger_1.png)
![Year of the tiger](img/year_of_the_tiger_2.png)