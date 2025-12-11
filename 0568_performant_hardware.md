---
ai_optimized: true
author: Jeremy Tammik
complexity_score: 1.7
content_type: qa
optimization_date: '2025-12-11T11:44:14.189591'
original_url: https://thebuildingcoder.typepad.com/blog/0568_performant_hardware.html
post_number: 0568
reading_time_minutes: 2
series: general
slug: performant_hardware
source_file: 0568_performant_hardware.htm
tags:
- family
- revit-api
- rooms
- views
title: Performant Hardware
word_count: 398
---

### Performant Hardware

I spent all day yesterday in trains, which was unfortunate, since it was the most beautiful Sunday, and springtime is in full bloom.
It also forced upon me some unfortunate comparisons of the Swiss and Italian train systems.
Needless to say, I arrived later than expected.

Still, I ended up happily back in
[Verona, Italy](http://en.wikipedia.org/wiki/Verona), where I started seriously learning Italian two years ago, for a
[repeat visit](http://thebuildingcoder.typepad.com/blog/2009/01/verona-revit-api-training.html) to
provide a Revit API training class to
[Steel & Graphics srl](http://www.steel-graphics.com).

One funny thing about Verona is that it has almost exactly the same flag as Sweden, where I grew up:

![Verona and Sweden's flags](img/verona_sweden_flag.png)

It is also the city of Shakespeare's Romeo and Juliet.
Maybe I can manage to get to
[Sirmione](http://en.wikipedia.org/wiki/Sirmione) and the
[Sirmio peninsula](http://en.wikipedia.org/wiki/Sirmio) today
and complete the final training preparations there.

Meanwhile, here is a non-API related question affecting all Revit users from Augusto Gonçalves:

**Question:** To specify a new machine for Revit, what should I consider, i.e. spend more money on?
Amount of memory?
Speed of memory?
Processor clock?
Multi-processors?
HD speed?

**Answer:** Anthony Hauck provides a very succinct answer:

In order, and dependent on how large the models you wish to edit are:

1. More RAM- Faster Processor- More Processers- Faster RAM- Faster HD

For a more complete and long-winded answer, you may want to peruse the Revit
[model performance technical note](http://images.autodesk.com/adsk/files/revit_tech_note.pdf) pointed out by Martin Schmid.

It has been around for quite a while now and includes some detailed software optimisation suggestions as well, which are well worth reading and understanding for add-in developers as well as end users, including:

- Revit Platform Model Optimization and Best Practices
  - Arrays- Constraints- Design Options- DWG Files- Family Creation- Importing & Linking- Modeling Economically- Project Templates- Railings- Raster Images- Stairs- Upgrading Linked Projects to a New Version of Revit- Views- Volumes - Rooms and Spaces- Worksets- Worksharing- Revit Structure 2010 Software Optimization and Best Practices- Revit MEP 2010 Software Optimization and Best Practices

Note that this document applies to Revit 2010, although I am assuming that most remains valid for Revit 2012 as well.