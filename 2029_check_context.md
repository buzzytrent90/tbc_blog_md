---
ai_optimized: true
author: Jeremy Tammik
complexity_score: 4.2
content_type: qa
optimization_date: '2025-12-11T11:44:17.312955'
original_url: https://thebuildingcoder.typepad.com/blog/2029_check_context.html
post_number: '2029'
reading_time_minutes: 4
series: general
slug: check_context
source_file: 2029_check_context.md
tags:
- csharp
- geometry
- python
- references
- revit-api
- sheets
- views
title: Check Context
word_count: 746
---

### API Context, Input State and DA4R Debugging
We present a long-awaited solution to check for a valid Revit API context and a whole bunch of short pointers to other topics of interest, mostly AI related:
- [Determining Revit API context](#2)
- [Detect Revit user input state](#3)
- [Easy Revit API](#4)
- [Gemini with image and video input](#6)
- [LLM is not self-aware](#7)
- [Generative AI transformer](#8)
- [Design to reduce junk data](#9)
- [C and C++ are risky](#10)
- [Ultra-processed food is toxic](#11)
#### Determining Revit API Context
Luiz Henrique [@ricaun](https://ricaun.com/) Cassettari finally cracked the
question [how to know if Revit API is in context](https://forums.autodesk.com/t5/revit-api-forum/how-to-know-if-revit-api-is-in-context/td-p/12574320):
> Revit API throws exceptions if your code is trying to execute Revit API methods in a modeless context, e.g., a WPF modeless view; that's the reason you need to use `ExternalEvent` to execute Revit API code in context.
> Sometimes you need to know whether code is running in context or if not, to just execute the Revit API code right away or send it to ExternalEvent to be executed.
> If you have access to `UIApplication` or `UIControllerApplication`, and if you try to subscribe to an event outside Revit API context, you are gonna have this exception: Invalid call to Revit API! Revit is currently not within an API context.
> Meaning: you can use that to know if your code is in context or not.
![In Revit API context check](img/ricaun_in_context.png "In Revit API context check")
> - [Code sample and video](https://ricaun.com/revit-api-context/)
- 14-minute video on [Tasks and InContext in Revit API](https://youtu.be/gyo6xGN5DDU)
> I'm using this technique using my open source library to manage the creation of an external event if it is not in context and enable it to run Revit API asynchronously in [ricaun.Revit.UI.Tasks](https://github.com/ricaun-io/ricaun.Revit.UI.Tasks).
Many thanks to ricaun for sharing this long-sought-after solution!
#### Detect Revit User Input State
In a related vein, we also discussed the question
of [detecting Revit user input state in real-time via Revit API](https://forums.autodesk.com/t5/revit-api-forum/detecting-revit-user-input-state-in-real-time-via-revit-api/td-p/12610444).
#### Easy Revit API
Big welcome to a new member in the Revit programming blogosphere,
[Easy Revit API](https://easyrevitapi.com/).
Welcome, Mohamed-Youssef.
Best of luck and much success with your blog and other projects!
#### Gemini with Image and Video Input
An impressive example of use of LLM with video input support is presented stating
that [the killer app of Gemini Pro 1.5 is video](https://simonwillison.net/2024/Feb/21/gemini-pro-video.
I tested using Gemini myself for
a [Revit API discussion forum](http://forums.autodesk.com/t5/revit-api-forum/bd-p/160) question
on [2024 dark theme colouring addins](https://forums.autodesk.com/t5/revit-api-forum/2024-dark-theme-colouring-addins/m-p/12614689) with
acceptable and useful results, afaict.
#### LLM is not Self-Aware
... Even
though [Anthropic’s Claude 3 causes stir by seeming to realize when it was being tested](https://arstechnica.com/information-technology/2024/03/claude-3-seems-to-detect-when-it-is-being-tested-sparking-ai-buzz-online/).
#### Generative AI transformer
A nice beginner's guide to understanding LLM explains
why [generative AI exists because of the transformer](https://ig.ft.com/generative-ai/).
#### Design to Reduce Junk Data
We are generating huge and ever-growing amounts of data, much of which is useless and never looked at again, so it is well worth
pondering – and avoiding — [design patterns that encourage junk data](https://css-irl.info/design-patterns-that-encourage-junk-data/).
#### C and C++ are Risky
70 percent of all security vulnerabilities are caused by memory safety issues, and many of those are automatically eliminated by working in a memory-safe programming language.
Therefore,
the [White House urges developers to dump C and C++](https://www.infoworld.com/article/3713203/white-house-urges-developers-to-dump-c-and-c.amp.html).
#### Ultra-Processed Food is Toxic
Talking about things we ought to dump, I seldom watch long videos, but this hour-long one had me mesmerised all the way through:
[the harsh reality of ultra processed food with Chris Van Tulleken](https://youtu.be/5QOTBreQaIk).