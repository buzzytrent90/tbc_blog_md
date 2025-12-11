---
ai_optimized: true
author: Jeremy Tammik
complexity_score: 2.3
content_type: qa
optimization_date: '2025-12-11T11:44:14.869014'
original_url: https://thebuildingcoder.typepad.com/blog/0910_purge.html
post_number: 0910
reading_time_minutes: 4
series: general
slug: purge
source_file: 0910_purge.htm
tags:
- doors
- elements
- family
- filtering
- parameters
- references
- revit-api
- transactions
- walls
- windows
title: Determining Purgeable Elements
word_count: 848
---

### Determining Purgeable Elements

A question that crops up from time to time and is rather hard to answer exactly and exhaustively is on purgeable elements:

**Question:** What is the best way to get all of the elements that can be purged?
I can use WhereElementIsElementType filter to return all types, and WhereElementISNotElementType to return instances, but comparing the two lists does not leave me with the purgeable result. I could compare individual lists of types (i.e. family symbol and family instance), but that seems prone to error. Is there any good way to get this information out of the model?

**Answer:** Yes, the Revit API does not currently provide any direct method to determine this.

Let's see what can be done about it anyway already.

When you say "all of the elements that can be purged", that can be a pretty huge task.

I would suggest breaking that down into individual sub-tasks for different types of purgeable elements, which will require pretty different handling.

For example, I implemented a method DeleteUnnamedNonHostingReferencePlanes to find and
[delete all reference planes not hosting any elements](http://thebuildingcoder.typepad.com/blog/2012/03/melbourne-devlab.html#2).

An explicit reference to the term purge is used in the post on
[removing unused text note types](http://thebuildingcoder.typepad.com/blog/2010/11/purge-unused-text-note-types.html).

One way to determine whether a certain element A is required by another one B is to delete A and check the return value of the Delete method, which provides a list of deleted element ids. In some cases, B is deleted as well, and its element id is included the resulting list of deleted element ids. This is used by the
[object relationship analyser](http://thebuildingcoder.typepad.com/blog/2010/03/object-relationships.html)
([VB](http://thebuildingcoder.typepad.com/blog/2010/03/object-relationships-in-vb.html)).

The deletion can be made temporary by encapsulating it in a transaction that is later rolled back. Here is a discussion of
[the temporary transaction trick](http://thebuildingcoder.typepad.com/blog/2012/10/the-temporary-transaction-trick-for-gross-slab-data.html) including
a list of numerous usage examples, and later
[touched up slightly](http://thebuildingcoder.typepad.com/blog/2012/11/temporary-transaction-trick-touchup.html).

Unfortunately, of course, this method becomes extremely slow in a large model with each deletion taking place in its own transaction and then rolled back.
Wrapping it all in one single transaction will essentially deleting the entire model, which causes other problems is also not very useful, so this idea may not really be very feasible for the task at hand.

When you do have access to a one-way relationship between two classes of objects, e.g. pointers from doors, windows and openings hosted in walls to their respective wall host elements, you can easily invert the relationship to determine the list of hosted object for each host. This is demonstrated in my discussion of the
[relationship inverter](http://thebuildingcoder.typepad.com/blog/2008/10/relationship-in.html) in
one of the very first blog posts.

Taking a look at family types only, which is a small subset of the potentially purgeable elements, a similar one-way relationship as the one between hosted objects and their hosts is defined between non-element types and types, since the former provide a property named GetTypeId pointing to the type. You could make use of this in a similar manner to find all unused types as follows:

1. Determine all non-type elements using WhereElementISNotElementType- Iterate over all non-type elements. Determine the element id of each one, and build a dictionary D of the inverse relationship, mapping type element id to a list or the elements using that type.- Use WhereElementIsElementType to determine all element types. Iterate over them. For each type, check D to see how many elements are referencing it. If D has no entry for this type, it is not being used and can be deleted.

There is no guarantee that this will really work, though, since there may be any number of exceptional situations that need to be handled differently on a case-by-case base.

You can try it out, though, see whether you corrupt the database in any way, test thoroughly, possibly add handling of all special cases that you encounter, and maybe achieve your goal in the long run.

Unfortunately, there is currently no API access to the list of objects or categories searched by the built-in purge functionality, and the logic used for it is non-trivial.

Definitely, a number of special cases need to be handled, and there are some things that cannot be determined using the method described above.

For example, there are no instance placements of materials, so you have to search through instances placed in the model and for each element determine whether the material is used by it or not.
This also leaves out materials that are set for type parameters on types that have not been used but are loaded into the model.