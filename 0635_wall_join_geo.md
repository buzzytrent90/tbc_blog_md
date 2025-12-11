---
ai_optimized: true
author: Jeremy Tammik
complexity_score: 3.0
content_type: qa
optimization_date: '2025-12-11T11:44:14.303889'
original_url: https://thebuildingcoder.typepad.com/blog/0635_wall_join_geo.html
post_number: '0635'
reading_time_minutes: 3
series: general
slug: wall_join_geo
source_file: 0635_wall_join_geo.htm
tags:
- elements
- geometry
- revit-api
- transactions
- walls
title: Wall Joins and Geometry
word_count: 646
---

### Wall Joins and Geometry

Once again, here is an issue that shows how Revit keeps strict control over the BIM, and that the possibilities to override the built-in behaviour are intentionally limited:

**Question:** I have found the API method to AutoJoin elements, but I cannot seem to find any method that allows me to simply join two walls, as is possible in the Revit user interface. How can I achieve that?

**Answer:** Basically, as far as I understand Revit, it wants to take and keep full control over wall joins itself in order to ensure consistency of the BIM.
The Document.AutoJoinElements method does nothing more than force the join to happen immediately. Note that the Revit API help file RevitAPI.chm adds the following remark to this method:

Forces the elements in the Revit document to automatically join to their neighbours where appropriate.

Use this method to force elements in the document to automatically join to their neighbours. Note that when a transaction is committed there is an automatic call to automatically join elements.

I
[discussed the AutoJoinElements method](http://thebuildingcoder.typepad.com/blog/2010/05/autojoinelements.html) in
some depth and added a list of questions and answers on its usage when it was temporarily not automatically called on closing a transaction, but that issue resolved way back by
[Revit 2011 Web Update 1](http://thebuildingcoder.typepad.com/blog/2010/06/revit-2011-web-update-1.html).

So regardless of whether or not you call this method, the walls will be joined as Revit sees fit when the transaction is closed.

In the special case of walls, though, there is a limited amount of control left over to the user.
Maybe that is exactly what you are referring to?
This control is also accessible programmatically through the following new methods introduced in the Revit 2012 API:

#### Allow and disallow wall end joins

The new methods

- WallUtils.DisallowWallJoinAtEnd()
- WallUtils.AllowWallJoinAtEnd()
- WallUtils.IsWallJoinAllowedAtEnd()

provide access to the setting for whether or not joining is allowed at a particular end of the wall. If joins exist at the end of a wall and joins are disallowed, the walls will become disjoint. If joins are disallowed for the end of the wall, and then the setting is toggled to allow the joins, the wall will automatically join to its neighbours if there are any.

I think that is probably what you want, and also all you can on this context.

In addition, you have the following members on the LocationCurve class:

- ElementsAtJoin retrieves all elements joining to the end of this element's location curve or changes the order of elements participation in the end join with this location curve's end.- JoinType gets or changes the type of the join at the specified end.

In the user interface, you can also join multiple parallel walls to simulate a single layered wall.
This option is currently not available in the API, though.

**Question:** I am interested in analysing the wall geometry and wish to retrieve the original unjoined geometry of the wall.
However, everything that is accessible through the API is modified by the joins to the neighbouring walls.
How can I access the original geometry of the unjoined wall, as it is displayed in the user interface on selecting it?

**Answer:** One possible approach is to determine the neighbouring joined walls at the ends using ElementsAtJoin, temporarily disallow wall joins in a separate transaction, retrieve the unjoined wall geometry, and abort the transaction to restore the original state.
This same technique was also used to
[determine the gross material quantities of walls](http://thebuildingcoder.typepad.com/blog/2010/02/material-quantity-extraction.html) by temporarily removing all openings from them.
That sample is now available in the MaterialQuantities SDK sample.
It shows that the transaction handling for the temporary operation can be simply and cleanly encapsulated.