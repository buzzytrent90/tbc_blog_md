---
ai_optimized: true
author: Jeremy Tammik
complexity_score: 1.5
content_type: documentation
optimization_date: '2025-12-11T11:44:14.480442'
original_url: https://thebuildingcoder.typepad.com/blog/0733_connecting_pieces.html
post_number: '0733'
reading_time_minutes: 2
series: general
slug: connecting_pieces
source_file: 0733_connecting_pieces.htm
tags:
- parameters
- revit-api
- walls
- windows
title: Connecting Pieces, Like Navisworks
word_count: 434
---

### Connecting Pieces, Like Navisworks

The motto of last year's world tour of DevDays developer conferences was ***Connecting the Pieces***, as in Autodesk suites, as in getting bits and pieces to work together seamlessly, as in the whole is greater than the sum of the parts.

This has also been reflected very clearly in the past few years of Revit product and API enhancements.
So much is about integration.

Henrik Bengtsson of
[Lindab](http://www.lindab.se) embraces
this idea and created a little
[gateway to Navisworks](http://www.youtube.com/lindabgateway):

In Henrik's words:

We implemented a plugin for Navisworks that shows all extended data that is created in Revit using our LindabRevitTools add-in.
The plugin is a way of reaching out to everyone working on site, since they use Navisworks a lot.
![Lindab gateway](img/lindabgateway.png)

All information that I want to share in Navisworks is written to a set of Revit parameters. These are shared ones in this case, since I don't want to mix anything up with anyone else.

These parameter values are read and displayed in Navisworks.
I use two different Navisworks plugin types, one to create the window and one that handles the events.
Most of the parameters are usually shown using the ordinary properties window, as long as they have a value.
On the other hand, I want the users to get the right amount of data (no more, no less) and formatted in a way that is easily to read.
That isn't achieved in any other way than this.

I decided to do all the hard work inside Revit instead of Navisworks, such as creating a class with the information that is about to be exchanged between Revit and Navisworks.

Half of the information that I show in Navisworks must be created inside Revit anyway, since it is plugin-specific information like quantities of material etc., so I didn't really have much choice anyway.

The first comments have been really good, since a lot of people on the construction site have purchased Navisworks licenses and are comfortable with the environment.

They can do a walk-through and simply click on any wall to see the exact amount of steel profiles, insulation, and board material needed to build it.
Are we talking BIM or what?

No idea, but some people might think so... ;-)

Cross-platform work like this feels like a good way to go...

I will definitely do more stuff available for Navisworks.

For more Revit and Navisworks videos from Lindab,
[www.youtube.com/lindabgateway](http://www.youtube.com/lindabgateway).