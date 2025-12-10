---
post_number: "1495"
title: "The Building Coder"
slug: "devdays"
author: "Jeremy Tammik"
tags: ['filtering', 'geometry', 'references', 'revit-api', 'rooms', 'sheets', 'views']
source_file: "1495_devdays.md"
original_url: "https://thebuildingcoder.typepad.com/blog/1495_devdays.html"
---

### BIM@TuDa, DevDays, Forge News and More Events
I am in Darmstadt preparing
the [Forge and BIM](http://www.bim.tu-darmstadt.de) presentation
and hands-on workshop
at [Technische Universität Darmstadt](http://www.tu-darmstadt.de),
[Institut für Numerische Methoden und Informatik im Bauwesen](http://www.iib.tu-darmstadt.de),
the institute for numerical methods and computer science in the construction industry at the technical university here.
Many other larger events are coming up after this:
- [BIM@TuDa agenda](#2)
- [Getting started with Dynamo](#3)
- [Forge news](#4)
- [New Forge resources](#5)
- [Forge events and community](#6)
- [DevDays – Developer Day conferences and accelerators](#7)
#### BIM@TuDa Agenda
The agenda here consists of two parts, a 90-minute presentation and a 4-hour hands-on workshop:
- Presentation – [Connecting desktop and cloud](http://thebuildingcoder.typepad.com/blog/2016/10/connecting-desktop-and-cloud-at-rtc-material.html)
- Workshop
- Revit API – [creating a Revit add-in with one single click](http://thebuildingcoder.typepad.com/blog/2016/10/revit-api-and-connecting-desktop-and-cloud-tuda.html#4)
- Connecting BIM with the cloud – [connecting Revit and Forge with 25 lines of code](http://thebuildingcoder.typepad.com/blog/2016/11/roomedit3dv3-diff-from-boilerplate-code.html#9)
Since some participants are probably not well versed in the Autodesk biosphere, we'll try to cram in some supplementary information, resulting in the following agenda:
- Presentation
- Do you know Revit?
- Do you know Forge? Quick Forge intro
- Getting started with – theory
- Revit API: add-in, macros, dynamo
- Forge: WebGL, [developer.autodesk](https://developer.autodesk.com), [tutorial](https://developer.autodesk.com/en/docs/viewer/v2/tutorials/basic-viewer/)
- Connecting desktop and cloud, Forge and BIM
- Workshop
- Getting started with – practice
- [Revit API add-ins](http://thebuildingcoder.typepad.com/blog/2016/10/revit-api-and-connecting-desktop-and-cloud-tuda.html#4)
- [Forge viewer tutorial](https://developer.autodesk.com/en/docs/viewer/v2/tutorials/basic-viewer/)
- [Adapting boilerplate code to connect Forge and BIM](http://thebuildingcoder.typepad.com/blog/2016/11/roomedit3dv3-diff-from-boilerplate-code.html)
Some last-minute coordination on this with Philipp Mueller is still outstanding...
![Technische Universität Darmstadt](img/logo_tuda_150x309.png)
#### Getting Started with Dynamo
Since I am not a Dynamo expert myself, and other people recently also asked me how to get started with it efficiently, here are some more or less random notes on that topic, in case that is of interest to our participants:
- **A**dam Sheather recommends
the [Dynamo samples in the DynamoDS repo](https://github.com/DynamoDS/Dynamo) as a great start.
- [**B**IMThoughts](http://bimthoughts.com) asks: Want to Learn Dynamo? Check
out [DynamoThoughts on YouTube](https://www.youtube.com/c/dynamothoughts) with Ian Siegel!
- [**D**ynamo primer](http://dynamoprimer.com/en/)
Before looking forward to further upcoming events and learning opportunities, a quick look at the breaking news on Forge.
#### Forge News
- The grace period
for [token scope enforcement](https://developer.autodesk.com/en/docs/oauth/v2/overview/scopes) will
expire for grandfathered-in apps on January 2, 2017. Please make sure to update your apps accordingly.
- A number of improvements to the viewer, including an
updated [getting started tutorial](https://developer.autodesk.com/en/docs/viewer/v2/tutorials/basic-viewer/),
a new [InViewerSearch extension](https://developer.autodesk.com/en/docs/viewer/v2/tutorials/in-viewer-search-ext/),
and [version 2.11](https://developer.autodesk.com/en/docs/viewer/v2/overview/changelog/2.11/):
- `viewer.getProperties()` now returns `attributeName`, which can be used as a filter for search geometry snapping for mark-ups.
- more efficient `viewer.restoreState()`
- The Data Management API will soon be introducing a new `hubs:autodesk.a360:PersonalHub` type that will allow distinguishing Team Hubs from Personal Hubs. More details about the update and any needed changes to your code will be described
in [Recent Changes](https://developer.autodesk.com/en/docs/data/v2/overview/changelog/),
and the full documentation will be on
the [GET hubs endpoint](https://developer.autodesk.com/en/docs/data/v2/reference/http/hubs-GET/) reference page.
- The 3D Print API will be retired on January 15, 2017, when all existing keys will lose access to this API. Read more in the [end of life notice](https://developer.autodesk.com/en/docs/print/v1/overview/end-of-life/). New APIs will be announced in the near future!
#### Forge Resources
[Recordings of the six recent technical webinars on Forge APIs](http://adndevblog.typepad.com/cloud_and_mobile/2016/10/new-training-on-autodesk-forge-apis-five-webinars-now-available-for-viewing.html) are now available, covering workflows, platform use cases, and code samples.
#### Forge Events and Community
Loads of events are coming up! Check out the [complete list of upcoming events](http://adndevblog.typepad.com/cloud_and_mobile/2016/10/upcoming-forge-related-events.html) with full descriptions. Here are the highlights:
- November 14 – [DevDays and DevLab](http://au.autodesk.com/las-vegas/pre-conference/adn-conference-devdays) – Las Vegas, Nevada
- November 15-17 – [Autodesk University](http://au.autodesk.com/) – Las Vegas, Nevada –
Join us at our [user study](https://autodeskuserresearch.doodle.com/poll/tbrmmpng3zevqnbr) while you're at AU!
- December 5-23 – [ADN DevDays](http://autodeskdevdays.com/) – Worldwide
Questions? Suggestions? Let us know at [@ForgePlatform](https://twitter.com/ForgePlatform) or check out our tips on finding answers on [StackOverflow](https://developer.autodesk.com/en/support/get-help).
#### DevDays – Developer Day Conferences and Accelerators
Partially covering Forge and also addressing the desktop products, the main traditional Autodesk developer events of year are coming: the annual, worldwide [Developer Day Conferences and Accelerator workshops](http://autodeskdevdays.com/).
Free of charge, they kick off in two weeks' time, starting in Las Vegas, Nevada, then moving on to Europe and Asia.
Attending DevDays is the best way for you to learn about the latest and upcoming Autodesk application development technologies. You can learn about the Autodesk Cloud Platform [Forge](https://forge.autodesk.com/), as well as network with Autodesk engineers and other developers working with Autodesk APIs.
Personally, I am attending the following sessions in USA and Europe:
- United States
- November 14, 2016 – DevDay at AU, Las Vegas
- November 15, 2016 – DevLab at AU, Las Vegas
- Germany
- December 5, 2016 – DevDay, Munich
- December 6-9, 2016 – Accelerator, Munich
If you have already registered to attend one of the events – congratulations, me and my colleagues look forward to seeing you soon!
If not, read on.
Registration for DevDays Las Vegas is via the [Autodesk University website](http://au.autodesk.com/las-vegas/pre-conference/adn-conference-devdays).
Register and choose your sessions here.
If you are attending another Monday conference at AU and therefore cannot register for DevDays but would like to drop in at one or more DevDay sessions, please let us know so that we can add you to a list for special access. Just send us an [email to adnreg@autodesk.com](mailto:adnreg@autodesk.com) with your name and company name and we’ll take care of it.
Registration for Asia and Europe is easy. Simply visit the event website at [www.autodeskdevdays.com](http://autodeskdevdays.com) for full information and click on the [Register button](http://autodeskdevdays.com/register).