---
post_number: "0266"
title: "Modeless Dialogues in Revit"
slug: "modeless_dialogue"
author: "Jeremy Tammik"
tags: ['elements', 'revit-api', 'transactions', 'views', 'windows']
source_file: "0266_modeless_dialogue.htm"
original_url: "https://thebuildingcoder.typepad.com/blog/0266_modeless_dialogue.html"
---

### Modeless Dialogues in Revit

Here is an overview and summary of a number of discussions that we have already presented previously on driving Revit from outside and interacting with modeless dialogues, which hopefully will help to further clarify the issues involved.

**Question:** How can I use a modeless dialogue in Revit?

I need to display a modeless dialogue in Revit.
On clicking a button in it, an element should be added to the Revit model.
The button can be clicked multiple times and elements should keep on being adding to Revit without closing the dialogue.

How can I programmatically move focus to the Revit application?

Finally, can you please provide some trusted sites for Revit programming tips and examples other than the Autodesk discussion groups?

**Answer:** In principle, using a modeless dialogue box within Revit is a non-issue. Any external command can launch a modeless dialogue box, and the dialogue box can make use of the Revit API in any way it chooses – as long as the external command that launched it has not yet terminated.

From your description, it sounds as if this is the case in your situation, which is good for you and highly recommended.

An example of a modeless dialogue box that remains active only as long as the external command which launched it and makes full use of the Revit API within this context is provided by Joel Karr's
[modeless pressure drop tool](http://thebuildingcoder.typepad.com/blog/2009/10/modeless-pressure-drop-tool.html).
It also shows how to obtain the Revit window handle, which you can use to set focus to the Revit application.

You should be able to design the command you describe along the same lines as that tool.
Just keep the external command running as long as your dialogue is active, and all will be fine.

In some cases, however, one would like to launch a dialogue box which remains active even after the external command that launched it has terminated.
This is a completely different issue and may be problematic.
Why? Because when Revit invokes an external command, it automatically opens a transaction for it.
When you are not within the context of an external command invoked by Revit, you need to
[manage your own transaction](http://thebuildingcoder.typepad.com/blog/2009/01/transactions.html).

The return code from an external command determines how the automatically generated transaction is terminated, and one effect of this is the setting of the
[IsModified document dirty flag](http://thebuildingcoder.typepad.com/blog/2008/12/document-ismodified-property.html).

To summarise, if you are working in a modeless dialogue box invoked by an external command which is still active, you have full access to Revit and can use the API to its full extent.

If you are in some other context, i.e. a modeless dialogue that was not invoked by a command, or the command has already terminated, you cannot make full use of the API and there is no guarantee that you can interact with Revit at all. Basically, you are trying to
[drive Revit from outside](http://thebuildingcoder.typepad.com/blog/2008/12/driving-revit-from-outside.html).

Our recent discussion on
[automated testing](http://thebuildingcoder.typepad.com/blog/2009/12/au-and-automated-testing.html)
links to that discussion and additional related posts.

I hope this fully explains the situation.

Regarding the trusted sites, I personally put my trust and effort into
[The Building Coder](http://thebuildingcoder.typepad.com).
We recently presented an overview of additional
[forums dealing with the Revit API](http://thebuildingcoder.typepad.com/blog/2009/10/revit-api-forums.html).
I cannot say that any of these except the
[Autodesk Developer Network](http://www.autodesk.com/joinadn) ADN
are especially trusted or specifically endorsed in any way.
The others are simply sources of information and places to mutually exchange experience, as always, with no guarantees whatsoever given.