---
post_number: "1885"
title: "Aec Forge Accel"
slug: "aec_forge_accel"
author: "Jeremy Tammik"
tags: ['csharp', 'levels', 'revit-api', 'schedules', 'sheets', 'views']
source_file: "1885_aec_forge_accel.md"
original_url: "https://thebuildingcoder.typepad.com/blog/1885_aec_forge_accel.html"
---

### I18n, Forge Accelerators, AEC and BIM360 Focus
An especially interesting Forge accelerator is coming up, and how to handle Revit add-in language resources:
- [Upcoming Forge accelerators](#2)
- [AEC focused Forge accelerator Waldspirale](#3)
- [Internationalisation using .NET language resources](#4)
#### Upcoming Forge Accelerators
We currently have two [upcoming Forge accelerators](https://forge.autodesk.com/accelerator-program) scheduled:
- Machu Picchu, Virtual Accelerator, January 25-29, 2021 – [apply now](https://www.eventbrite.com/e/autodesk-virtual-forge-accelerator-machu-picchu-january-25-29-2021-registration-131468575047)
- Waldspirale, Virtual Accelerator, February 22-26, 2021 – [apply now](https://www.eventbrite.com/e/acc-focused-autodesk-virtual-accelerator-waldspirale-feb-22-26-2021-registration-131597280007)
Joining a Forge accelerator is the most effective way to get up to speed on Forge in general and also
to [answer any and all BIM360 related questions](https://thebuildingcoder.typepad.com/blog/2020/04/2021-migration-add-in-language-and-bim360-login.html#2) you
may be pondering.
As a knowledgeable and architecturally interested person might guess, the February event is AEC oriented.
#### AEC Focused Forge Accelerator Waldspirale
An AEC focused online Forge accelerator is coming up February 22-26.
It is named "Waldspirale" in honour of
the Austrian artist, environmentalist and architect [Friedensreich Hundertwasser](https://en.wikipedia.org/wiki/Friedensreich_Hundertwasser) and
his [105-apartment building of the same name](https://en.wikipedia.org/wiki/Waldspirale) in Darmstadt, Germany.
We are hence looking for proposals that aim to solve problems in design, construction and building life cycle management.
We very much look forward to discussing and supporting your many creative ideas and helping you to jump start your development with Forge during the week-long event.
In addition to the APIs that are already publicly available, we are also expecting to have some new sets of BIM 360 APIs to play with by then [(\*1)](#3.1):
- Assets API
- Data Connector API
If you are interested in using these new APIs, this event will be a great opportunity to learn and develop while "sitting" with development teams.
If you would like to know what will be possible with the Assets feature and its API, please check out the recordings of the recent Autodesk University presentations:
- [Tracking Assets and Equipment in BIM 360](https://www.autodesk.com/autodesk-university/class/Tracking-Assets-and-Equipment-BIM-360-2020)
- [BIM 360 API Update and Beyond](https://www.autodesk.com/autodesk-university/class/BIM-360-API-Update-and-Beyond-2020)
– the Assets API overview starts at 16:15
In addition, the [Data Connector](https://help.autodesk.com/view/BIM360D/ENU/?guid=BIM360D_Insight_data_extractor_html) feature enables you to extract data at an account level with Executive privilege.
This API will allow you to schedule extraction jobs and access extracted data.
In case of interest, here are
more [full articles and video recordings of the Forge classes at AU classes](https://thebuildingcoder.typepad.com/blog/2021/01/forge-at-au-and-open-source-property-access.html#3).
Please submit your proposal
to [forge.autodesk.com/accelerator-program](https://forge.autodesk.com/accelerator-program).
If you would like to find out more about the new APIs in order to detail your proposal, feel free to indicate so in your proposal.
We will get back to you and discuss further in your specific context.
(\*1) Disclaimer: until we actually see the API officially released, the public release date might change.
![AEC focused Forge accelerator Waldspirale](img/2021-02_forge_accelerator_waldspirale.jpg "AEC focused Forge accelerator Waldspirale")
#### Internationalisation Using .NET Language Resources
Before closing for today, let's quickly also address a desktop Revit API issue.
[Lukáš Kohout](https://forums.autodesk.com/t5/user/viewprofilepage/user-id/767846), Czech Architect and Revit Developer, very kindly solved
a [Revit API discussion forum](http://forums.autodesk.com/t5/revit-api-forum/bd-p/160) thread
about internationalisation or I18n
on [how to convert a Revit plugin from English to other languages](https://forums.autodesk.com/t5/revit-api-forum/how-to-convert-a-revit-plugin-from-english-to-other-languages/m-p/10008918):
\*\*Question:\*\* I have been developing Revit plugins for the past two years in English language.
Now, I intend to convert them into other languages to make them useful for people all around the world.
I would appreciate if anybody could help me with the procedure for doing so.
\*\*Answer:\*\* You can use Resources with string tables as described in the StockOverflow discussion
on [how to use localisation in C#](https://stackoverflow.com/questions/1142802/how-to-use-localization-in-c-sharp).
Many thanks to Lukáš for the very helpful pointer.