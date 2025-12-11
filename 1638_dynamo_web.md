---
ai_optimized: true
author: Jeremy Tammik
complexity_score: 2.2
content_type: qa
optimization_date: '2025-12-11T11:44:16.439777'
original_url: https://thebuildingcoder.typepad.com/blog/1638_dynamo_web.html
post_number: '1638'
reading_time_minutes: 5
series: general
slug: dynamo_web
source_file: 1638_dynamo_web.md
tags:
- parameters
- revit-api
- sheets
- views
title: Dynamo Web
word_count: 914
---

### Dynamo on the Web?
Do you Dynamo?
Do you have a potential application for [Autodesk Dynamo on the Cloud](#3)?
If yes, we want to talk to you.
First, however, I'll mention that I just returned from my nicest and longest ski tour so far this year, four days, in Austria, for a change: the famous Venter Runde in
the [Ötztal Alps](https://en.wikipedia.org/wiki/%C3%96tztal_Alps).
We visited the place
that the Hauslabjoch mummy [Ötzi](https://en.wikipedia.org/wiki/%C3%96tzi) was
found and climbed the [Finailspitze (3514 m)](https://en.wikipedia.org/wiki/Fineilspitze),
[Weißkugel (3739 m)](https://en.wikipedia.org/wiki/Wei%C3%9Fkugel) and
Fluchtkogel (3500 m) in splendid conditions
([high resolution](http://thebuildingcoder.typepad.com/img/588_finailspitze_jeremy.jpg) [^](/p/2018/2018-03-24_skitour_venter_runde/thomas/588_finailspitze_jeremy.jpg)):
![Jeremy on the Finailspitze summit in Austria](img/588_finailspitze_jeremy.jpg)
#### Do You want Dynamo on the Web?
Autodesk is considering making [Dynamo](https://www.autodesk.com/products/dynamo-studio/overview)
– Autodesk’s Computational Design App and Engine – available as a Forge API.
We want to talk to customers and partners about how they might want to leverage Dynamo on the Cloud. The opportunity is to take your Dynamo projects and make them available to a much wider audience – across the company, to business partners, or even as a commercial app – through a tailored web page (or desktop or mobile app) you create.
We are considering three possible implementations of Dynamo on the Cloud, and would like to know which of them most interests you, why, and how you envision using it:
1. [Dynamo as part of Forge Design Automation for Revit](#3.1)
2. [Headless Dynamo](#3.2)
3. [Full Dynamo on the Web](#3.3)
#### 1. Dynamo as part of Forge Design Automation for Revit
As you may have heard already, Autodesk is well down the path
developing [Forge Design Automation for Revit](http://au.autodesk.com/au-online/classes-on-demand/class-catalog/classes/year-2017/autocad/sd124720#chapter=0),
aka [Revit I/O](http://thebuildingcoder.typepad.com/blog/about-the-author.html#5.28b).
This is Revit as an 'engine' on the cloud. No user interface, aka 'headless' – an engine you can drive using the Revit API we have today. We are currently working on moving Forge Design Automation for Revit from private to public beta.
Would you like to be able to leverage Dynamo too? No user interface means you would be programmatically loading a Revit app and your Dynamo project into a Revit instance on the cloud, pushing parameters and getting results – with your own handling of all user interaction (and maybe even server to server?).
#### 2. Headless Dynamo
Dynamo as a standalone 'headless' engine on the cloud to orchestrate between services.
Dynamo is architected to not just drive Revit, but to power computational design processes that span any number of design tools and web services. Do you have an application for Dynamo as a computational design engine independent of Revit? Again, this is a 'no UI' Dynamo web service – you would be responsible for creating and delivering the UI.
The Dynamo team has been building a new computational system that is cloud-native and completely independent from desktop applications. Do you have a set of Forge services you would like to tie together but are not sure how? You can use this to connect to existing Forge services and orchestrate how data flows between your application and services. You can use it as a way to extract logic and features from your desktop applications and re-use them as microservices in the cloud.
#### 3. Full Dynamo on the Web
Dynamo on the Web – with full user interface as Dynamo Studio users experience today.
You have used Dynamo Studio along with Revit. Do you have a problem that would be best solved by including Dynamo Studio (as a web service) inside your application of choice? This would likely enable you to embed the complete Dynamo Studio UI in a web page, desktop or mobile app of your making.
#### Your Choice
Over the last few years Autodesk has invested time replumbing Dynamo to make all three of these Dynamo on the Cloud implementations a real possibility. Now we need to hear which (if any) of these Dynamo on the Cloud you want and why. Based on your input, both what you want to create and how many Autodesk customers and partners are also interested in these various Dynamo on the Cloud implementations, we’ll decide on how to best move forward delivering Dynamo as a Forge Computational Design API.
Please don’t hold back. We need to hear from you. If we don’t hear enough interest from the Dynamo and Forge development community, we might just leave Dynamo on the Cloud on the back shelf for another day.
How to let us know what you want? Just send an email
to [jim.quanci@autodesk.com](mailto:jim.quanci@autodesk.com) to
participate in Dynamo as a Service research and discussion, and let him know:
- Which of the three above potential implementations you are interested in
- Why
- A few sentences of about your 'vision' of the user experience/app you would like to create
- The company you work for
- Your depth of experience with Dynamo today (beginner, advanced, 'guru')
- Your telephone number, so we can talk live if we want to explore and better understand your potential application of Dynamo on the Cloud
Thank you!