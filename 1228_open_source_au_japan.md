---
ai_optimized: true
author: Jeremy Tammik
complexity_score: 3.4
content_type: qa
optimization_date: '2025-12-11T11:44:15.557605'
original_url: https://thebuildingcoder.typepad.com/blog/1228_open_source_au_japan.html
post_number: '1228'
reading_time_minutes: 9
series: general
slug: open_source_au_japan
source_file: 1228_open_source_au_japan.htm
tags:
- references
- revit-api
- rooms
- views
title: "Autodesk Open Source All Over \u2013 Germany and Japan"
word_count: 1843
---

### Autodesk Open Source All Over – Germany and Japan

Here are some notes on Autodesk Open Source involvement from AU Germany, the Japan hackathon, and two other nice topics for today:

- [Autodesk University Germany](#2)
- [Japanese View and Data API hackathon projects](#3)
- [CAD Term Translation](#4)
- [Cool presentation gimmick](#5)

#### Autodesk University Germany

In the keynote session, Carl Bass introduced [Spark](http://spark.autodesk.com), Autodesk's new 3D printer and total commitment to open source:

- **Connected** – Spark connects digital information to 3D printers in a new and streamlined way, making it easier to visualize and optimize prints without trial and error, while broadening the range of materials used for printing.
- **Open** – Because the Spark platform is open, everyone – from hardware manufacturers, to app developers, to product designers – can use its building blocks to further push the limits of 3D printing and drive fresh innovation.
- **Free** – To encourage any and all members of the 3D printing industry to not only participate in Spark, but also to fuel it forward, Spark will be free to license. Together, we can accelerate the new industrial revolution.

All plans and details of the mechanical construction, electronics, materials, everything about the printer is open source and can be freely reused by anybody.

You can buy the Autodesk printer or use the freely available plans to build every single piece of it yourself.

Obviously, this also enables the entire open source community to participate and improve these plans.

Special invitation to
[students, universities and research institutes](http://www.autodesk.com/education) of all kinds.

Where is the development heading? Commercial versus open source?

Another interesting indication of Autodesk's involvement in open source can be had by taking a look at the
[threejs.org](http://threejs.org) open source project web site, where Autodesk figures quite prominently:

![Autodesk on threejs.org](img/threejsorg.png)

The third from the left in the top line is [autodesk360.com](http://autodesk360.com),
and the
[Udacity](http://www.udacity.com/overview/Course/cs291) entry
in the bottom left is from Autodesk as well:

Eric Haines, Instructor, is a Senior Principal Engineer at Autodesk, Inc., working on a next-generation interactive rendering system for computer-aided design applications. He is a coauthor of the book “Real-Time Rendering”, a founder and editor of the Journal of Computer Graphics Techniques, and maintainer of the Graphics Gems code repository, among other activities. He received an MS from the Program of Computer Graphics at Cornell in 1985.

Very cool, isn't it?

Returning to the AU Germany keynotes, Prof. Dr. Günther Schuh talked about the future of factories.

Be aware of the fourth industrial revolution:

1. Power – steam engine etc.
2. Workflow – Taylor, Ford, etc.
3. IT – computation
4. Collaboration productivity, evolving from human/human > human/machine > machine/machine

Computers and machines are starting to assist the design process in significant ways.
Where will automated machine ***collaboration*** lead?

My session on the
[Autodesk View and Data API](http://thebuildingcoder.typepad.com/blog/about-the-author.html#5.46) went
very well and was great fun, with standing room only:

![ADVA session with standing audience](file:////j/photo/jeremy/2014/2014-10-23_darmstadt_au/1_jeremy_session_standing_audience.jpg)

I already published the
[ADVA session slides and notes](http://thebuildingcoder.typepad.com/blog/2014/10/autodesk-view-and-data-api-notes-and-samples.html).

#### Japanese View and Data API Hackathon Projects

Here is a report by my colleague
[Toshiaki Isezaki](http://adndevblog.typepad.com/technology_perspective/toshiaki-isezaki.html) on
the four hackathon projects that show what can be achieved in a weekend using the
[Autodesk View and Data API](http://thebuildingcoder.typepad.com/blog/about-the-author.html#5.46) and
other open source components.

Toshiaki already published this overview on the
[Japanese DevBlog](http://adndevblog.typepad.com/technology_perspective) in his report on
[Autodesk 3D Hackathon – Day2](http://adndevblog.typepad.com/technology_perspective/2014/10/autodesk-3d-hackathon-results.html).

There were four teams at the hackathon:

- [Castle 360](#3.1)
- [Okamoto](#3.2)
- [Hackshion](#3.3)
- [Use of ReCap](#3.4)

![Japan hackathon teams](img/japan_hackathons.jpeg)

Here is an overview of their projects and results with a short video on each:

#### Castle 360 Team

Ogasawara-san joined to this team.

The concept of web app they created is to provide user experience with much presence. They suppose there are many cameras in soccer stadium including wearable camera putting on soccer players. This app has views from each camera at the right side of screen. When user chooses a single view among them that displayed on the main screen. The main screen is created by the Autodesk viewer LVM and it actually shows movie. In order to display movie on LMV, Ogasawara-san put a planar surface in the space. And he applies movie to planar surface as a material using Three.js. This is unique technique. This app also has '3D Capture' button to capture pictures from all cameras at a time. The captured pictures are processed by ReCap Photo API to generate 3D model. As a result, customer who uses this web app can feel presence with 3D model even if he is in his house. They couldn't implement ReCap Photo API portion actually due to timeout.

You can watch the short [demo video](https://www.youtube.com/watch?v=STSRbVrpbXY):

#### Team Okamoto

Saitou-san and Takamizawasa-san joined this team.

Japan is the society that is concerned with its decreasing birth rate and aging population. The created web app reflects it. They suppose an administrator's work for villas in resort area. He has to manage many villas alone. He has a convenient web to control facilities in each villa. He can select a villa he wants to manage on Google Map and can show 3D state of villa in LMV. 3D model has switches to turn on and off within LMV for example. He can turn off a light with it.

One of team member has designed lot of apartments using Revit, and he knows that BIM world also has lot of file formats. So he thinks LMV is best solution to show 3D models came from various file format as a front end interface for Home Automation. Home Automation is a recent trend on housing industry. Unfortunately, it seemed they cannot find a way to change light state in current LMV API.

You can watch the short [demo video](https://www.youtube.com/watch?v=oM1n1BcRS90):

#### Hackshion Team

This team was organized with interesting members. One person came from 3D printer service bureau. Other one is the president of start-up company. He has development skills for mobile but his company is to create and sell custom suit. The problem he is having is that it is difficult to sell suits on e-commerce system, because customer doesn't know whether jackets and pants that have been sold on internet will be fit to his/her body. So this team intended to develop a mobile app which uses ReCap Photo to measure sizes around human body by simply taking pictures. However, ReCap Photo API has not exposed taking survey point features, and it is sometimes slow response depending on internet connection when they operate it interactively. So they cancelled development work for that portion and tried to use Memento instead. As a result, app doesn't work along to single story. But business model to resolve current problem is clear comparing to other teams. Also this team intended to deliver API for other apparel industry that has an e-commerce system.

You can watch the short [demo video](https://www.youtube.com/watch?v=GYhjYaMGVBg):

#### Use of ReCap Team

Original number of this team was 3. But one person could not participate in Day 2. So that they could not complete their entire project. Their idea is to realize food samples (fake foods) that are shown at almost restaurant in Japan. If you don't know about it, refer to
[fake food](http://en.rocketnews24.com/2013/08/26/japan-and-the-art-of-fake-food).

Second idea this team brings is to share 3D model captured from children to grand father/mother in far place using social media such as Facebook. Both ideas are considered LMV and ReCap Photo API. But they are only two. So they connected our github samples to realize that story, using the workflow-wpf-view.and.data.api-master and Autodesk-ReCap-Samples-master projects.

You can watch the short [demo video](https://www.youtube.com/watch?v=RZiXoxFh58Q):

#### CAD Terminology Translation

Talking about all this international stuff raises the question CAD terminology translation:

**Question:** I am working on translations for my apps but running into difficulties when it comes to technical CAD terms in other languages. These are important for writing the app descriptions. Are there any resources available that list the proper translations for CAD Lingo? For example, what is a 'spline-fit vertex' in Japanese?

I came across this
[AutoCAD command dictionary](http://www.cadforum.cz/cadforum_en/command.asp),
but it only lists AutoCAD command translations and does not support all languages.

**Answer:** The Autodesk localisation team uses a cross product corpus database [NeXLT](http://langtech.autodesk.com/nexlt) for terminology and message translation.

This link is accessible from outside the company and translation companies working with the localisation team around the world make use it, so let's assume this resource is available to anyone working on translating products for Autodesk platforms.

Another option is to reference the CAD platform online help. For instance, AutoCAD provides an end user online document per language. You can read and analyse the various language versions of this document through these links:

- [English](http://help.autodesk.com/view/ACD/2015/ENU/)
- [Japanese](http://help.autodesk.com/view/ACD/2015/JPN/)
- [French](http://help.autodesk.com/view/ACD/2015/FRA/)
- [Korean](http://help.autodesk.com/view/ACD/2015/KOR/)
- [Russian](http://help.autodesk.com/view/ACD/2015/RUS/)
- ...

You can obviously also use the language packs, e.g. the
[AutoCAD 2015 language packs](http://knowledge.autodesk.com/support/autocad/downloads/caas/downloads/content/autocad-2015-language-packs.html).

#### Cool Presentation Gimmick

My son Christopher, composer and sound professional, pointed out this short video demonstrating a funny and enticing presentation gimmick that I thought you might enjoy as well, by Sam Tarakajian, having the happiest day of his life, presenting an update to the Max visual programming of music, sound, video, and interactive media applications.

Max supports audio and video manipulation, similar to
[Dynamo](http://dynamobim.org) for
3D modelling, extending building information modelling with the data and logic environment of a graphical algorithm editor.

Max does the same for audio and video.
You arrange boxes on a canvas and connect them together to create, experiment, and play.

One new feature is crash recovery.
How to best demonstrate that?
Simulate a crash, and bring in ***emotions***.

So, enjoy this 3.5-minute video on
[Max 7 Sneak Preview Episode 3](http://cycling74.com/2014/10/03/max-7-sneak-preview-episode-3):