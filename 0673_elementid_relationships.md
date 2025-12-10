---
post_number: "0673"
title: "ElementId Relationships"
slug: "elementid_relationships"
author: "Jeremy Tammik"
tags: ['elements', 'filtering', 'levels', 'parameters', 'revit-api', 'views']
source_file: "0673_elementid_relationships.htm"
original_url: "https://thebuildingcoder.typepad.com/blog/0673_elementid_relationships.html"
---

### ElementId Relationships

I recently looked at determining the
[coordinates of a filled region](http://thebuildingcoder.typepad.com/blog/2011/09/filledregion-corrdinates.html) by
looking at the sketch related to it, which we can identify by using an unsupported feature of the current Revit implementation, the generation of consecutive element ids for related elements.

Rudolf Honke of
[Mensch und Maschine acadGraph GmbH](http://www.acadgraph.de) pointed
out that this approach can be used to determine other relationships as well, and brought up a whole bunch of ideas related to this topic.
He says:

The sketch of a stair for example can also be found by 'stair.Id - 1', so we can access this type's generating curves, too.

Did you know that you also can access a section view symbol by decreasing the ViewSection.Id.IntegerValue by 1?
I don't know if this is always the case, but in my files, the symbol for the section, i.e. the visible element that you use for manipulating the view itself, always has this relation to the according view.

In this case, 'symbol' means the 'viewer' or the 'viewPort', not the ElementType.
'Viewer' versus 'view'.

This viewer-to-view relation is just an observation.
Apparently there are some other elements paired with each other that can also be identified in this way.

This opens up lots of possibilities to explore the viewer-view theme in more depth.

Thinking of Parameters that are 'shared' between viewer and view.
They will react to each other, but how are they 'mapped' to each other?
Or think of a Parameter that is read-only in views but settable in viewers (but Parameters may have different names).

As mentioned looking at the
[filled region coordinates](http://thebuildingcoder.typepad.com/blog/2011/09/filledregion-corrdinates.html),
the difference between the paired elements may be greater than 1.
So one has to scan the nearer neighborhood, in fact.

In case of the FilledRegion/Sketch pair, we know that the type of the paired element is a Sketch.
In the case of View/Viewer, we must filter the nearby elements by category, which is Ost\_Viewers because the Viewers are just elements.

I searched for some correlating parameters between them, but the only link between them seems to be their similar element ids.

Another thought on the stairs theme:
if we can access the lines of the stair sketch, then perhaps we can edit them as well and and thus edit the shape of a whole stair programmatically?

These thoughts are just ideas.
It would be interesting to explore deeper into these topics.
Any takers?

Obviously these kind of explorations make use of completely unsdocumented and unsupported features.
Use of such features is completely at your own risk, and the behaviour may and will almost certainly change with no notice whatsoever at some point.

**Addendum by Arnošt Löbel**, Sr. Principal Engineer in the Revit development team: I find the ideas presented in this article slightly disturbing especially if it was meant as suggestions for any production-level code. One thing is to poke around and look at the IDs to get some ideas how Revit put parts together or just for one's amusement. Another thing all together is using any of such discovered relations and assumed dependencies between Ids in actual production code. Please be aware that Revit (and Autodesk Revit developers) does not guarantee any fixed schema around element Ids. Assignments of Ids can change and possibly changes between versions and even between minor updates. Also, Ids are not constant – they can change and definitely would change in work-sharing environment.

I cannot agree more with Jeremy's wise advice: 'If you wish to have particular features in the Revit API, submit a 'wish' through Revit customer support.' Revit R&D takes all requests very seriously and even though they cannot possibly satisfy all, they do their best to implement at least the 'hottest' wish-list items every release.