---
ai_optimized: true
author: Jeremy Tammik
complexity_score: 1.5
content_type: qa
optimization_date: '2025-12-11T11:44:14.983414'
original_url: https://thebuildingcoder.typepad.com/blog/0955_revitrubyshell.html
post_number: 0955
reading_time_minutes: 2
series: general
slug: revitrubyshell
source_file: 0955_revitrubyshell.htm
tags:
- python
- revit-api
- rooms
- views
title: RevitRubyShell for Revit 2014
word_count: 491
---

### RevitRubyShell for Revit 2014

[Håkon Clausen](http://www.hclausen.net) of
[dRofus AS](http://www.drofus.no) published
an updated version of
[RevitRubyShell](http://thebuildingcoder.typepad.com/blog/2012/07/meetings-football-and-revitrubyshell.html#2).
In Håkon own words:

I have updated the add-in to 2014 and added new docking and idling features.

It was all completely modal in Revit 2013, which was a little inconvenient.
There was some trouble with the key input focus working with modeless and idling in earlier versions, e.g., using the tab and enter keys.

In Revit 2014, the new dockable panel feature in combination with idling solves that cleanly.

The add-in is updateable from the [RevitRubyShell setup](http://www.hclausen.net/RevitRubyShell/setup.exe).
If you have any problems, try to uninstall the previous version first.

The new source is also available from the [RevitRubyShell GitHub repository](https://github.com/hakonhc/RevitRubyShell).

If you have any feature suggestions, please let me know.

Unfortunately, the SharpDeveloper Python and Ruby access built into Revit itself does not support direct interpreted interaction with the Revit API from the console, so this is currently the only way to have a real hands-on real-time experience with the API.
All other options require you to implement and compile a macro or an external command and launch that.

#### Running Revit from the Debugger on a Mapped Drive

Here is a generic Visual Studio 2012 security issue that can also affect Revit add-in development:

**Question:** I am using Visual Studio 2012 Professional as my authoring environment for Revit add-ins, and encountering the following problem, not limited to any particular version of Revit:

When I run Revit from the debugger, its file dialogues do not display or allow access to any mapped drive letters.
Local drives display, such as C: and DVD drives, for example, but no network drives or even local path drives mapped with the SUBST command.
I can access network folders via their UNC paths, but cannot browse to the mapped drive.

What's up?

**Answer:** You are running Visual Studio in admin mode, but executed your command as a normal user.

This is causing you to hit this
[Visual Studio security issue](http://social.msdn.microsoft.com/Forums/en-US/vssetup/thread/b833bde0-c3a3-46e3-8408-3ed5f0090c93).

#### Arrival in Massachusets and Tech Summit Pre-Recording

I arrived safe and sound in the USA and spent a pleasant, warm, sunny morning in
[Marblehead](http://en.wikipedia.org/wiki/Marblehead,_Massachusetts).

![Marblehead](img/Marbleheadneck.jpg)

Before leaving, I posted a 30-minute preview
[video recording](http://thebuildingcoder.typepad.com/room_editor_preview/index.html)
of my upcoming Tech Summit presentation on my
[cloud-based simplified 2D Revit model editor](http://thebuildingcoder.typepad.com/blog/2013/05/my-cloud-based-2d-editor-implementation-status.html).
The live presentation will probably also be recorded, so it remains to be seen which one I like better in the end.