---
post_number: "0487"
title: "Pattern for Semi-Asynchronous Idling API Access"
slug: "async_idling_pattern"
author: "Jeremy Tammik"
tags: ['elements', 'revit-api']
source_file: "0487_async_idling_pattern.htm"
original_url: "https://thebuildingcoder.typepad.com/blog/0487_async_idling_pattern.html"
---

### Pattern for Semi-Asynchronous Idling API Access

Here is another wonderful contribution from Daren Thomas:
[A Pattern for Asynchronously Updating Revit Documents](http://darenatwork.blogspot.com/2010/11/pattern-for-asynchronously-updating.html).

As an attentive reader of this blog, you will certainly remember one of my favourite and most powerful recent projects, the
[modeless loose connector navigator](http://thebuildingcoder.typepad.com/blog/2010/07/modeless-loose-connectors.html).
It retrieves and displays a list of unconnected MEP connectors in a modeless dialogue box.
Being modeless, the dialogue is not within the context of a Revit external command Execute method, nor any other Revit API call-back, and thus has no access to the Revit API, which does not permit
[asynchronous access](http://thebuildingcoder.typepad.com/blog/2010/04/asynchronous-api-calls-and-idling.html).
Happily, the Idling event provides a workaround for that.

The modeless loose connector navigator demonstrates a solution for handling a very specialised need, accessing the Revit API semi-asynchronously to highlight the elements with loose connectors.

Daren's post generalises this solution, allowing a modeless dialogue to queue up a whole collection of actions to be taken, which can then be picked up and processed by the Idling event handler the next time it becomes active.
A wonderful generic solution, including neat features such as:

- Use of the generic Queue template class.- Use of the generic Action delegate, cf. [5. Generic Delegates](http://geekswithblogs.net/BlackRabbitCoder/archive/2010/09/09/c.net-five-final-little-wonders-that-make-code-better-3.aspx).- Locking support to protect against simultaneous access to the queue from the modeless dialogue and the Idling event.- Use of .NET => lambda statements to execute the queued-up tasks.

A truly beautiful job, Daren.
Thank you!