---
post_number: "1043"
title: "Move Tag to Host Location"
slug: "move_tag_to_host"
author: "Jeremy Tammik"
tags: ['csharp', 'elements', 'revit-api', 'views']
source_file: "1043_move_tag_to_host.htm"
original_url: "https://thebuildingcoder.typepad.com/blog/1043_move_tag_to_host.html"
---

### Move Tag to Host Location

After a number of pretty lengthy heavy-duty discussions in the past few days, here is a short and quick summary of a useful result from the discussion forum thread on
[moving a tag to its host location point](http://forums.autodesk.com/t5/Autodesk-Revit-API/Move-tag-to-host-location-point/td-p/4440217):

**Question:** I'm trying to move a tag to the host location point.
Can anyone show me what the syntax is to do this?
I already have 'tag' and 'hostLocation' assigned.
I've tried this but get an error:

```csharp
tag.Location = new LocationPoint(
hostLocation.Point );
```

**Answer:** You can use the ElementTransformUtils.MoveElement or Location.Move methods instead.

The trick though is knowing how far to move it as the tag's Location property cannot be cast to either LocationPoint or LocationLine. You instead need to use the point returned by the TagHeadPosition property for the location of the tag.

Then you can do something like this:

```csharp
tag.Location.Move( hostLocation
- tag.TagHeadPosition )
```

Thanks to [sdwil2k5](http://forums.autodesk.com/t5/user/viewprofilepage/user-id/1236248) for
confirming and clarifying this.