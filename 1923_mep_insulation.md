---
ai_optimized: true
author: Jeremy Tammik
complexity_score: 3.1
content_type: qa
optimization_date: '2025-12-11T11:44:17.051714'
original_url: https://thebuildingcoder.typepad.com/blog/1923_mep_insulation.html
post_number: '1923'
reading_time_minutes: 5
series: mep
slug: mep_insulation
source_file: 1923_mep_insulation.md
tags:
- csharp
- elements
- filtering
- parameters
- python
- revit-api
- sheets
- views
- mep
title: Mep Insulation
word_count: 903
---

### Sci-Fi, Languages and Pipe Insulation Retrieval
Three quick notes from my recent email correspondence and reading:
- [Pipe insulation retrieval performance](#2)
- [Programming languages to learn](#3)
- [Agency by William Gibson](#4)
#### Pipe Insulation Retrieval Performance
Alexander [@aignatovich](https://forums.autodesk.com/t5/user/viewprofilepage/user-id/1257478) [@CADBIMDeveloper](https://github.com/CADBIMDeveloper) Ignatovich, aka Александр Игнатович,
shares an interesting observation on a performance issue retrieving MEP pipe insulation elements using `GetDependentElements`:
Recently, I faced with a performance issue getting pipe insulation.
My previous implementation looked like this:
```csharp
var pipeInsulation = pipe
.GetDependentElements( new ElementClassFilter(
typeof( PipeInsulation ) ) )
.Select( pipe.Document.GetElement )
.Cast()
.FirstOrDefault();
```
I didn't notice it before because I tested the code on a small model.
However, in the big model, the entire calculation took over an hour.
Calculations were also huge, so I spent some time trying to figure out what is going wrong.
The improved solution looks like this:
```csharp
var pipeInsulation = InsulationLiningBase
.GetInsulationIds( pipe.Document, pipe.Id )
.Select( pipe.Document.GetElement )
.OfType()
.FirstOrDefault();
```
With that, the entire calculations take seconds instead of hours.
This issue is related only to MEP; I haven't faced any other performance issues using the `GetDependentElements` method.
Thank you very much for the interesting observation, Alex!
#### Programming Languages to Learn
A frequent question is which programming language to learn to implement Revit add-ins.
Here it comes up again, with an F# twist:
\*\*Question:\*\* We are now starting our in-house computational strategy in order to automate processes on both Revit-Dynamo and Inventor-iLogic and I am struggling to decide which language code we should start to learn.
C#, Python or F#?
I am takin my first steps into coding, but I have a few years of experience on Dynamo & Grasshopper.
I would also like to add that our Inventor application has an automation interface developed by a consultant company written in F# that we would like to take control over in the long term.
Long story short: I would like to understand the pro and cons of which of these three languages should we start to learn to better unify a future workflow between Revit and Inventor, keeping in mind that we have something in-house already developed in F#.
\*\*Answer:\*\* Since the Revit API is completely .NET based and all .NET languages are completely interoperable, it really does not matter much at all which one you learn and use.
Any one of them can be used to interact fully with any other.
Furthermore, all .NET languages compile to the same underlying IL or intermediate language.
From IL, you can decompile back into any other .NET language, making it easy to switch back and forth between languages and even transform your code base from one to another.
Therefore, obviously, you need not really worry about learning F# at all, if you are not interested in procedural programming yourself.
In short, I would say:
- Python: best for learning, and for Dynamo
- C#: best for pure Revit API, most example code available, cleanest .NET interface
- F#: best for generic stateless procedural logical lambda computation, and you'll need it in the long run anyway
Here are some other thoughts on this topic:
- [What Language to choose for a Revit Add-In?](https://thebuildingcoder.typepad.com/blog/2020/04/2021-migration-add-in-language-and-bim360-login.html)
- [A .NET Language Learning Resource](http://thebuildingcoder.typepad.com/blog/2009/07/a-net-language-learning-resource.html)
- [Running Language Code and More Exporters](http://thebuildingcoder.typepad.com/blog/2012/07/determine-running-language-code.html)
- [Revit Future and Saving User Configuration Settings](http://thebuildingcoder.typepad.com/blog/2015/08/revit-future-and-saving-user-configuration-settings.html)
- [RTC Classes and Getting Started with Revit Macros](http://thebuildingcoder.typepad.com/blog/2015/10/rtc-classes-and-getting-started-with-revit-macros.html)
- [Choose a Programming Language](http://thebuildingcoder.typepad.com/blog/2015/10/rtc-classes-and-getting-started-with-revit-macros.html#15)
- [Converting Code from One Language to Another](http://thebuildingcoder.typepad.com/blog/2015/10/rtc-classes-and-getting-started-with-revit-macros.html#16)
- [Most Popular Programming Languages 1965-2019](https://thebuildingcoder.typepad.com/blog/2019/10/invitation-to-devcon-visual-programming-in-infrastructure.html#5)
- [The Most Popular Programming Languages 2015](http://thebuildingcoder.typepad.com/blog/2013/05/removing-extreneous-mac-architectures-and-languages.html#3)
#### Agency by William Gibson
I just finished reading \*Agency\* by William Gibson, a brilliant sci-fi including a critical look at politics and its rather helpless and fruitless attempts to control climate change, big business, the current pandemic, probably AI, soon, and other interesting challenges.
It includes an original new (for me, anyway) idea on time travel and its impossibility.
It treats the possibility of benevolent and humane AI with a lot of optimism, which I agree with.
Gibson is a true visionary.
He also coins (?) the term CCA, competitive control area, a territory where it is unclear who holds the power: government, warlords, multinational companies, criminal organisations...
one wonders whether that might be an accurate and critical way of viewing our current real world right now...
![William Gibson Agency](img/william_gibson_agency.jpg "William Gibson Agency")
#### Addendum – Jackpot
Wow! Good news!
I just discovered that Agency is part two of the [Jackpot trilogy](https://en.wikiquote.org/wiki/Jackpot_trilogy).
So, next thing I do is order part one, [The Peripheral](https://en.wikipedia.org/wiki/The_Peripheral).