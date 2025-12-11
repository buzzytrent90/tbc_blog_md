---
ai_optimized: true
author: Jeremy Tammik
complexity_score: 1.7
content_type: qa
optimization_date: '2025-12-11T11:44:15.459886'
original_url: https://thebuildingcoder.typepad.com/blog/1177_ifc_guid_fami_modi.html
post_number: '1177'
reading_time_minutes: 2
series: general
slug: ifc_guid_fami_modi
source_file: 1177_ifc_guid_fami_modi.htm
tags:
- csharp
- family
- filtering
- parameters
- revit-api
title: IFC GUID Algorithm Update and Family Modification
word_count: 484
---

### IFC GUID Algorithm Update and Family Modification

Two short topics for today, an important little
[IFC GUID algorithm bug fix](#2) and
some thoughts on
[detecting family modification](#3).

#### IFC GUID Algorithm in C# Update

Håkon Clausen implemented a bug fix update and created a GitHub repository for the
[IFC GUID algorithm in C#](http://thebuildingcoder.typepad.com/blog/2010/06/ifc-guid-algorithm-in-c.html) that
he originally published back in June 2010.

Here are the
[details on the update](http://thebuildingcoder.typepad.com/blog/2010/06/ifc-guid-algorithm-in-c.html#2)
and a direct link to his
[IfcGuid GitHub repository](https://github.com/hakonhc/IfcGuid).

#### Detecting Family Modification

For ages, content creators and developers have been asking how to prevent users from modifying their families to protect their IP, or at least for foolproof ways to detect such changes.

Here is a typical query:

**Question:** I keep being asked how the user can be prevented from tinkering with my components.

I cannot and will not implement any such barrier, since the resulting system would be much too rigid.

However, it would be very helpful if I could detect whether any changes have been made to the family.

For instance, a parameter value could be set to zero as soon as a family is copied or modified.

That would allow my add-in or a model tester to define a filter to see which components were manually created and need additional attention.

**Answer:** You really need to discuss this with a content creation expert.

I do believe there are some tricks that I am not properly aware of, e.g. placing parameters in nested families and thus restricting access to them.

The Revit 2015 API provides one important new feature in this area, read-only parameters that can only be set programmatically and are not modifiable manually at all.
For more on this, look at the section
[shared parameter creation – description and user modifiability](http://thebuildingcoder.typepad.com/blog/2014/04/whats-new-in-the-revit-2015-api.html#2.04).

Finally, the following cool idea comes to mind spontaneously:

- Define and implement a standard method to create a binary lump of some kind from a family definition. For instance, you could use the result of calling the EditFamily method on it and then saving to an external file.
- Calculate a checksum for the binary lump.
- Save the checksum value in a read-only parameter or in extensible storage, possibly in encrypted form.

From now on, every time you encounter an instance of this family, you can recalculate the current checksum value that it produces and compare it with the stored original value.

I would hope that a proper implementation of this would enable you to detect changes in the way you wish.

I would appreciate hearing from you, dear reader, about other suggestions and solutions for this.

Thank you!