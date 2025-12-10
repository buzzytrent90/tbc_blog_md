---
post_number: "0798"
title: "RevitRubyShell Implementation and Installer"
slug: "revitrubyshell"
author: "Jeremy Tammik"
tags: ['doors', 'elements', 'family', 'filtering', 'python', 'revit-api', 'windows']
source_file: "0798_revitrubyshell.htm"
original_url: "https://thebuildingcoder.typepad.com/blog/0798_revitrubyshell.html"
---

### RevitRubyShell Implementation and Installer

I had a very impressive train ride on Friday evening on my way up to meet my friends to climb the
[Spitzhorn](http://en.wikipedia.org/wiki/Spitzhorn) mountain,
happening to take the
[panoramic express](http://www.myswitzerland.com/en/montreux-chateau-d-oex-gstaad-panoramic-express.html) up
from Montreux to Gstaad, completely by chance.
I was just expecting a normal train ride, but it turned out to be extremely beautiful, the most comfortable way I have yet experienced to get an almost overwhelming impression of Swiss mountain beauty without having to do any hiking, just looking out of the train window.
We made it to the summit on Saturday
([photos](http://www.facebook.com/media/set/?set=a.4253220338006.2173087.1510729143&type=1)):

![Jeremy on Spitzhorn peak cross](file:////j/photo/jeremy/2012/2012-07-07_spitzhorn/jeremy_on_gipfelkreuz.jpg)

Last week, I provided a
[short description](http://thebuildingcoder.typepad.com/blog/2012/07/meetings-football-and-revitrubyshell.html#2) of
the interactive real-time Revit programming environment
[RevitRubyShell](https://github.com/hakonhc/RevitRubyShell) provided
by
[Håkon Clausen](http://www.hclausen.net) and
mentioned how impressed I was by its minimalistic single-click
[installer](http://www.hclausen.net/RevitRubyShell/setup.exe).

In the train on the way up to Gstaad, I took a closer look at it.
Håkon points out that RevitRubyShell is heavily inspired by
[RevitPythonShell](http://code.google.com/p/revitpythonshell) and
Jimmy Schementi's article about
[embedding IronRuby](http://blog.jimmy.schementi.com/2009/12/ironruby-rubyconf-2009-part-35.html).

The entire source code is provided on the
[GitHub social coding platform](http://en.wikipedia.org/wiki/GitHub).
[GitHub](https://github.com) is a web-based hosting service for software development projects using the
[Git revision control system](http://git-scm.com).

I found the RevitRubyShell source impressive and edifying, sporting a number of aspects well worth looking at in depth for most Revit API programmers, and for that matter most .NET programmers in general as well.
I probably missed lots of other interesting features, but here are some that sprang to eye:

- Optimal use of numerous bits of .NET functionality to achieve a lot with a minimum of code.- Use of XAML to implement its shell window form, including use of tabs and command buttons.- A couple of useful extension methods for the .NET KeyEventArgs and RichTextBox classes.- One-click installer making use of RevitAddInUtility and its RevitProductUtility and AddInManifestUtility classes.

I sat down with the RevitRubyShell console and tried to figure out how to count the number of doors in the Revit basic sample model.

It took me a bit of fiddling and googling, but this seems to do the trick:

```
bic = BuiltInCategory.OST_Doors

doors = FilteredElementCollector.new(doc)
  .OfCategory( bic ).ToElements();

count = doors.select {|d| d.is_a?( FamilyInstance )}.size

=> 29
```

Only three lines of pretty obvious code, even if you know just a little bit of Ruby and the Revit API:

![Door count in Ruby](img/ruby_door_count.png)

I find this pretty impressive, once again.
The handling is totally intuitive, and everything simply works.
Brilliant.

Many thanks again to Håkon for developing and sharing this!