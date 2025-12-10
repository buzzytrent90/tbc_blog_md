---
post_number: "0553"
title: "Internal Imperial Units"
slug: "imperial_units"
author: "Jeremy Tammik"
tags: ['revit-api']
source_file: "0553_imperial_units.htm"
original_url: "https://thebuildingcoder.typepad.com/blog/0553_imperial_units.html"
---

### Internal Imperial Units

To start off this week, here is a pretty useless question that still seems to be of great interest to many people:

**Question:** Odd question perhaps, but why does Revit internally work in imperial units?

**Answer:** I do not find your question odd at all; many others wonder that as well, I am sure.

Briefly, I would hazard that the answer is simply that this is a religious question, or a question of taste, or historically evolved.

Does that satisfy your curiosity?

Here is a bit more background information in a little more depth:

1. Revit was first made in the US, and originally only for the US market. US architects work with imperial units.- At the time it was simplest to write Revit to use imperial units internally (feet for length) to match customer needs.- When Revit's market expanded, support was extended for metric units. But the existing database structures were preserved for backwards compatibility.- In addition, Revit started expanding in capability and had to cover additional new areas requiring other units such as force, mass, etc. For those new capabilities, metric units were chosen as being more universally understood and useful.- When Revit's API was introduced, the simplest method of introduction was to expose the database values directly. Thus feet for length, metric for everything else.

I hope this helps, useless as the information may be!

Many thanks to Giles and Scott for contributing to this!