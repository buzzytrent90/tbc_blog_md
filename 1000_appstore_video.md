---
ai_optimized: true
author: Jeremy Tammik
complexity_score: 3.6
content_type: qa
optimization_date: '2025-12-11T11:44:15.092339'
original_url: https://thebuildingcoder.typepad.com/blog/1000_appstore_video.html
post_number: '1000'
reading_time_minutes: 4
series: general
slug: appstore_video
source_file: 1000_appstore_video.htm
tags:
- geometry
- revit-api
- rooms
- views
title: Exchange App Videos and DevTV YouTube Channel
word_count: 853
---

### Exchange App Videos and DevTV YouTube Channel

I recently mentioned the **Exchange App Portathon** event at which you can
[Earn $100 Submitting an App](http://thebuildingcoder.typepad.com/blog/2013/07/earn-100-submitting-an-autodesk-exchange-app.html).

To further support and simplify this task, my colleagues
[Mikako Harada](http://adndevblog.typepad.com/aec/mikako-harada.html) and
[Gopinath Taget](http://adndevblog.typepad.com/aec/gopinath-.html) have
lately been busy creating a series of videos showing exactly how to publish apps to the
[Autodesk Exchange Store for Revit](http://apps.exchange.autodesk.com/RVT/en/Home/Index).

The videos are also accessible from the
[Autodesk DevTV YouTube channel](http://www.youtube.com/channel/UC6Fl_1mFNt0rBqa068Vaxcg) as
part of the
[Publish Your Revit Apps to Autodesk Exchange Store](http://www.youtube.com/watch?v=D_MpPk3nsME&list=PL4CYqBFEfhpJakE-DAuchu785fU_RkAfs) playlist.

|  |  |  |
| --- | --- | --- |
| Topic | Length (min:sec) | Media (recording and slide deck) |
| 1 Goals and Agenda | 1:33 | [video](http://www.youtube.com/watch?v=D_MpPk3nsME)    [ppt](http://adndevblog.typepad.com/aec/ExchangeStorePublisher/1%20Autodesk%20Exchange%20Publish%20Revit%20Apps%20-%20Goals%20and%20Agenda.pptx) |
| 2 Store Overview | 4:19 | [video](http://www.youtube.com/watch?v=e3Q-Ie1sIHA)    [ppt](http://adndevblog.typepad.com/aec/ExchangeStorePublisher/2%20Autodesk%20Exchange%20Publish%20Revit%20Apps%20-%20Store%20Overview.pptx) |
| 3 Preparing Apps for the Store: Guidelines | 8:30 | [video](http://www.youtube.com/watch?v=Ht1i34gFgC0)    [ppt](http://adndevblog.typepad.com/aec/ExchangeStorePublisher/3%20Autodesk%20Exchange%20Publish%20Revit%20Apps%20-%20Preparing%20Apps%20for%20the%20Store_Guidelines.pptx) |
| 4 Preparing Publishing Information | 5:10 | [video](http://www.youtube.com/watch?v=3tRkBbUkiEw)    [ppt](http://adndevblog.typepad.com/aec/ExchangeStorePublisher/4%20Autodesk%20Exchange%20Publish%20Revit%20Apps%20-%20Preparing%20Publishing%20Information.pptx) |
| 5 Submission Process | 9:13 | [video](http://www.youtube.com/watch?v=1N4sZhAXJtA)    [ppt](http://adndevblog.typepad.com/aec/ExchangeStorePublisher/5%20Autodesk%20Exchange%20Publish%20Revit%20Apps%20-%20Submission%20Process.pptx) |
| 6 Publishing Revit 2013 Apps | 2:37 | [video](http://www.youtube.com/watch?v=3kHraYsv4tU)    [ppt](http://adndevblog.typepad.com/aec/ExchangeStorePublisher/6%20Autodesk%20Exchange%20Publish%20Revit%20Apps%20-%20Publishing%20Revit%202013%20Apps.pptx) |
| 7 FAQ | 3:49 | [video](http://www.youtube.com/watch?v=8yblBDUzudw)    [ppt](http://adndevblog.typepad.com/aec/ExchangeStorePublisher/7%20Autodesk%20Exchange%20Publish%20Revit%20Apps%20-%20FAQ.pptx) |
| 8 Summary | 1:57 | [video](http://www.youtube.com/watch?v=iwPmHcFwCJc)    [ppt](http://adndevblog.typepad.com/aec/ExchangeStorePublisher/8%20Autodesk%20Exchange%20Publish%20Revit%20Apps%20-%20Summary.pptx) |

#### Free Interactive eBook on Revit Architecture 2014

Here is a short note to point out that a free interactive eBook is available explaining the
[new features of Revit 2014](http://www.cadlearning.com/revit-2014-new-features).

#### No Revit API Class Diagram

A recent
[query](http://forums.autodesk.com/t5/Autodesk-Revit-API/2014-Revit-Object-Model/m-p/4357581#M4355) on
the Revit API discussion forum asked again for a Revit API object model or class diagram:

**Question:** I worked with the Revit 2010 API.
Back then there was a graphical representation of the Revit Object Model in PNG format.
I have not been able to locate one for the Revit 2014 Object model.
Does anyone know if one exists?

**Answer:** Actually, the Revit 2010 version is indeed the last one known.

I already mentioned that
[no Revit API class diagram](http://thebuildingcoder.typepad.com/blog/2012/01/no-revit-api-class-diagram.html) is
provided any longer, adding some background information, suggestions on what tools and methods to use instead, and how you might be able to create one for yourself, if you really want to.

For completeness' sake, it also includes the Revit 2010 API class diagram.

If you do decide to create an up-to-date one yourself, please let us know. Thank you!

#### Publish to SVG

Here is a final little question to wrap up this week:

**Question:** Is there any Revit API to publish the Revit model to an SVG file? Or is there any workaround to achieve that in Revit?

**Answer:** I am not aware of any built-in Revit API method to publish to SVG.

Your first question before attacking a Revit programming task should always be: Does the user interface support this in any way?

If not, then the API probably does not do so either.

However, I use SVG extensively and exclusively myself in my
[cloud-based round-trip 2D Revit model editing project](http://thebuildingcoder.typepad.com/blog/2013/04/room-editor-project-overview-and-couchdb-configuration.html).

It is an open format, so you can read the relevant Revit geometry and generate the SVG file yourself.

I also discussed it repeatedly before diving into that project.
In fact, The Building Coder has an entire category dedicated to the topic of
[SVG](http://thebuildingcoder.typepad.com/blog/svg).

The most up-to-date version of the RoomEditorApp for Revit 2013 that implements the SVG export of a simplified 2D view of a Revit BIM is provided along with the
[Sydney Revit API training material](http://thebuildingcoder.typepad.com/blog/2013/07/sydney-revit-api-training-and-vacation.html#9).

That's it for this week, then.

I wish you a happy weekend!