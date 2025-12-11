---
ai_optimized: true
author: Jeremy Tammik
complexity_score: 4.0
content_type: qa
optimization_date: '2025-12-11T11:44:15.813573'
original_url: https://thebuildingcoder.typepad.com/blog/1344_api_context_form_err.html
post_number: '1344'
reading_time_minutes: 4
series: general
slug: api_context_form_err
source_file: 1344_api_context_form_err.htm
tags:
- family
- geometry
- python
- revit-api
- transactions
title: Revit API Context and Form Creation
word_count: 806
---

### Revit API Context and Form Creation

I am back from my vacation.
I had a wonderful time.

Now, let's look at two important recurring topics that came up while I was away:

- [Revit API Context Summary](#2).
- [Detailed Form Creation Error Information](#3).

Before getting to those, let me mention some of the nice places and mountain tours I was blessed with in the past few weeks:

- Pallanza and the Lago Maggiore:
![Pallanza sunset](file:////j/photo/jeremy/2015/2015-07-14_pallanza/722_sunset.jpg)
- Mont Blanc, mountaneering and climbing around the Cabane d'Orny:
![Sunrise on the glacier above Cabane d'Orny](file:////j/photo/jeremy/2015/2015-07-18_mont_blanc/jogi/2015-07-18_HT_038.jpg)
- Monte Leone above Locarno ([photo album)](https://www.facebook.com/media/set/?set=a.10206313375086153.1073741837.1019863650&type=1&l=46a726a900):
![Monte Leone](file:////j/photo/jeremy/2015/2015-07-22_monte_leone/770.jpg)
- Pizzo Stella and Pizzo Groppera:
![Pizzo Stella](file:////j/photo/jeremy/2015/2015-07-25_pizzo_stella/michael/3732.jpg)
- Les Monnins Dessous and some creative art:
![Painting with Moni at les Monnins Dessous](file:////j/photo/jeremy/2015/2015-08-01_jusoca/moni/20150803_190352.jpg)
- Levantina and Val Sambuco, hiking, cooking, bivouacing:
![Between Levantina and Val Sambuco](file:////j/photo/jeremy/2015/2015-08-08_sambuco_levantina/franziska/3668_jeremy.jpg)

Now I am refreshed and happily back at work, so let's continue with the
[Revit API discussion forum](http://forums.autodesk.com/t5/revit-api/bd-p/160) and
look at those Revit API issues:

#### Revit API Context Summary

This kind of question comes up on a regular basis, and is discussed in the topic group on
[driving Revit from outside](http://thebuildingcoder.typepad.com/blog/about-the-author.html#5.28);
now Daren Thomas provided a beautifully succinct answer that says it all in the StackOverflow discussion on
[starting a transaction from an external application running outside of API context is not allowed](http://stackoverflow.com/questions/31490990/starting-a-transaction-from-an-external-application-running-outside-of-api-conte):

**Question:**
Starting a transaction from an external application running outside of API context is not allowed.
Cannot start transaction.

**Answer:**
Using my magic psychic crystal ball to guess you are asking how to avoid getting this error message in your Revit / RPS plugins, here is a short bit of extra information:

- all Revit API calls should happen inside the "API context"
- this "API context" lives in a special thread
- you are probably accessing the API from another thread
- this often happens when you make a `Form` and call into the API from one of the events, e.g., `Button.OnClick`

What you want to do is figure out how to get back into the API context to execute your code.
There are two main methods for doing this, assuming you have already left the `IExternalCommand.Execute` context:

- polling for jobs inside the `Idling` event
- using an `ExternalEvent`

Since you mentioned RevitPythonShell in the tags, why don't you check out how I used
[external events in my web server example](http://darenatwork.blogspot.ch/2015/06/embedding-webserver-in-autodesk-revit_30.html).

- create an IExternalEventHandler
- implement its Execute method, which runs in the Revit API context
- create an `ExternalEvent` (using the event handler just created)
- when you need to do something in the Revit API context, notify the external event via `my_external_event.Raise()`

As said, The Building Coder provides more details and other samples in the topic group on
[driving Revit from outside](http://thebuildingcoder.typepad.com/blog/about-the-author.html#5.28).

#### Detailed Form Creation Error Information

**Question:**
How can I receive detailed error messages from failed geometry creation calls?

I'm making a call to the doc.FamilyCreate.NewSweptBlendForm method.

However, it throws an exception: Autodesk.Revit.Exceptions.InvalidOperationException.

Unfortunately, the message of the exception is "Cannot create swept blend form."

This does not provide much information, and it's difficult to debug with just this in hand.

Is there any more detailed information that I can access when a call like this fails?

**Answer:**
Unfortunately, the answer is No, not at this moment. The API method makes a direct call to a native method in Revit core which returns an error value, which – again unfortunately – seem to be only either SUCCESS or FAILURE. Additional observation can be made by debugging the concrete case, but for the caller, there is no additional info Revit API can provide. Sorry about that.

The best approach for now might be to create model lines from the input that you are passing into the NewSweptBlendForm method and then execute the sweep in the manual user interface instead. That often provides more detailed info on what is going wrong, as explained in the discussion on
[debugging geometric form creation](http://thebuildingcoder.typepad.com/blog/2009/07/debug-geometric-form-creation.html).