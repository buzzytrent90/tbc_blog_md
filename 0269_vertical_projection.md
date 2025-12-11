---
ai_optimized: true
author: Jeremy Tammik
complexity_score: 2.5
content_type: qa
optimization_date: '2025-12-11T11:44:13.652088'
original_url: https://thebuildingcoder.typepad.com/blog/0269_vertical_projection.html
post_number: 0269
reading_time_minutes: 3
series: views
slug: vertical_projection
source_file: 0269_vertical_projection.htm
tags:
- csharp
- elements
- parameters
- revit-api
- schedules
- views
title: Vertical Projection of Beam Analytical Model
word_count: 548
---

### Vertical Projection of Beam Analytical Model

We tried to fly from Milano to Paris this evening for the last DevDays conference, but unfortunately our flight was cancelled due to snow in Paris.
Now we are stuck at the Milano airport and hoping that the first flight in the morning will not be cancelled as well.
If it is, we will not be able to make it at all.
Otherwise, we will arrive rather late and have to reschedule the agenda, but at least then we will not be forced to cancel the whole conference.
Fervently hoping for the best...

Since I am not sleeping anyway, I may as well post.

First, here are some more titbits from AU:

Kean posted an
[AU wrap-up](http://through-the-interface.typepad.com/through_the_interface/2009/12/post-au-wrap-up.html) with
a short video and some snapshots from various Autodesk University experiences.

As part of AUv, Jim Quanci gave a very informative
[virtual interview](http://www.youtube.com/AutodeskUniversity#p/u/11/4c9RPYBOBj4)
on the
[Autodesk Developer Network ADN](http://www.autodesk.com/joinadn)
which is the part of Autodesk that provides developer support and that I work in.
It has now been posted on the AU YouTube channel and...

- Provides an overview of and helps better understand the ADN.- Shows the wide variety of applications being created by our partners.- Describes the benefits of being a member of the network.- Raises interest in ADN and our partners.

Getting back to the Revit API, here is a recent case handled by my colleagues Joe Ye and Saikat Bhattacharya which shows how to handle the setting of the vertical projection of the analytical model of a beam.
An interesting aspect of this is that the parameter data type is an element id, and yet we are actually storing an enumeration value in it.
This is not uncommon in Revit parameters; there are a few of them that can take both element ids to refer to other database objects as well as a fixed set of enumeration values for specific exceptional cases.

**Question:** How can I set the 'Vertical Projection' parameter of a beam to the 'Center of Beam' value?"

**Answer:** Here is a description on how to set the vertical projection parameter of a beam.

Start by defining the following enumeration:
```csharp
enum AnalyticalVerticalProjection\_e
{
  AVP\_AUTODETECT = -1,
  AVP\_TOP\_OF\_PHYSICAL = -2,
  AVP\_CENTER\_OF\_PHYSICAL = -3,
  AVP\_BOTTOM\_OF\_PHYSICAL = -4,
};
```

This can be used as follows:

- Construct an ElementId object, for instance, id.- Set id.Value to one of the enumeration member values.- Set the vertical projection parameter value to id.

Note that the parameter value can be set successfully to the value, even though it is not a real element id.

Here is a code snippet showing how to set value to top of the slab:
```csharp
Element e = SelectSlab( "Please select a slab",
  commandData, typeof( Floor ) );

if( null != e )
{
  Floor floor = e as Floor;

  BuiltInParameter bip = BuiltInParameter
    .STRUCTURAL\_ANALYTICAL\_PROJECT\_FLOOR\_PLANE;

  Parameter p = floor.get\_Parameter( bip );

  ElementId id = p.AsElementId();

  // id.Value is negative if the parameter
  // value is not a real element id:

  id.Value = -2; // top of the slab

  p.Set( ref id );
}
```

Thank you Joe and Saikat for this illuminating answer!