---
ai_optimized: true
author: Jeremy Tammik
complexity_score: 5.8
content_type: qa
optimization_date: '2025-12-11T11:44:14.959404'
original_url: https://thebuildingcoder.typepad.com/blog/0943_duplicate_views.html
post_number: 0943
reading_time_minutes: 9
series: views
slug: duplicate_views
source_file: 0943_duplicate_views.htm
tags:
- elements
- family
- filtering
- revit-api
- rooms
- schedules
- transactions
- views
- walls
- windows
title: Copy and Paste API Applications and New Modeless Assertion
word_count: 1877
---

### Copy and Paste API Applications and New Modeless Assertion

Here is a detailed description of one of the most exciting Revit 2014 API features that you should have heard about by now and promises numerous uses, prompted by a question from a developer on duplicating views from one document to another.
Another couple of uses of it cropped up in the past, so it seems like an overview is called for:

- [Duplicating a view from one document to another](#2)
- [Solving the non-unique unique id problem](#3)
- [Managing materials in adsklib or template files](#4)
- [Transferring groups between projects](#5)
- [Copy family symbols to another project](#5.2)
- [Revit API use in modeless form throws an exception](#6)

#### Duplicating a View from one Document to Another

**Question:** I need to perform the following action programmatically:

Insert Tab > Insert from File > Insert view from file > Select File > Select view in this file.

Is there some way to achieve this via API in Revit?

**Answer:** Yes, you are in luck, this is covered by the new Revit 2014 API.

And even more luckily, the development team selected very good and suitable API samples, so the exact workflow you describe is covered by the new Revit 2014 DuplicateViews SDK sample illustrating the new copy and paste API.

Look at the second bullet item in the overview of
[Revit 2014 API news](http://thebuildingcoder.typepad.com/blog/2013/03/revit-2014-api-and-room-plan-view-boundary-polygon-loops.html),
mentioning the new copy and paste API.

Here is the description from the
[What's New section](http://thebuildingcoder.typepad.com/blog/2013/04/whats-new-in-the-revit-2014-api.html) of
the Revit API help file RevitAPI.chm, reproduced here:

#### Copy & paste elements

The new methods:

- ElementTransformUtils.CopyElements(Document, ICollection<ElementId>, Document, Transform)
- ElementTransformUtils.CopyElements(View, ICollection<ElementId>, View, Transform)
- Transform ElementTransformUtils.GetTransformFromViewToView(View, View)

support copy and paste of arbitrary elements. The first overload supports copy within documents, or from document to document. The second overload also support copying within one document or between two documents, but specifically supports copy and paste of view-specific elements.

#### The DuplicateViews SDK Sample

DuplicateViews copies and pastes drafting views and schedules from one document to another , demonstrating use of the CopyPasteOptions, ElementTransformUtils, FilteredElementCollector, IDuplicateTypeNamesHandler and IFailuresPreprocessor classes. The drafting view contents are also copied.

The DuplicateAcrossDocumentsCommand.cs module implements an external command to copy schedules and drafting views across documents and makes use of a number of useful utility classes defined in DuplicateViewUtils.cs.

In detail, DuplicateViews copies all drafting views, image views, and schedules from the currently active document into another document that is currently in memory. All drafting elements in the non-schedule views are also copied, as is all schedule formatting and filtering. It uses API tools to hide any duplicate types warnings that typically arise when doing this from the user interface.

To test this, open any two project documents in Revit. Set the active document to be the document from which you wish to copy views. Launch Add-ins > CopyPaste > Duplicate across documents and enjoy. A popup message will appear listing the number of schedules, drafting views and drafting elements copied by the command.

If you prefer to watch someone else run this sample rather than trying it yourself, it is also demonstrated live in the
[Revit 2014 DevDays presentation recording](http://thebuildingcoder.typepad.com/revit_2014_api/index.html).

#### Solving the Non-unique Unique Id Problem

A couple of times, people mentioned issues caused by copied files, resulting in copied unique ids on certain elements, e.g. thus:

**Question:** Do you have any recommendation on how to handle unique ids in copied files?

We cannot tell people 'do not copy the files', because they sometimes put in a large effort to create a file that is used as a basic starting point for multiple projects.

**Answer:** Yes, you are in luck again. The copy and paste API should solve this.

Documents that are copied obviously contain exact copies the ids of their elements.
This also can happen when using a document as a template to create another document.

Elements created by copying them through the copy and paste API obviously are assigned new unique ids.
That should solve the issue.

Of course, there is no way to change the unique id of an element that already exists.

#### Managing Materials in Adsklib or Template Files

**Question:** I would like to know if it is possible to access data stored in adsklib files through API functions.
The desired functionality would be to load materials, appearance, physical and thermal assets from an external adsklib file to the current active project document.

The aim is to implement a custom materials library with our application using an adsklib file instead of a project template,
as described in the Autodesk wiki on
[managing material libraries](http://wikihelp.autodesk.com/Revit/enu/community/videos/Customize_Revit/Managing_Materials_Libraries_(2013)),
so only the required materials are included and not all, to decrease the project template size and speed up the material browser.

Is this possible? If so, how?

**Answer:** Nope, sorry, as far as I know, there is no API support for this.

From my naive grass roots perspective, the adsklib idea was born quite a few years back and has since been displaced by visions of cloud and mobile.

The file formats have died, and long live the new format, ubiquitous JSON or whatever.

You say you "have been looking into this subject for some time already" ... maybe it is time to start looking into something new?
I see two possibilities for moving forward, besides the solution that you mention of including all materials in the template file:

1. Store the materials in several separate adsklib files, and only load the ones you really need.
2. Use the new Revit 2014 copy and paste API functionality to copy and paste only the required materials from your or some other project.

**Response:** Thank you for thinking of other solutions.

Option 1 probably still requires API access to the adsklib file to programmatically transfer the material assets from the adsklib into the project.

I had a quick look at the copy and paste functionality and think this would be a good solution.
We could use a separate project or template file as our material library instead of the adsklib one and copy/paste the required materials into the current project.

#### Transferring Groups Between Projects

I have been working with the Revit API to create a Revit application that can transfer a group type from one project file to another. I am working on a very large project, so we have to separate it into several project files due to performance. We are using groups to ensure that rooms having the same characteristics always are the same and can be updated quickly. But the model separation creates a problem to ensure that all group types in all project files are updated. We tried to use the link functionality instead of groups, but when there are many instances of a link Revit become very slow. We also considered using family nesting to solve the problem, but we want to be able to include system families as well, so that solution won't work.

We have one project file where we create the Groups, use 'Save Group' to create a project file, then 'Load Group' or 'Reload Group' to get it in to the actual project file (we have 21 project files). So what I want is to create a Revit application that optimizes this workflow by transferring groups from one project file to another. But working with the API has raised a couple of questions:

- Why are the user interface functions 'Load Group', 'Save Group', 'Reload group' not accessible from the API? Are there any plans to make them accessible?
- If I want to transfer a group type from one project file to another, is the only way to do this to create every group member in the receiving project file using the API functions for every member, e.g. Wall.Create and then group the members?
- Is there a way to transfer system families from one project file to another using the API? The user interface function transfer project standards is not available, are there any plans to make it available?
- It seems like Revit is more appropriate for small projects, there are some plans to make Revit better for larger projects? For instance by sharing families (system and loadable) and groups across project files?

**Answer:** I am surprised that you say that Revit is more appropriate for small projects. I was under the impression that the opposite is true.

You may be surprised to hear that the answer to all your questions is **yes**.

You will be glad to hear that a number of improvements in Revit 2014 are specifically targeted at supporting still larger projects, and many of these improvements specifically target the family management and loading performance.

Regarding all of your other questions, I have one single big suggestion to make to you: please take a look at Revit 2014, and especially at the new copy and paste API functionality it provides.
I think that may meet your needs exactly and efficiently.

**Response:** I took a look at the Revit 2014 API and I must say that the copyElement method is just what I need ;-)

It is a much safer way to transfer groups from one project file to another.

#### Copy Family Symbols to Another Project

Joe Ye presents source code demonstrating how to
[copy all symbols of a selected family](http://adndevblog.typepad.com/aec/2013/05/copy-family-between-documents-via-api.html) from
the current project to another.

#### Revit API use in Modeless Form Throws an Exception

Finally, on a completely different topic, several people making illegal use of the Revit API from an invalid context in Revit 2013, e.g. in a modeless form, are now running into a problem.
David Rock describes it in his
[comment](http://thebuildingcoder.typepad.com/blog/2010/06/revit-parent-window.html?cid=6a00e553e168978833017eeae9657f970d#comment-6a00e553e168978833017eeae9657f970d) on
[accessing the Revit main window handle](http://thebuildingcoder.typepad.com/blog/2010/06/revit-parent-window.html):

**Question:** In Revit 2014 I'm having trouble showing a modeless form and calling a transaction whilst inside the form. It is producing an error stating that "starting a transaction from an external application running outside of API context is not allowed".

**Answer:** Yes, it looks as if you are calling the Revit API directly from your modeless dialogue running in another thread.

As you should know, actually, that was never supported.

Now, in Revit 2014, an exception is raised when you make such an attempt.

You are lucky that it worked so far without corrupting anything.

The solution is described in depth here on the blog: simply google for "modeless site:thebuildingcoder.typepad.com", or look at the discussions in the
[Idling](http://thebuildingcoder.typepad.com/blog/idling) category.

In short, make use of the Idling event, or implement an external event, which is a simplified wrapper around that, and base you application on the ModelessForm\_ExternalEvent and ModelessForm\_IdlingEvent SDK samples.

Do not call the Revit API directly, ever.