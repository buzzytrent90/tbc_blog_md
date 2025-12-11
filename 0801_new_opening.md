---
ai_optimized: true
author: Jeremy Tammik
complexity_score: 2.1
content_type: qa
optimization_date: '2025-12-11T11:44:14.620761'
original_url: https://thebuildingcoder.typepad.com/blog/0801_new_opening.html
post_number: 0801
reading_time_minutes: 4
series: general
slug: new_opening
source_file: 0801_new_opening.htm
tags:
- geometry
- references
- revit-api
- transactions
- walls
- windows
title: Creating an Opening
word_count: 759
---

### Creating an Opening

Here is a question that we have looked at in the distant past, brought up again and reinforced by Jon Mirtschin of
[Geometry Gym](http://geometrygym.blogspot.com),
working on BIM and model exchange with Rhino, Grasshopper et al:

**Question:** Sorry to bother you, but trying to get a quick answer to this problem.
If you happen to have 5 minutes to help, it will be greatly appreciated.

I've attached a sample Revit project.
It simply creates a polyline curve array and slab outline, and then attempts to add an internal opening.
This is a simplification from my IFC importer.
When I run it, I get an error message about a circular reference.

Can you please advise what is wrong and how to fix?
I've tried searching samples and online help to no avail.

**Answer:** Okey-doke, five minutes.

- Countdown starting at 9:45:34.70.- I read the mail and stored your attachment: 9:47:09.20;- Created a repository for your issue and unzipped: 9:48:12.65;- Started up Visual Studio to look at your project: 9:49:02.60;- Updated references to be able to compile: 9:49:55.64;- Bumped .NET framework version to 4.0: 9:50:33.15;- Fixed post-build event... or no, I'll just ignore that error: 9:51:21.87;

Sorry for being so literal, I just can't help it, and I even think it is fun :-)

Well, this looks like an interesting issue.

Before I dig any further, have you looked at the suggestions on creating a
[hole in a floor](http://thebuildingcoder.typepad.com/blog/2009/05/hole-in-a-floor.html)?
They include exploring the NewOpenings and ShaftHolePuncher SDK samples.

Do those SDK samples work for you?

In additional to that very old post, here is another half-old one which raises another interesting and important point for you to explore: an
[extra transaction may be required](http://thebuildingcoder.typepad.com/blog/2010/01/extra-transaction-required.html) after
creating the slab and before punching a hole through it.

Please try that out as well.

I expect a full report including answers to every single one of the questions above, ok?

Thank you!

P.S.: 9:57:42.51 :-)

**Response:**

Well, at least you have a great stop watch.
Most the time I'm too scared to even glance at the clock on the wall.
I actually should also ask you to report your Revit start up time;
I can probably allow a minute reduction for your great computers vs. my poor laptop ;-)

Ok, good news.
You are correct about the transaction.
It was one of the things I had in the back of my mind but hadn't checked yet.
Thanks for pointing it out.

Adding the manual transactions and setting a commit and start before adding the opening profile worked.
Note that when I went back to my real project, it has tiers of commands, so I was getting an exception to start a new transaction within the outer program command transaction.
I added a call to the doc.Regenerate method instead, and that resolved this issue, so problem solved.

I did look at the examples you mention, and will be using them.
In this case, the IFC data is presenting an outer and inner profile curves, so the nominated method is the best for this situation.

I don't know if I've answered every single one of your questions, but I'm close.

Here is my
[amended source code](zip/geometrygym_new_opening.cs.txt).
Now I'm off to implement the change in my real project.

Please feel free to blog about this, I have read through most of your blog several times now and to me it is an invaluable resource.
I would like to contribute in any small way.

Here is a sample image of a basic test:

![Sample model with new floor opening](img/geometrygym_new_opening.png)

This is an IFC generated from an Archicad model and now opened in Revit.
I already had the wall openings working, and now slabs are there too.
There are some more impressive and real world projects behind this.

**Answer:** For the stopwatch, I use the Windows command console time command, i.e. an old MS-DOS command :-)

```
C:\a\doc\revit\blog\ > time
The current time is: 10:43:48.29
Enter the new time:
```

My computer is pretty old and decrepit, actually.
I am up for renewal soon :-)

Thank you for the appreciation and nice example!