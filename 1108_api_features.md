---
ai_optimized: true
author: Jeremy Tammik
complexity_score: 5.8
content_type: qa
optimization_date: '2025-12-11T11:44:15.320035'
original_url: https://thebuildingcoder.typepad.com/blog/1108_api_features.html
post_number: '1108'
reading_time_minutes: 9
series: general
slug: api_features
source_file: 1108_api_features.htm
tags:
- doors
- family
- geometry
- revit-api
- walls
- windows
title: More on Revit API Aspects and Features
word_count: 1759
---

### More on Revit API Aspects and Features

Yesterday, I mentioned a couple of
[Revit API aspects and features](http://thebuildingcoder.typepad.com/blog/2014/02/different-revit-api-aspects-and-features.html) that
triggered follow-up comments.

Seeing as the topic appears to be of general interest, let's pick them up and continue this.

I'll add a few other little titbits first, as well, though, related to meat, BIM, and the Revit API, in that order:

- [Help escape the Meatrix](#2)
- [Autodesk BIM interoperability](#3) landing page
- [BIM acronyms and dictionary](#4)
- [Creating non-rectangular openings in walls](#5)
- [Reactions on the Revit API abomination](#6)

#### Help Escape the Meatrix

As you might know, I am vegetarian, and have been so for well over 30 years now.

You are probably also at least a little bit familiar with the well-known science fiction adventure film trilogy on
[The Matrix](http://en.wikipedia.org/wiki/The_Matrix) by
[the Wachowski brothers](http://en.wikipedia.org/wiki/The_Wachowskis).

Regardless, you will probably enjoy this funny and well-made four-minute appeal to
[help escape the Meatrix](http://www.youtube.com/rEkc70ztOrc):

#### Autodesk BIM Interoperability Landing Page

The Autodesk web site recently unveiled a new landing page addressing
[BIM interoperability](http://www.autodesk.com/campaigns/interoperability).

It deals mainly with IFC, plus some items related to gbXML, STL, COBie etc.

#### BIM Acronyms and Dictionary

Talking about BIM, a scarily large number of related terms and acronyms have evolved.

A large help in getting a grip on them is provided by
[Bond Bryan Architects](http://www.bondbryan.com) on their
[BIM blog](http://bimblog.bondbryan.com/document):

- [BIM acronyms](http://bimblog.bondbryan.com/wp-content/uploads/2013/11/30001-BIM-acronyms.pdf)
- [BIM dictionary](http://bimblog.bondbryan.com/wp-content/uploads/2014/01/30004-BIM-dictionary.pdf)

#### Creating Non-rectangular Openings in Walls

OK, enough non-API related stuff for now.

Let's get back to the real thing again.

I'll simply share the following hint from my colleague Joe Ye in the Revit API discussion thread on
[creating a non-rectangular opening in a wall](http://forums.autodesk.com/t5/Revit-API/Wall-Opening/td-p/4640815):

One way to create a non-rectangular opening in a wall is to define a window family with no window frame.
It can cut a hole in the wall.
You can define any shape whatsoever for the window.
This enables you to easily cut arbitrarily shaped openings in walls.
The 'window' instance can be placed by calling the NewFamilyInstance method.

One example showing how to create such a window family is provided by the 'Window-Round Opening.rfa' sample family in the Revit default family library.

#### Reactions on the Revit API Abomination

Let us return to the interesting and informative discussion on whether or not
[the Revit API is an abomination](http://thebuildingcoder.typepad.com/blog/2014/02/different-revit-api-aspects-and-features.html#2).

It was initially triggered by Adrian Hodos'
[comment](http://thebuildingcoder.typepad.com/blog/2014/01/no-inheritance-and-no-strong-naming.html?cid=6a00e553e16897883301a73d77a325970d#comment-6a00e553e16897883301a73d77a325970d) and
Arnošt Löbel's
[answer](http://thebuildingcoder.typepad.com/blog/2014/01/no-inheritance-and-no-strong-naming.html?cid=6a00e553e16897883301a73d77a325970d#comment-6a00e553e16897883301a73d77a325970d) on
the lack of
[inheritance and strongly named .NET assemblies](http://thebuildingcoder.typepad.com/blog/2014/01/no-inheritance-and-no-strong-naming.html).

Here are a couple of follow-up responses by them both and Guy Robinson, another extremely competent and experienced independent Revit API application developer, who, unlike many others, has conducted significant benchmarking experiments to optimise his API handling of large BIM projects:

- [Adrian](#6.1)
- [Arnošt](#6.2)
- [Jeremy](#6.3)
- [Guy](#6.4)

**Adrian** Well, it's nice to see that my (perhaps too strongly worded) feedback has reached someone. Now, regarding the managed only API access. You said that "In my personal opinion, the .NET environment makes the API also more approachable to users that aren't professional developers". It is good that you are trying to broaden the possible developer base of your product. But I don't see why you are forcing every developer out there to take the managed path. Why can't you provide a native API too? (If I'm not mistaken, this is exactly what Maya and AutoCAD do). Another problem is the functionality (or lack of functionality) in the Revit API. Its 2014 and I still can't rotate/translate/scale a solid :) Come on people, you can do better than that (just look at Maya's API – its great)! As pointed out by others, the lack of backward compatibility is also a major source of pain.

I really hope to see some improvements in the future Revit APIs.

Have a nice day.

**Arnošt** Adrian, it would be a long-ish discussion if I’d tried to answer all your inquiries. Not that I wouldn’t like to; not at all (ask Jeremy, right Jeremy? :-), but there is simply too many aspects of the API we normally do not mention anywhere, or not very often a perhaps clearly enough. I feel that this would be a very good discussion to have at, say AU or DevCamp, or some similar event. I will still try to fill at least some of the blanks here, however.

You have mentioned, Adrian, the differences between Revit and other applications, namely AutoCAD and Maya. And I confirmed that, indeed, there are differences. Not only those applications were written for different purpose and audiences, they were also developed with different philosophies (and by different companies, let’s not forget). While AutoCAD virtually stands and falls with its API (and really, what would AutoCAD be without its applications?), Revit, on the other hand was planned not to ever need an API. Yes, you read that correctly. In Revit, the API was an afterthought only introduced years after Revit 1.0. With that original concept being deeply embedded in Revit it is not easy and sometimes even not appropriate to expose the internal layer directly. The core components have simply not been written to be exposed. That is why we decided to introduce a proper API layer and at that time it sounded like most reasonable to have it as a .NET API, since there is where most of Revit-tweaking customers come from.

As for the speed advantage of native programming, you do not have to convince me. I am a native programmer too, after all. However, I do not see the speed being the problem for most external applications. At least that is not the issue we hear the most complaints about. I already mentioned our Cloud Rendering add-on in my previous comment. I did not tell the whole story though. That component was first written as a completely native application using a very special back door into Revit. We found it very unmaintainable (obviously!) and decided to rewrite it. A part of it we left as a native component (the part that communicates with the cloud), but the whole piece which interacts with Revit is now completely managed using only the public Revit API. Not only we got rid all of the maintenance issues, we also found, rather to our surprise, that this new add-on is now faster than the old one. Go figure! :-) Again, I do not want to argue over speed of native vs. managed applications. I am pretty sure that native applications would win. But it seems that a) it is not always that bad, and b) most users do not seem to care about that specifically.

Honestly, I do not know what the lack of 3D operations in Revit is about. I thought the available geometric API was reasonably adequate. Well, maybe not Maya-adequate, but somehow adequate. It is not my area, but I know one can obtain geometry of any solid and use transformation utilities that include rotation, moving, mirroring, and other. If the API still lacks functionality (even in the about to be released R2015), requests can be made via customer support or ADN. I am confident that with enough pressure we, developers will be ordered to add more. For, you see, we developers only implement what we are told by project management, which is told by marketing department, which is told by customers. And, in my experience, the most influential and vocal customers get what they want.

**Jeremy** I understand Adrian's request for functionality to rotate/translate/scale a solid.
I ran into that very same issue myself repeatedly when getting started with the Revit API.
This is a clear indication of not understanding the difference between the kind of CAD geometry you encounter in a parametric BIM model and what you see in products like AutoCAD and Maya.

Searching the Revit API for anything corresponding to the (very fundamental) ObjectARX transformBy method is futile.
Why?

Just imagine this simplest of all possible transformations: take a real building and scale it by a factor of 0.5.
What happens? It is too small to enter! All the wall types are corrupted, because they specify a thickness! You cannot enter through the main door, or only by getting down on your knees and crawling. All the furniture is reduced to doll size.

**Guy** I hate to think what Adrian would have thought about the Revit V8 API ;-)
Massive fundamental shifts in functionality and performance since then.

From my experience in Revit benchmarking, when working in a work shared project and particularly interacting with web services, true multithreading would not be a huge benefit IMO. Almost a bit of red herring without a major redesign of the architecture and workflows I think. Even though I've been vocal about this in the past ;-)

However, the ability to query the Db on multiple threads would be of measurable benefit. Threaded writes arguably significantly less useful in the context of a typical .rvt object tree, from my observations.

Regarding missing functionality. As an 'open' API it's in Autodesk's interests for the API to reach parity with the internal API. So I would hope there is a list somewhere that is being ticked off as quickly as possible.

The benchmark for me will be the ability to query an existing project and rebuild it purely via the API. In some respects it's pretty close, in others miles off. Autodesk have been pretty good at rolling out new public API's as internal API's change in the last few releases though.