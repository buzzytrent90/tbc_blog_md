---
ai_optimized: true
author: Jeremy Tammik
complexity_score: 1.9
content_type: qa
optimization_date: '2025-12-11T11:44:14.735059'
original_url: https://thebuildingcoder.typepad.com/blog/0851_gross_slab_boundary.html
post_number: 0851
reading_time_minutes: 4
series: general
slug: gross_slab_boundary
source_file: 0851_gross_slab_boundary.htm
tags:
- doors
- elements
- geometry
- references
- revit-api
- transactions
- views
- walls
- windows
title: Gross Slab Boundary and the Temporary Transaction Trick
word_count: 828
---

### Gross Slab Boundary and the Temporary Transaction Trick

Last week I provided an
[updated version](http://thebuildingcoder.typepad.com/blog/2012/10/slab-boundary-revisited.html)
of the
[CmdSlabBoundary](http://thebuildingcoder.typepad.com/blog/2008/10/slab-boundary.html)
external command to determine a floor slab boundary.

Here is a follow-up to that issue:

**Question:** The solution provided to
[retrieve the slab boundary](http://thebuildingcoder.typepad.com/blog/2012/10/slab-boundary-revisited.html) works
fine for me if no openings are cut out.

I have a situation where my slabs are cut by openings, however, and I need to retrieve the original uncut gross slab boundary.

Here is an example with two slabs A and B and an opening cutting out part of both of them:

![Two slabs cut by an opening](img/gross_slab_boundary_1.png)

The solution provided retrieves the cut out net slab boundary:

![Net cut slab boundary](img/gross_slab_boundary_2.png)

How can I obtain the following original gross uncut slab boundary?

![Gross original uncut slab boundary](img/gross_slab_boundary_3.png)

**Answer:** One way to obtain the unmodified geometry of the slab is to use the temporary transaction trick to temporarily eliminate all openings and other elements affecting its shape.

This technique was used to
[determine gross material quantities](http://thebuildingcoder.typepad.com/blog/2010/02/material-quantity-extraction.html) of
floors, walls and roofs in the material quantities sample from Scott Conover's Autodesk University 2009 class on
[analysing building geometry](http://thebuildingcoder.typepad.com/blog/2010/01/analyse-building-geometry.html),
which was later add to the Revit SDK as the
[MaterialQuantities](http://thebuildingcoder.typepad.com/blog/2010/02/material-quantity-extraction.html) sample.

It shows how the transaction handling for the temporary operation can be simply and cleanly encapsulated.

It analyses the material quantities of walls, floors, and roofs, and outputs the result to Excel by implementing the following steps:

1. Collect all roof elements- Iterate through each material found in each roof element- Find the net volume and area of the material- Store the material quantities- Write the results to the output file- Find the gross material quantities
             1. Start a new transaction- Delete the elements that cut the host (doors, windows, openings)- Find the volume and area for each material- Store the material quantities- Write the results to the output file- Rollback the transaction- Repeat the above steps for walls and floors- Open the output file in Microsoft Excel

To determine the boundary lines of the floor slab with all opening removed, you could proceed in the same fashion as described in step 6 for obtaining the gross material quantities.

Temporarily delete then openings in a separate transaction, retrieve the original slab geometry, and abort the transaction to restore the original state.

This technique is easy to implement.
We reused this method for many different purposes a number of times, e.g.

- [Determine host reference](http://thebuildingcoder.typepad.com/blog/2009/06/host-reference.html)- [Material quantity extraction](http://thebuildingcoder.typepad.com/blog/2010/02/material-quantity-extraction.html)- [Unmodified element geometry](http://thebuildingcoder.typepad.com/blog/2010/02/unmodified-element-geometry.html)- [Object relationships](http://thebuildingcoder.typepad.com/blog/2010/03/object-relationships.html)
        ([VB](http://thebuildingcoder.typepad.com/blog/2010/03/object-relationships-in-vb.html))- [Retrieve all model elements via a temporary 3D view](http://thebuildingcoder.typepad.com/blog/2010/10/model-elements-revisited.html)- [Access to sketch and sketch plane](http://thebuildingcoder.typepad.com/blog/2010/11/access-to-sketch-and-sketch-plane.html)- [Retrieve unjoined wall geometry](http://thebuildingcoder.typepad.com/blog/2011/08/wall-joins-and-geometry.html)- [Retrieving detailed wall layer geometry](http://thebuildingcoder.typepad.com/blog/2011/10/retrieving-detailed-wall-layer-geometry.html)- [Unit conversion and display string formatting](http://thebuildingcoder.typepad.com/blog/2011/12/unit-conversion-and-display-string-formatting.html)- [Determine Revit demo mode](http://thebuildingcoder.typepad.com/blog/2012/03/determine-revit-demo-mode.html)

**Addendum:** Arnošt Löbel pointed out that
[this will not always work](http://thebuildingcoder.typepad.com/blog/2012/11/temporary-transaction-trick-touchup.html).
Some modifications require the transaction to be committed before they take full effect.
The
[workaround](http://thebuildingcoder.typepad.com/blog/2012/11/temporary-transaction-trick-touchup.html) for
that is to enclose it within a transaction group and roll back the group at the end.

#### Apphack @ AU

Do you have an interesting AutoCAD app you would like to publish?

Will you attend Autodesk University 2012 and would you like to win an attractive prize?

If so, take a look at the
[APPHACK @ AU Hackathon](http://promoshq.wildfireapp.com/website/6/contests/296215):

![Apphack @ AU](img/au_2012_apphack.png)

The
[official rules](http://promoshq.wildfireapp.com/website/6/contests/296215/rules)
list some great prizes.
Unfortunately, they also restrict the participation to AutoCAD apps, so Revit add-ins are not permissible.

Kean Walmsley provides a
[a few more details](http://through-the-interface.typepad.com/through_the_interface/2012/10/apphack-au-2012.html).