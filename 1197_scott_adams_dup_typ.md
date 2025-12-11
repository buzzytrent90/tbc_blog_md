---
ai_optimized: true
author: Jeremy Tammik
complexity_score: 2.3
content_type: qa
optimization_date: '2025-12-11T11:44:15.491004'
original_url: https://thebuildingcoder.typepad.com/blog/1197_scott_adams_dup_typ.html
post_number: '1197'
reading_time_minutes: 4
series: general
slug: scott_adams_dup_typ
source_file: 1197_scott_adams_dup_typ.htm
tags:
- elements
- references
- revit-api
- views
title: How to Fail, Still Win Big and Handle Duplicate Types
word_count: 856
---

### How to Fail, Still Win Big and Handle Duplicate Types

Let's start this week with a couple of interesting recent topics that have been hanging around a while up, one Revit API related and not:

- [How to fail at almost everything and still win big](#2)
- [Handling duplicate types when copying elements](#3)

#### How to Fail at Almost Everything and Still Win Big

I received a very nice present from my colleague Steve Mycynek, Principal Engineer of the Revit API development team, at the Autodesk
[Technical Summit](http://thebuildingcoder.typepad.com/blog/2014/06/technical-summit-day-1-and-removing-rvt-references.html) in
Toronto:
[How to Fail at Almost Everything and Still Win Big: Kind of the Story of My Life](http://dilbert.com/blog/entry/how_to_fail_at_almost_everything_and_still_win_big_kind_of_the_story_of_my_life),
by Scott Adams, author of the famous
[Dilbert](http://dilbert.com) comic series.

I was surprised how much I liked the book, and how impressed I am by Adams' original and creative way to analyse and pursue the topic of success.

He underlines the importance of systematic approaches, reprogramming yourself and your habits, especially regarding health, exercise and failure.

Here are Steve's thoughts on and motivation for his generosity:

Scott Adams claims that most people’s definition of 'genius' is 'someone who thinks just like they do, only better'. In that way, because often Adams says what I’m thinking, but with style and lots of best-selling book money, I, by his definition, think he’s a genius, especially when he makes serious comments about life, work, and happiness :-)

One point Adams makes is that 'every skill you have doubles your chances of success'. I see this over and over again. For example, a few years back, I created an interactive Ruby read-eval-print loop for a special programming environment within another application to work with various API-related data structures at the command line. Not much came of it, but recently, when learning Ruby-on-Rails, I came across the 'rails console' tool, which is, what-do-you-know, a 'read-eval-print loop for Ruby with all of your rails application data-structures pre-loaded so you can interactively query, edit, and create data via code fragments.' I was instantly motivated to use this to learn, develop, and debug rails issues, but I probably wouldn’t have gotten on board with the rails console with such enthusiasm if I hadn’t spent time building silly read-eval-print loop applications years earlier.

I’ll admit that I have sort of an addiction to skill building and bit of a manic survival instinct when it comes to learning in general. One thing that really helps with this is all the insight Adam has into health. I find that the better my diet and the more exercise I get, the more I can get up early and have energy to quickly try out a new library or introduce a new feature into a toy application I’m working on.

This gets into a later point that Adams makes, which is 'health + freedom = happiness.' If you’re not well, nothing will make you happy, even if you have millions of dollars. If you’re basically healthy but chained to a job you hate – pretty much the same thing. So, I come full circle, trying to be as healthy as possible so I can have as many skills as possible so that I can have the most opportunities and ultimately freedom as possible – so I can be as happy as possible.

There’s more to it than that, and if you want more Adams insight, check out 'God’s Debris' and 'Stick to Drawing Cartoons, Monkey Brain' – more of his philosophy is covered there. In a nutshell, Adams is the self-help guru that engineers can stomach, right up there with Steve McConnell of 'Code Complete' fame. He’s sort of the 'wise uncle' of XKCD. That’s why I started the tradition of giving his non-fiction books to co-workers – I hope he writes more so I can keep doing it.

Thank you very much, Steve, for this wonderful and enriching gift!

#### Handling Duplicate Types When Copying Elements

Another colleague of mine,
[Augusto Gonçalves](http://adndevblog.typepad.com/aec/augusto-goncalves.html) of
the ADN developer support team, presented a useful note on
[handling duplicate types on CopyElements](http://adndevblog.typepad.com/aec/2014/08/handling-duplicate-types-on-copyelements.html),
which can raise a 'Duplicate Types' warning saying 'The following Types already exist but are different...':

!['Duplicate Types' warning](img/duplicate_types.png)

It is based on an answer by Arnošt Löbel, Sr. Principal Engineer of the Revit development team, on the Revit API discussion forum thread on
[how to implement IDuplicateTypesNameHandler for the CopyPasteOptions class](http://forums.autodesk.com/t5/revit-api/how-to-implement-iduplicatetypesnamehandler-for-the/m-p/4926792).

It shows a use of the CopyPasteOptions class and a custom duplicate type names handler – another one is provided by the
[DuplicateViews SDK sample](http://thebuildingcoder.typepad.com/blog/2013/05/copy-and-paste-api-applications-and-modeless-assertion.html#2.1).