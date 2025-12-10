---
post_number: "1230"
title: "Attached Detail Groups and Inverse Relationships"
slug: "attached_detail_group"
author: "Jeremy Tammik"
tags: ['doors', 'elements', 'parameters', 'revit-api', 'walls', 'windows']
source_file: "1230_attached_detail_group.htm"
original_url: "https://thebuildingcoder.typepad.com/blog/1230_attached_detail_group.html"
---

### Attached Detail Groups and Inverse Relationships

I am still at the
[Berlin hackathon](https://twitter.com/hashtag/tmuhack),
working with the [MovieMemory](http://m3my.github.io) team...

In between other tasks, let me mention the interesting discussion and solution by Revitalizer on the Revit API discussion forum to
[check for attached detail groups](http://forums.autodesk.com/t5/revit-api/check-for-attached-detail-group/m-p/5335867):

**Question:** Can someone tell me, please, how I can find out if a Group (Type) has an Attached Detail Group (Type) via the API?

**Answer:** A group of category OST\_IOSAttachedDetailGroups has a BuiltInParameter.GROUP\_ATTACHED\_PARENT\_NAME.

This way, you will find matching Model groups.

**Response:** Thanks for the answer.

So this would mean that instead of just asking a group Abc for its Attached Detail Groups, I'd have to go over all groups of the right category in the project and check if their parent is group Abc... right?

Well I guess I could do that – it just seems a bit complicated.
I mean, how come an element knows its parent but not its children, and not even whether it has any children in the first place, which is all I actually need to know, in my case?

**Answer:** I usually create Dictionaries to store relationships like these so I can access them quickly when needed.

Another point to note:

As you can see, this parameter is of type string; I wonder why it stores a name instead of an element id.

I don't know why...

Many relationships and tips and tricks like this are not officially documented in the Revit API, and require exploration and understanding of the product usage to discover.

Regarding the storage of parent-children relationships:

The approach described above is the standard way of storing such relationships in Revit.

For instance, a door or window hosted by a wall maintains a pointer to the hosting wall via the Element.Host property.

The inverse relationship is not available directly from the API, and probably not stored anywhere at all, since it can always be calculated and saved in a temporary dictionary when needed.

One of the very early discussions on The Building Coder discusses the issue of implementing such a dictionary to
[determine the inverse relationship from host to hosted elements](http://thebuildingcoder.typepad.com/blog/2008/10/relationship-in.html).

In fact, that is blog post number 16 from 2008 :-)

It makes sense for the Revit model to store the relationship one way only, not both ways, to minimise the model data, possibilities for errors and synchronisation issues.

I hope this helps explain.

**Response:** Good question indeed...

Once again thanks for the answer!   :-)