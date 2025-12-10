---
post_number: "0802"
title: "View of NewGroup Duplicated Elements"
slug: "new_group_view"
author: "Jeremy Tammik"
tags: ['csharp', 'elements', 'revit-api', 'views']
source_file: "0802_new_group_view.htm"
original_url: "https://thebuildingcoder.typepad.com/blog/0802_new_group_view.html"
---

### View of NewGroup Duplicated Elements

I already mentioned that you can work around the lack of programmatic creation possibilities for some element types by grouping existing ones, creating new instances of that group, and then ungrouping it again.

One question that came up in that context is what view the new group appears in:

**Question:** I am using code like this to duplicate one or more elements:
```csharp
  void CreateGroup(
    Document doc,
    ICollection<ElementId> ids,
    XYZ offset )
  {
    Group group = doc.Create.NewGroup( ids );

    LocationPoint location = group.Location
      as LocationPoint;

    XYZ p = location.Point + offset;

    Group newGroup = doc.Create.PlaceGroup(
      p, group.GroupType );

    group.UngroupMembers();
  }
```

However, under certain circumstances, the new group is not placed in the same view as the old one.

Is there any way to specify the target view?

**Answer:** Sorry, there is currently no direct API access to control the view used when placing a group.

The active view will be used to place the new group, so all you have to do is ensure that the target view really is activated.

To be on the really safe side, you might want to try closing all other views to absolutely force the open one to be used.

The Revit API calls should in theory obviously not be influenced by any of the user interface settings, but there may still be some obsolete remnants of such dependencies left.