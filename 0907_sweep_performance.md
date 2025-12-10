---
post_number: "0907"
title: "Sweep Family Performance Enhancement"
slug: "sweep_performance"
author: "Jeremy Tammik"
tags: ['csharp', 'elements', 'family', 'python', 'revit-api', 'transactions', 'views', 'windows']
source_file: "0907_sweep_performance.htm"
original_url: "https://thebuildingcoder.typepad.com/blog/0907_sweep_performance.html"
---

﻿

### Sweep Family Performance Enhancement

Now I am back at work again, enjoying the queries coming in from developers, postponing other important things such as my own long-term class and presentation preparations etc.
Addicted to helping, that's me.

Anyway, here is a piece of doubly good news, in that the developers mentioned below did a good job of helping themselves, in addition to achieving a radical speed improvement from 40 hours down to 3 minutes, i.e. just 0.125% of the original time, to programmatically generate some Revit families.

**Question:** The problem was initially reported by
[Hu Zhao](http://www.facebook.com/hu.zhao.3) in a recent
[comment](http://thebuildingcoder.typepad.com/blog/2013/02/mp3-manipulation-using-python-mutagen-and-ffmpeg.html?cid=6a00e553e168978833017d410ee261970c#comment-6a00e553e168978833017d410ee261970c):

> I have 80,000 elements to generate, now I use sweep; do you know which is faster of sweep and extrusion?
> Which can offer a better performance for the computer?
>
> It took 40 hours for the sweep operations.
>
> Is it possible to accelerate this?
>
> In Rhino script, only 10 minutes is enough for this operation.

Hu's colleague and Revit API blogger
[David Tan](http://blog.csdn.net/flower4wine)
[clarifies](http://thebuildingcoder.typepad.com/blog/2010/06/to-regenerate-or-not-to-regenerate.html?cid=6a00e553e168978833017c36e43308970b#comment-6a00e553e168978833017c36e43308970b):

> Zhao's scenario is to create about 80,000 sweeps in Revit in one shot.
> Per his test, it will take about 20 hours, while his workmate can achieve it in Rhino within 10 minutes.
> Zhao expects that Revit could do faster.
>
> As Zhao's test was done in an open family document (with UIDocument), I tried to use in-memory document but it still take about 18 hours to draw all 80K sweeps.
>
> Zhao noticed that although all the drawing work is done in a single transaction, it seems that Revit tries to regenerate the document after there are sweeps created already automatically.
> We did set the RegenerationOption to be Manual.
> We say it because we saw Revit progress bar in the bottom-left corner indicates it.
>
> Our questions:
>
> 1. Can we prohibit Revit's automatic regeneration?
> 2. More straightly, do we have any approach to get the job done faster?
>
> Thanks a lot! And Happy Chinese Snake Year!

**Answer:** There may be a stray regeneration happening in sweep creation.

If you can demonstrate that (with a sample that creates something less than 80,000 sweeps, please :-), please provide a reproducible case for further analysis.

It is also worth asking what the use case is for 80,000 sweeps in one family.
Are you possibly modelling things so detailed that that a building model with multiple instances of this will have performance problems?

Maybe you could add an image of what you are creating to motivate the development team by helping them understand.

**Response:** Here is an image showing what we are trying to achieve:

![Roof with sweeps](img/sweep_performance_roof.jpeg)

Each sweep corresponds to a single wooden roof in the image.

Here is also
[Hu Zhao's source code](zip/sweep_performance_david.zip).
We used an in-memory document and added a StopWatch to measure the time consumed.

The source code implements an external command in Program.cs.
It will pop up a window and let you select a XML file, which contains creation data of all the sweeps.
You can find the XML file along with the .sln file.
To make things simple, I only put 2000+ sweeps creation data in the XML file.

Finally I want to share the good news from Hu Zhao.
He resolved the performance problem by splitting all sweeps into 24 families and then combining them together.
It only takes 3 minutes now.
Of course, the cost is the reduced granularity; the model now no longer has individual roofs with embedded family instances.
But as he is OK with it, we can say we have found the solution.

Hu added the following
[more detailed project description](http://thebuildingcoder.typepad.com/blog/2013/02/mp3-manipulation-using-python-mutagen-and-ffmpeg.html?cid=6a00e553e168978833017ee88d66c1970d#comment-6a00e553e168978833017ee88d66c1970d):

I am now working in a project plan team for the design of Arch\_Tec\_Lab in ETH Zürich
([youtube link](http://www.youtube.com/watch?v=tzKfXra9o54)).

The parametric roof is generated in Rhino by my colleague, and the wooden beams will be assembled by robotic arms.

Before I joined this team, they imported the roof (including roof wooden structure) in DWG format, which did not display correctly in some plan and section views.
In 3D view, it led to Revit crash because of too many triangle meshes in this imported format.

So I wrote a plugin to import the wooden beams (around 80k) of the roof.

We choose XML to exchange the position information, and I choose sweep instead of extrusion to generate each beam.

For now, the roof structure is divided into 7 parts of XML, which generates 7 families.
It took me about 20 hours by using 3 dell precision computers.

Because the roof will be modified several times in the following days, I am now searching for a way to reduce the generation time.

I found that the speed of sweep is slower when the amount increased, and the regeneration that I cannot avoid is especially slow when the amount is above 1500.

Yesterday I found a way to accelerate the generation greatly.

I did like this: divided each 1 part into 24 pieces sections with the mark in XML, then generate each section into single family, and then combine them into a major family.
These actions are all made by the plugin.

This is much more quicker.

Let me introduce our project once again.

The Arch\_Tec\_Lab is a role model project of department of Architecture, ETH Zürich.
The team includes most important professors in the Institute of Technology in Architecture.
The project aims to realize the BIM control in the life circle.
Now we are using Revit to an extremely detail, and we enjoy the cloud rendering in Autodesk 360, except we wish it would also support animation rendering.
Now we are working on the construction drawings in Revit.

In Switzerland, few architects use Autodesk products, because of the majority of the MAC users.
But we want build a model for the architects and the students in department.

Many thanks to David and Hu for their research and sharing this remarkable performance enhancement!

**Addendum:** For completeness' sake, here is the
slow source code and the
complete fast solution.

Hu clarifies: the code we posted yesterday is the slow version.
The 'slow source code' shows how it works.
The 'complete fast solution' is the quicker version.

There are 8 parts to generate.
I finished all the work above in 20 hours using 3 computers, and now I can generate the smallest part in 3 minutes.
Actually, the bigger part took about 6 hours by the slow method, and now takes about 50 minutes.

The time depends on how many parts I separate.
I never test all the 80000 sweeps in one run, maybe it will cost one month ~:D – because the speed runs slower when time goes.