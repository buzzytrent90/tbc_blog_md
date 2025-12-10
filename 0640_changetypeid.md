---
post_number: "0640"
title: "Changing the Type of Many Instances"
slug: "changetypeid"
author: "Jeremy Tammik"
tags: ['elements', 'family', 'revit-api']
source_file: "0640_changetypeid.htm"
original_url: "https://thebuildingcoder.typepad.com/blog/0640_changetypeid.html"
---

### Changing the Type of Many Instances

Did you ever run into a performance problem changing the type of a large number of family instances?

Well, is you use the Element Symbol property or ChangeTypeId method to change them one by one, such as we did to set the type of a
[tag](http://thebuildingcoder.typepad.com/blog/2010/06/set-tag-type.html),
[elbow fitting](http://thebuildingcoder.typepad.com/blog/2011/02/set-elbow-fitting-type.html) or
[cable tray](http://thebuildingcoder.typepad.com/blog/2011/06/modifying-cable-tray-shape.html),
that may trigger a regeneration within Revit for each call, even if you are using manual regeneration mode and not explicitly asking for a regeneration yourself.
Changing a family instance type requires subsequent regeneration.

Bad news.

But hey, wait, there is good news as well!

You may not have noticed another overload of the Element.ChangeTypeId method.

The second overload is static, does not operate on only specific single element instance, and instead takes a whole collection of element instances to operate on.

And the good news is that that overload changes the type of all elements in the given set at once, with only one regeneration at the end.

Give it a whirl and let us know whether it helps.

**Addendum:** As Rudolf Honke points out, you can use the method Element.IsValidType taking the three arguments Document, ICollection elementIds, ElementId typeId to check beforehand whether all elements can accept the new element type. Thank you, Rudolf!