---
ai_optimized: true
author: Jeremy Tammik
complexity_score: 3.6
content_type: tutorial
optimization_date: '2025-12-11T11:44:16.800696'
original_url: https://thebuildingcoder.typepad.com/blog/1815_devdays_migrate.html
post_number: '1815'
reading_time_minutes: 5
series: general
slug: devdays_migrate
source_file: 1815_devdays_migrate.md
tags:
- family
- references
- revit-api
- sheets
- views
title: Devdays Migrate
word_count: 960
---

### DevDays Online and Add-In Migration
I share a contribution from fellow blogger Eric Boehlke and the announcement of the upcoming yearly DevDays Online presentations:
- [Add-in migration – update API references](#2)
- [Addendum – NuGet package and .NET framework version](#2.1)
- [Join us for our DevDays Online webinars](#3)
#### Add-In Migration – Update API references
Eric Boehlke of [truevis](https://truevis.com) BIM Consulting
wrote a blog post about how to upgrade a Revit API add-in to a version of Revit.
Says he:
> You may already have done this thousands of times.
One of the reasons to write these posts is to remember for myself how to do such things a year from now.
Hence:
[Upgrading Revit API Apps For Newer Revit Versions](http://revthat.com/upgrading-revit-api-apps-for-newer-revit-versions)
Here is a [75-second video](https://youtu.be/ypC_0REg22U) reiterating the same instructions to make a dry subject more fun:

The cautionary message on not confusing `RevitAPIUI.dll` and `RevitUIAPI.dll` is due to personally running into that issue once.
Removing the existing Revit API references, adding the new ones, and setting `Copy Local` to `false` is the foolproof way to upgrade.
In many cases, though, you can do this all in one single step by just adding the new references.
This overwrites the old references and retains the `Copy Local` `false` setting.
However, that is an unimportant detail.
It's not actually much effort to create such a video from a blog post.
[lumen5.com](https://lumen5.com) makes an initial "AI" attempt at turning my blog posts into videos.
Then I just refine it. It takes 10 or 15 minutes.
They say: "We automatically create Instant Videos for you based on your RSS feeds. "
Here's another one of Eric's videos,
on [how to get coordinates of an existing Revit View, then use them for placing other Views in Dynamo](https://youtu.be/UZl9gpFgxy0).
#### Addendum – NuGet Package and .NET Framework Version
Jason Masters adds some important notes to this in
his [comment below](https://thebuildingcoder.typepad.com/blog/2020/01/devdays-online-and-add-in-migration.html#comment-4774664575):
In terms of upgrading Revit API versions, I'd highly recommend switching from references to the SDK to referencing
the [NuGet package published by Matthew Taylor (Revit_All_Main_Versions_API_x64)](https://www.nuget.org/packages/Revit_All_Main_Versions_API_x64):
![NuGet package Revit all main versions API](img/revit_all_main_versions_api.jpg "NuGet package Revit all main versions API")
This package contains all the Revit API references across all versions.
Using NuGet means that any other developer opening your repo won't necessarily need the SDK to build it, it will integrate more easily into CI/CD pipelines, and checking against different API versions is as simple as changing which version of the NuGet package you're using.
Also, if you need to migrate to 2020, you're going to need to first change the .NET target framework version of your project to 4.7, then update your references to the 2020 API.
Many thanks to Jason for these important notes!
#### Join us for our DevDays Online Webinars
We welcome you to join us for our special series of webinars where we’ll be going over the current important development topics.
Some of them were already covered last year's DevCon events. These webinars are a great opportunity for you to learn about Autodesk Forge and where Autodesk is taking the desktop platforms in the coming year.
Click on the links below to register for the webinar(s) of your choice.
All webinars start at 8am PST (4pm GMT, 5pm CET, 11am EST).
If the session timing is inconvenient for you to attend (which is true for most of our partners in Asia), you can rest assured we will be recording all the sessions and will post them on the web for your later viewing.
Registration is open to all except where noted below:
- **Tuesday 2020-02-25 – DevDays Keynotes**
– Jim Quanci, Senior Director for Software Partner Development, kicks off our DevDays online webinar series with the latest news for desktop developers (get ready for the 2021 releases this Spring), Forge roadmap that includes the Forge Design Automation CAD Engines on the cloud (AutoCAD, Revit, Inventor and 3ds Max) and an overview of the enhanced BIM 360 APIs for the Autodesk Construction Cloud.

[Register](https://autodesk.zoom.us/webinar/register/WN_J-iJ9Iy1TQ-TYgB3CdQoLg)
- **Wednesday 2020-02-26 – Forge API Update**
– Augusto Goncalves will discuss in detail the updated and new APIs added to the Forge platform during the last year.

[Register](https://autodesk.zoom.us/webinar/register/WN_MlyzAqW8TF-oC7XPFcC7FA)
- **Thursday 2020-02-27 – Revit, Civil 3D and InfraWorks API Updates** (for ADN members only)
– Join Sasha Crotty and Augusto Goncalves to discover the product and API changes and enhancements coming in the next releases of Revit, Civil 3D and InfraWorks.

[Register](https://autodesk.zoom.us/webinar/register/WN_jLl0gXjxTnK3PWHsGyzARg)
- **Tuesday 2020-03-03 – Inventor, Vault and Fusion API Update** (for ADN members only)
– We will review upcoming changes in the next release of Inventor as well as recent updates in Vault and Fusion 360 API.

[Register](https://autodesk.zoom.us/webinar/register/WN_XlRo7ADySLGofmc7M9cdkQ)
- **Wednesday 2020-03-04 – BIM 360 API Update**
– Learn about new APIs in the BIM 360 family products.
We’ll talk about new Model Coordination and Cost Management APIs and other API enhancements.

[Register](https://autodesk.zoom.us/webinar/register/WN_TIxv3ZpPS1228DYy_-i7HA)
- **Thursday 2020-03-05 – The next Release of AutoCAD APIs** (for ADN members only)
– Discover API changes coming in the upcoming release of AutoCAD Rogue.

[Register](https://autodesk.zoom.us/webinar/register/WN_h1Mmc-leRjKAFhqIjiv9Sw)
After registering, you will receive a confirmation email containing information about joining the webinars.
![DevDays Online 2020](img/devdays_online_2020.jpg "DevDays Online 2020")