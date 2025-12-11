---
ai_optimized: true
author: Jeremy Tammik
complexity_score: 3.8
content_type: code_example
optimization_date: '2025-12-11T11:44:14.010717'
original_url: https://thebuildingcoder.typepad.com/blog/0475_place_furniture.html
post_number: '0475'
reading_time_minutes: 4
series: general
slug: place_furniture
source_file: 0475_place_furniture.htm
tags:
- csharp
- elements
- family
- levels
- parameters
- references
- revit-api
- rooms
- walls
- windows
title: Place Furniture Instance
word_count: 882
---

### Place Furniture Instance

The discussion on placing family instances and proper usage of the various overloads of the NewFamilyInstance method continues.
After looking at the use of
[PromptForFamilyInstancePlacement](http://thebuildingcoder.typepad.com/blog/2010/06/place-family-instance.html), placing
[detail instances in 2D](http://thebuildingcoder.typepad.com/blog/2010/10/place-detail-instance.html), and
[site components on a topo surface](http://thebuildingcoder.typepad.com/blog/2010/11/place-detail-instance.html),
we now turn to the everyday task of placing a furniture family instance, e.g. a chair into a room.

This
[furniture insertion question](http://thebuildingcoder.typepad.com/blog/2009/02/inserting-a-beam.html?cid=6a00e553e168978833013488b40382970c#comment-6a00e553e168978833013488b40382970c) was
raised by Kristoffer Kvello of
[Selvaag](http://www.selvaag.no/Sider/default.aspx) in Norway.
He also asks about the reference direction, i.e. the rotation of the family instance, and ends up providing the answer for himself:

**Question:** I'm struggling with inserting a furniture component in my Revit 2011 model.

What I want to do is to insert a piece of furniture (a bed) onto the first floor (i.e., the first with elevation greater than zero).
The Revit 2011 API Developer Guide.pdf says on page 134 that "Some FamilyInstance objects do not have host elements, such as tables and other furniture".
However, an example of "Insert(ing) a new instance of a family into the document" in the Revit 2011 API help doc uses this call:
```csharp
  FamilyInstance instance
    = doc.Create.NewFamilyInstance(
      location, symbol, direction, floor,
      StructuralType.NonStructural );
```

This seems to be exactly what I want (the furniture in the example is even a "bedbox"), so I tried that.
I have ensured that both the location (including the Z-coordinate) and the floor are correct.
But the bed is nevertheless located on the zeroeth floor.
In the Property dialogue I see that the bed has been assigned a host, which is a floor, but not a level.
Confusingly, when I inspect the bed using the Snoop tool, the host property as shown there is null.
If I set a break point after the bed has been created and check its host property in the Immediate Window, it is also null.

When I place the bed manually, I see that it has both a host (although the Snoop tool still says it hasn't) and also a level.
The manually placed bed is on the correct floor, so my next thought was that I should use an overload that includes the Level argument.
In the guide-to-placing-family-instances-with-the-api.doc document I found an overload appropriate for furniture with this comment: "If it is to be hosted on a wall, floor or ceiling and associated to a reference level" which sounded good, so I tried:
```csharp
  FamilyInstance revitBed =
    doc.Create.NewFamilyInstance(
      location,
      bedFamilySymbol,
      floorBedIsOn,
      levelBedIsOn,
      StructuralType.NonStructural );
```

Now the bed is located on the correct floor; the property dialogue says its host is a floor; and it also has a Level (which is the correct level), just as when I insert the bed manually.

However, this overload doesn't include the reference direction argument, and I have so far not succeeded in finding a way of rotating the bed after it has been created.

Clearly there are many more things to try out (yet more overloads, using a Rotate method post-creation, etc.) but I suspect that there might be something not quite right here, and that the first overload I attempted "should have worked".
Could you throw some light upon this?
Have I misunderstood something?

**Answer:** Yes, I found a solution to my furniture problem.

I solved the problem by using the NewFamilyInstance overload that specifies both a host and a level:

- XYZ- FamilySymbol- Host Element- Level- StructuralType

I then rotate the furniture with the Document.Rotate method after it is created.
This works flawlessly, and this way, the property dialogue of the bed displays the same information when I put in the bed using the API as when I do it manually (i.e., both a host and a level) and the bed has the correct orientation.

I tried the overload with Level but no Host, and that seemed to work too, but the property dialogue in that case does not list a host (naturally), whereas it does when I position the bed manually.
So I think that the overload with both host and level is the one to use.

Regarding the Z-coordinate: I suspected that it possibly should be zero and I tried that as well with the first overload I thought should be correct (XYZ, FamilySymbol, XYZ (direction), Element (host), StructuralType) but I saw no difference in behaviour: the bed still ended up on the zeroeth floor, not the first.

Finally, I tried to set the offset parameter as you suggested (with location.Z = 0) just to see the effect, but with little luck: the bed seems to be positioned right at the top of the floor it's on and in the UI I cannot find the "Offset" parameter, neither on the instance nor the type. I do find it with the Snoop tool however, but the value is zero despite me setting it to 6.5.

Many thanks to Kristoffer for sharing these valuable results!