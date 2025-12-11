---
ai_optimized: true
author: Jeremy Tammik
complexity_score: 2.5
content_type: qa
optimization_date: '2025-12-11T11:44:14.553446'
original_url: https://thebuildingcoder.typepad.com/blog/0768_prompt_user.html
post_number: 0768
reading_time_minutes: 3
series: general
slug: prompt_user
source_file: 0768_prompt_user.htm
tags:
- elements
- revit-api
- selection
title: Prompt User to Avoid Modeless Interaction
word_count: 559
---

### Prompt User to Avoid Modeless Interaction

Once again on the topic of
[asynchronous interaction](http://thebuildingcoder.typepad.com/blog/2010/04/asynchronous-api-calls-and-idling.html) with
Revit, which is not supported by the Revit API, and often worked around using the
[Idling](http://thebuildingcoder.typepad.com/blog/idling) and
[external events](http://thebuildingcoder.typepad.com/blog/2012/04/idling-enhancements-and-external-events.html)...

As I recently pointed out, no Revit API methods at all can be called outside a valid Revit API context, not even the new
[DoDragDrop method](http://thebuildingcoder.typepad.com/blog/2012/04/drag-and-drop-api.html#4) or
Application.IsQuiescent property.

I already presented two convincing examples in the past of how you could work around a lack of direct API access by interacting intelligently with the user instead:

- [Prompt the User for Pinning and IFC Export](http://thebuildingcoder.typepad.com/blog/2009/07/prompt-the-user-for-pinning-and-ifc-export.html)- [Power to the User (and Application)](http://thebuildingcoder.typepad.com/blog/2010/10/power-to-the-user-and-application.html)

In the first, lack of certain API functionality was worked around by prompting the user to execute an action instead.

In the second, an action was detected that could not be undone programmatically, but a warning can easily be displayed allowing the user to undo if she so desires.

Here is an example by Paul Vella of Autodesk Consulting in Melbourne, Australia, of how you can easily and safely avoid a potentially complex and risky modeless interaction by providing an adequate user interface instead:

**Question:** I have a command that identifies a list of elements from the Revit model, and displays them in a modeless dialog to the user. The dialog is modeless so that the user can interact with the model, but I also want the user to be able to change the current selection (and zoom to that item) by selecting items from my dialog.

The zoom to part works, and the selection appears to change, but the properties panel does not update correctly.

Apparently the issue is that the Revit command has ended once the dialog has been displayed, and therefore my selection code runs outside of a command and lacks a valid Revit API context.

Is it possible to programmatically create a command context or otherwise to get my selection code to work, and have the properties panel update correctly?

One idea I had was to have a separate command button to do the selection, and call this programmatically from my form. But I could not work out how to programmatically click a PushButton on the ribbon.

**Answer:** Nope, I am sorry to say it is not possible to create a command context in the way you describe, nor does the Revit API provide any way to programmatically simulate a ribbon push button click.

The only way I know to safely achieve what you are aiming for is to make use of the
[Idling event or the new external event framework](http://thebuildingcoder.typepad.com/blog/2012/04/idling-enhancements-and-external-events.html).

**Response:** The solution I went with was to create a new button on the ribbon entitled 'Zoom to Sync Error'.
When clicked, this button determines the currently selected item in the modeless form, which is accessible through a singleton instance.