---
ai_optimized: true
author: Jeremy Tammik
complexity_score: 1.9
content_type: news
optimization_date: '2025-12-11T11:44:14.612723'
original_url: https://thebuildingcoder.typepad.com/blog/0797_revitrubyshell.html
post_number: 0797
reading_time_minutes: 3
series: general
slug: revitrubyshell
source_file: 0797_revitrubyshell.htm
tags:
- python
- revit-api
- views
title: Meetings, Football, and RevitRubyShell
word_count: 565
---

### Meetings, Football, and RevitRubyShell

The whole EMEA DevTech team is meeting in Neuchâtel prior to the coming weekend's
[annual Autodesk football tournament](http://through-the-interface.typepad.com/through_the_interface/2012/07/thinking-about-au-2012-and-vacation.html) taking place here.
I won't be able to play myself, unfortunately, for the first time in many years, since I had a prior appointment with friends to climb the
[Spitzhorn](http://en.wikipedia.org/wiki/Spitzhorn) mountain this very weekend.

We also held a virtual meeting with Stephen Preston from the US, who is now our world-wide DevTech manager.
Here we are in virtual union, from left to right, Gary Wassell, Marat Mirgaleev, Vladimir Ananyev, Stephen (virtually), Jeremy, Adam Nagy and Philippe Leefsma:

![Virtual meeting with Stephen](file:////j/photo/jeremy/2012/2012-07-05_ne_emea_team_meeting/IMG_9873_virtual_meeting_2.JPG)

Meanwhile, I finally got around to taking a quick look at a completely new exciting topic:

#### RevitRubyShell

I
[repeatedly](http://thebuildingcoder.typepad.com/blog/2009/12/revit-python-shell.html)
[pointed out](http://thebuildingcoder.typepad.com/blog/2010/03/dynamically-load-and-debug-plugins.html)
[the power](http://thebuildingcoder.typepad.com/blog/2010/09/access-to-curtain-grid-panels.html) of
being able to interact directly with the Revit API using Daren Thomas'
[Revit Python shell](http://code.google.com/p/revitpythonshell),
originally implemented for Revit 2010, then for
[Vasari](http://thebuildingcoder.typepad.com/blog/2011/07/python-shell-in-revit-and-vasari.html),
and updated for
[Revit 2012 and Vasari 2.1](http://thebuildingcoder.typepad.com/blog/2011/09/python-shell-for-revit-2012-and-vasari-21.html) last autumn.

Here is another interactive interactive Revit programming environment by
[Håkon Clausen](http://www.hclausen.net) of
[Nosyko AS](http://www.nosyko.no).

His RevitRubyShell is based on the interpreted language
[Ruby programming language](http://www.ruby-lang.org),
which is more modern than
[Python](http://www.python.org) and
inherently object oriented.

According to the introductory blurb on its home page, Ruby is a dynamic, open source programming language with a focus on simplicity and productivity.
It has an elegant syntax that is natural to read and easy to write.

Kean Walmsley also had a look at it in its IronRuby incarnation for use in
[programming AutoCAD](http://through-the-interface.typepad.com/through_the_interface/ruby).
He presents a first overview and introduction to Ruby in his initial post on
[using IronRuby with AutoCAD](http://through-the-interface.typepad.com/through_the_interface/2009/04/using-ironruby-inside-autocad.html).

Here is the
[Ruby Shell source code](https://github.com/hakonhc/RevitRubyShell).

Hkon says: I hope you will find this useful.
I use it a lot for rapid prototyping and testing when programming against the Revit API.

It sports the simplest installation I have ever seen for any Revit add-in whatsoever, the absolutely ultimate one-click
[RevitRubyShell installer](http://www.hclausen.net/RevitRubyShell/setup.exe),
which should be opened IE and will install the add-in directly, setting up an external application creating its own simple single-button panel:

![RevitRubyShell add-in panel](img/ruby_panel.png)

Clicking the button launches the Revit Ruby shell including a snippet of sample code that can be immediately executed by pressing the blue 'Run' arrow button:

![RevitRubyShell console](img/ruby_console.png)

Well worth looking at, and might prove a huge programming productivity booster.

Many thanks to Håkon for developing and sharing this!