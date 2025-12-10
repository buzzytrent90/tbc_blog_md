---
post_number: "0460"
title: "Power to the User (and Application)"
slug: "prompt_user"
author: "Jeremy Tammik"
tags: ['elements', 'levels', 'revit-api', 'transactions', 'views']
source_file: "0460_prompt_user.htm"
original_url: "https://thebuildingcoder.typepad.com/blog/0460_prompt_user.html"
---

### Power to the User (and Application)

Here is a little discussion that reminds me of the one on
[prompting the user](http://thebuildingcoder.typepad.com/blog/2009/07/prompt-the-user-for-pinning-and-ifc-export.html) to
pin a link and to export to IFC instead of trying to solve the whole thing programmatically.
It started from the following question:

**Question:** I want to display a warning message before deleting an element.
It is possible to capture the element before deleting it from the document?

We believe the DocumentChanged event can capture the deleted elements, i.e. after an element is deleted from the document.
But we want to capture it before the deletion.

Is there any way to achieve this?

**Answer:** You are right in your statement that the DocumentChanged event will report elements that were deleted after the fact.

As explained in the Revit API help file RevitAPI.chm, this event is raised whenever a Revit transaction is either committed, undone or redone. This is a read-only event, designed to allow you to keep external data in synch with the state of the Revit database. To update the Revit database in response to changes in elements, use the IUpdater framework.

The IUpdater framework is the only other element level notification mechanism. It will also enable you to determine when elements are being deleted and react to these changes, and it also allows you to make changes to the document and register them in the same transaction as the original changes that triggered the notification.

I do not see any way in which you can cancel the transaction, though, so I do not think that you can prevent the deletion from taking place even using the IUpdater interface.

Can you explain why you need to do this? Maybe we can find another approach to solve the issue.

**Response:** Actually we want to alert the user before deleting the elements from the document, particularly views.

Is there any other work around for this?

**Answer:** One approach that I can suggest that is not really a workaround at all, but a different way of looking at things, is to avoid doing any work on this yourself and let the Revit application do it for you, making use of the powerful features already available.

One such feature that exactly suits your requirement is the Undo feature ... what you describe is exactly what it is there for.

Using the DocumentChanged notification, you can easily detect if any views have been deleted.

If that is the case, you can pop up a dialogue box saying "The view so-and-so has been deleted", or "X views were deleted", and "Are you really sure you want to do this? If not, please click the Undo button."

I wrote about a similar way to use existing Revit functionality and work with it instead of against it in the post mentioned above on
[prompting the user](http://thebuildingcoder.typepad.com/blog/2009/07/prompt-the-user-for-pinning-and-ifc-export.html).

This kind of thinking and development empowers the user and makes use of existing features instead of working against them.