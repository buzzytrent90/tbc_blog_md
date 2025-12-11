---
ai_optimized: true
author: Jeremy Tammik
complexity_score: 3.5
content_type: qa
optimization_date: '2025-12-11T11:44:16.754414'
original_url: https://thebuildingcoder.typepad.com/blog/1792_dynamo_infrastructure.html
post_number: '1792'
reading_time_minutes: 8
series: general
slug: dynamo_infrastructure
source_file: 1792_dynamo_infrastructure.md
tags:
- elements
- family
- geometry
- levels
- references
- revit-api
- sheets
- views
title: Dynamo Infrastructure
word_count: 1593
---

### DevCon Invitation and Dynamo for Infrastructure
Let's highlight another class from BILTNZ and a personal invitation to the upcoming DevCon in Las Vegas:
- [Invitation to Forge DevCon in Las Vegas](#2)
- [Visual Programming in Infrastructure](#3)
- [Class description](#3.1)
- [Table of contents](#3.2)
- [Placing an instance along an edge](#4)
- [Most popular programming languages 1965-2019](#5)
#### Invitation to Forge DevCon in Las Vegas
Before diving into the main topic for today, here is a personal invitation to the upcoming to Forge DevCon in Las Vegas from Jim Quanci, Senior Director, Forge Platform Development and Autodesk Developer Network:
On November 18th we'll be holding our 4th global [Forge DevCon](https://forge.autodesk.com/devcon-2019) in Las Vegas.
The Forge DevCon is a 'pre-conference' to Autodesk University.
But don't let the 'pre-conference' bit fool you.
Forge DevCon is a fully-fledged conference in its own right.
Of course, a lot of attendees will be staying on in Vegas all week to get the full AU experience, but Forge DevCon is well worth the trip on its own – and around 20% of our attendees do just that.
We're expecting over 1500 attendees just at Forge DevCon, and we're continuing our collaboration with the Connect & Construct Summit so that attendees at either conference can attend any class or keynote – that adds another 2500 attendees into the mix.
The theme for this year’s Forge DevCon is Digital Transformation.
Whether you are taking your business thru a Digital transformation – or you are helping your customers thru their Digital Transformation.
We've lined up 30 classes for you at Forge DevCon and have ensured there's something for everyone.
There will be deep technical classes for coders, Forge roadmap, case studies delivered by partners, business-oriented classes, panel sessions by industry leaders, “Meet the Experts” where the Autodesk expert panel will answer all of your questions, and even a series
of [Learning Lab classes](https://autodeskuniversity.smarteventscloud.com/connect/search.ww?searchType=session&searchPhrase=FDC320314+FDC320976+FDC322917+FDC331632#loadSearch-searchPhrase=FDC322542%20FDC322546%20FDC322554%20FDC323665%20FDC322552&searchType=session&tc=0&sortBy=dayTime&i(72466)=&p=) to
help those of you who are new to Forge to get up and running quickly.
What’s new to learn at the DevCon that shouldn’t be missed?
- Forge Design Automation with the release of Revit, Inventor and 3ds max “engines in the cloud” (along with AutoCAD that is already released). What are Autodesk partners and customers doing with design Automation? Automate. Automate. Automate. From automating design to automating model checking and more.
- Forge Model Derivative and Viewer using next generation “OTG” format. This is all about an order of magnitude increase in model size and performance. Work with your largest models in the browser at “high speed” and on hardware limited mobile devices (phone, tablet, AR/VR and more).
- New BIM 360 APIs. Model Coordination (clash), Cost, Submittals, and more.
The Forge team will be present at the Conference and at your disposal to answer any of your questions, up close and personal help with your challenges.
Face to face time is important when you are climbing the learning curve and driving change in your organization – so come and get face time with Autodesk decision makers and engineers – and share learnings with other developers just like you.
Convince your manager today that he needs to send you to the Forge DevCon – and
then [register for the conference now](https://autodeskuniversity.smarteventscloud.com/portal/myAccount.ww?pass=forgeDevCon).
It's only four weeks away!
If you already registered for AU, it’s easy to add your Forge DevCon pass for just $150 – go to
your [AU registration](https://autodeskuniversity.smarteventscloud.com/portal/myAccount.ww?pass=forgeDevCon) and
click “Add a pass”.
In case you have questions about your registration, our AU customer service is ready to help,
just [email autodeskuniversity@autodeskevents.com](mailto:autodeskuniversity@autodeskevents.com).
Questions (of any sort)? Unsure?
Want to have a 1 on 1 meeting while I am in Las Vegas?
Please do feel free to reach out to me directly.
Just [drop me an email](mailto:jim.quanci@autodesk.com) – or call/text me.
Really.
Cheers,

Jim Quanci

Senior Director, Software Partner Development, Autodesk, Inc.

+1-415-640-4461 (mobile)

![Jim Quanci, sailor](img/jim_quanci_sailor.png)
#### Visual Programming in Infrastructure: Gung-ho Tactics for Getting Stuff Done
Back to the Revit API and Dynamo, in this case:
Let's highlight another BILT 2019 class shared by
[Jostein Berger Olsen](https://www.linkedin.com/in/jostein-berger-olsen-97416741/),
of [Bad Monkeys](https://www.badmonkeys.net) Norway
– the AEC workflow changers,
not the [psychological thriller](https://en.wikipedia.org/wiki/Bad_Monkeys)
by [Matt Ruff](https://en.wikipedia.org/wiki/Matt_Ruff),
nor the [film currently under development](https://www.imdb.com/title/tt5901378) :-)
Says Jostein [@Jos_ols](https://twitter.com/jos_ols) on Twitter:
> My blog is dead close to being.. well, dead..
But anyways, here is the link to a blog post containing all my session materials for #BILTeur 2019: [bit.ly/32kHNSY](https://bit.ly/32kHNSY).
Hopefully it can be of some value. Cheers!
I copied Jostein's material here locally as well, to [jostein_olsen_bilt_2019.zip](zip/jostein_olsen_bilt_2019.zip).
Many thanks to Jostein for sharing his valuable experience with us!
#### Class Description
Session 2.2 at the Digital Built Week Europe 2019, BILT Europe 2019, EICC, Edinburgh, October 10-12, 2019:
Visual Programming in Infrastructure: Gung-ho Tactics for Getting Stuff Done
The introduction of BIM in Infrastructure has been, and still is, a bumpy ride. Adding to it, Infrastructure projects are complex and faceted in its very nature. With visual programming tools you as an engineer, modeller or drafter get a new set of tools that can be applied to a range of different production challenges for consultants, owners and contractors alike. This lab will look into how we can parametrically drive Infrastructure BIM-models using visual programming tools. We'll cover some basics, but also explore aspects of geometry, math and data handling that in sum can more or less directly be applied to live projects.
So, if you're an Infrastructure BIM'mer come attend this lab, and learn how things can be done better and faster than ever before!
#### Table of Contents
1. Introduction — A Little Note on Software – How Do We Learn? – What’s the Deal with These Curves – Curves ain’t curves
2. Points (Along curves) — Points along curves – Points Projected – Points Analysed
3. Vectors and Angles — Setting an Elements Rotation
4. Data — Renumber Elements – Parsing Text Data
5. Coordinate Systems — Placing Utility poles again – Building Solids using CS – CS and Discrete Geometry
6. Closing Remarks
![Visual programming in infrastructure](img/dynamo_infrastructure.png)
#### Placing an Instance Along an Edge
Among many other things, Jostein shows how to place family instances along curves.
Here is another discussion on that at a lower level, from the [Revit API discussion forum](http://forums.autodesk.com/t5/revit-api-forum/bd-p/160) thread
on [placing a family instance along a selected edge of another family instance](https://forums.autodesk.com/t5/revit-api-forum/place-a-family-instance-along-selected-another-family-instance/m-p/9096037),
once again showing the use of simple snooping to discover what is going on under the hood:
\*\*Question:\*\* I have two families in a project.
I bring in one family instance based on the selected model line.
I have an issue placing another family instance in the project based on a selected edge in the already placed family instance.
It is placed in opposite orientation (it seems rotated).
I used code for `GetInstanceEdgeFromSymbolRef` from [The Building Coder samples CmdDimensionInstanceOrigin module](https://github.com/jeremytammik/the_building_coder_samples/blob/master/BuildingCoder/BuildingCoder/CmdDimensionInstanceOrigin.cs) to select the reference edge for placing the second family instance.
Points of the selected edge are correct as per code in above link.
But the orientation of the selected geometry seems to be wrong.
Placing manually using family instance edge is working fine.
![Placing an instance along an edge](img/place_inst_along_edge_1.jpg)
I used RevitLookup to check, and it seems the orientation of the instances differ:
![Placing an instance along an edge](img/place_inst_along_edge_2.jpg)

(0,1,0) and (1,0,0)

![Placing an instance along an edge](img/place_inst_along_edge_3.jpg)

(0,-1,0) and (-1,0,0)

\*\*Answer:\*\* Here is the solution I found:

```
  Edge selEdge = RevitActions.Instance
    .GetInstanceEdgeFromSymbolRef(LeftSideEdge);

  Curve refcurve = selEdge.AsCurve().CreateReversed();
```

After extracting the curve, I reversed it.
I still don't understand why the family placement works fine while manually, whereas using the API I need to reverse it.
Many thanks to Pandian Srinivasan for sharing this!
#### Most Popular Programming Languages 1965-2019
I'll close with this very nice five-minute animation based on historical data on the relative popularity of programming languages in the past six decades,
the [most popular programming languages 1965-2019](https://youtu.be/Og847HVwRSI):
> Timeline of the most popular programming languages since 1965 to 2019.
So far, the most intense ranking I've ever done :)
For recent years, I've used multiple programming languages popularity indexes with adjustments thanks to the data from GitHub repositories access frequency.
For historical ranking, I've used aggregation of multiple national surveys to establish several data points, plus a world-wide publications rate of occurrence.
In this, ranking popularity is defined by percentage of programmers with either proficiency in specific language or currently learning/mastering one.
`Y` axis a relative value to define ranking popularity between all other items.
Have fun!