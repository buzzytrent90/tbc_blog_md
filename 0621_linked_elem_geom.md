---
post_number: "0621"
title: "Linked Element Geometry Access"
slug: "linked_elem_geom"
author: "Jeremy Tammik"
tags: ['elements', 'family', 'filtering', 'geometry', 'references', 'revit-api', 'walls']
source_file: "0621_linked_elem_geom.htm"
original_url: "https://thebuildingcoder.typepad.com/blog/0621_linked_elem_geom.html"
---

### Linked Element Geometry Access

I just arrived in Greece to give a Revit API training here during the next few days.
I am in a pleasant place called Κηφισια (Kifissia), a suburb of Athens.
I still need to get the training material set up properly...

Meanwhile, here are some notes on a different topic that I have been sitting on for a while:

Analysing the geometry of linked files is a challenge.

Certain basic information is actually easily accessible, as shown by the discussion on
[listing elements in linked documents](http://thebuildingcoder.typepad.com/blog/2009/02/list-linked-elements.html) and the
[CmdLinkedFileElements](http://thebuildingcoder.typepad.com/blog/2009/02/list-linked-elements.html) command.

More intense geometric analysis is not so easy, however.
For instance, the FindReferencesWithContextByDirection method does not work on linked elements, which is understandable, and even the Intersect method will fail when fed geometry from linked elements.

The Revit 2012 API introduces the new Instance.GetTransform and Instance.GetTotalTransform methods which help handle this.
These new methods provide the transform for a given Instance (which is the parent class for elements like family instances, link instances, and imported CAD content).
GetTransform obtains the basic transform for the instance based on how the instance is placed.
GetTotalTransform provides the transform modified with the true north transform, for instances like import instances.

The transform obtained can be applied to geometry elements using the new GeometryElement.GetTransformed method, which returns a copy of the geometry in the original element, transformed by the input coordinate transformation.

We had a look at this method recently to analyse
[getting the transformed family instance geometry](http://thebuildingcoder.typepad.com/blog/2011/06/get-transformed-family-instance-geometry.html) and
taking into account the transformation already built into the GetTransformed method.

Here is an explanation of some further aspects of this topic:

**Question:** I just don't understand why you say that geometry of the linked elements cannot be obtained.
When walls, floors etc. are obtained from a linked document, it seems that I'm able to also obtain their geometry.
Obviously it would not even be possible to call the Intersect method without obtaining the geometry first.
The problem is that result is incorrect.
Why is it that the linked geometry appears to be OK but still is somehow incomplete?

**Answer:** The geometry of the linked document is not necessarily the same as the geometry of the document itself.
When you get the elements from the document, the geometry does not reflect (for example) the transformation of the linked document into the coordinate system of the host document.
You can now get the transformation into the current project using the inherited method Instance.GetTransform on the link instance, and the transformed geometry of an element using GeometryElement.GetTransformed.

Hopefully, by applying the appropriate transform retrieved from the link instance to the target curve, e.g. using Curve.Transformed, you will then be able to correctly detect the intersection.

There are other potential differences regarding filters applied by the host document on the link which would not necessarily be reflected in the geometry of the document which is linked in. So when we say that the geometry of the link is not truly available this is accurate; the geometry you can find from the document in memory doesn't know how the host modifies the link.