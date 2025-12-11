---
ai_optimized: true
author: Jeremy Tammik
complexity_score: 1.3
content_type: qa
optimization_date: '2025-12-11T11:44:14.768447'
original_url: https://thebuildingcoder.typepad.com/blog/0871_get_schedule_elements.html
post_number: 0871
reading_time_minutes: 1
series: elements
slug: get_schedule_elements
source_file: 0871_get_schedule_elements.htm
tags:
- elements
- filtering
- revit-api
- schedules
- views
title: Accessing all Elements in a Schedule
word_count: 250
---

### Accessing all Elements in a Schedule

Here is a quick little post, with a surprisingly short and simple answer to a short and seemingly difficult Revit API question.

Before getting to that, I will just mention that I arrived safe and sound in Paris for a developer conference here.
I had a walk in the sunset on the north bank of the Seine from the Gare de Lyon along the industrial areas on Quai de Bercy and crossed the river over the new-built Pont National to reach the rue de Tolbiac by a rather roundabout route.

![Sur le Pont National](file:///j/photo/jeremy/2012/2012-12-12_devdays_paris/sur_le_pont_national.jpeg)

So, back to Revit; how can we easily retrieve all the elements listed in a schedule view?

**Question:** The Revit 2013 API finally enabled the creation of schedule views, which is great.

I would also need to query that schedule and retrieve all the element ids of the elements contained in it.

Currently, I am using the workaround of exporting the schedule and then comparing with elements within the project, which takes a considerable amount of time.

Is there any simpler way to achieve this?

**Answer:** Yes, there is.

The schedule is a view, and its view id can be passed in to the filtered element collector just like any other.

That will return the relevant elements.

Sorry you had to go to all that unnecessary trouble :-)

Many thanks to
[Guy Robinson](http://redbolts.com) for
pointing this out!