---
ai_optimized: true
author: Jeremy Tammik
complexity_score: 3.0
content_type: qa
optimization_date: '2025-12-11T11:44:15.582813'
original_url: https://thebuildingcoder.typepad.com/blog/1239_directshape.html
post_number: '1239'
reading_time_minutes: 5
series: general
slug: directshape
source_file: 1239_directshape.htm
tags:
- csharp
- elements
- family
- filtering
- geometry
- levels
- parameters
- revit-api
- schedules
title: DirectShape versus Families, Category and Texture
word_count: 992
---

### DirectShape versus Families, Category and Texture

Let's begin this week with a discussion of several aspects of direct shapes:

- [Direct shapes, part families and application porting](#2)
- [DirectShape categories](#3)
- [DirectShape texture assignment](#4)

#### Direct Shapes, Part Families and Application Porting

**Question:** I am looking for some guidance related to porting my application to Revit.

I have a parametric part engine that lives outside of Revit with a large set of existing library content.

It would be very costly to rebuild from scratch using the Revit Part Family editor and framework, so I'm trying to link it into Revit by alternative methods.

When the DirectShape and TessellatedShapeBuilder API was added, I thought it would be the ideal solution. With minimal effort, I was able to convert and import my part geometry into a Revit project. But I quickly realized that wasn't quite the right solution. My geometry is typically just an open mesh, not a solid. I found that I cannot dimension to the faces of the DirectShape mesh result, which I know users would want to do.

I also would prefer to programmatically create part families so my parts look and behave as much like other part families, with shared parameters, etc. But I found that I cannot add a DirectShape result into a part family definition... is that correct? It actually lets me put it in there, but it doesn't appear when I place a family instance.

This led me to look into other types of geometry to use. I tried a Form created using NewFormByCap which I found could only be added to a massing type of element, which I didn't think sounded right for my solution, along with the fact that I couldn't get it to display. I think Generic Model seemed more appropriate.

I have my UI framework in place, reacting to parameter change events in a way that I can update and regenerate the geometry as necessary to get my own parametric engine working... I'm just trying to fill in a few missing pieces to the puzzle, making the best choices for the geometry in my part families.

**Answer:** Porting an existing application to Revit is often a challenging undertaking, especially because the underlying philosophical paradigms and user workflows in
[Revit differ substantially](http://thebuildingcoder.typepad.com/blog/about-the-author.html#5.41) from
what we application developers coming from other CAS systems are normally used to.

The DirectShape API was mainly developed to support certain workflows and
[round trip functionality for IFC files](http://thebuildingcoder.typepad.com/blog/2014/05/directshape-performance-and-minimum-size.html).
For that, it is indeed an effective, simple and performant solution:

For defining your own part families, a normal family would normally be preferable and provide more functionality, just as you have discovered.

I am not a content creation expert, so I cannot say for sure what the optimal way to define a suitable family for your purposes might be, nor have you stated what they are. However, the Revit family system does provide support for all imaginable building components, so I am pretty sure that all you need can be found in there.

I would suggest that you continue researching in that direction and pretend for just a moment that you wanted to create the families you need manually. Discuss in depth with content creation experts what the best structure and workflows would be for both the manual definition and subsequent use of these families.

Once you have a reliable optimal solution for a manual workflow in place, I would next explore how to drive that process programmatically and hook it up with your own parametric system.

If in doubt, a generic model sounds like a good starting point to me as well. I am also pretty sure you can add whatever you need to such a family.

With that said, the direct shape functionality was first introduced in Revit 2015, and we are still looking at ways to enhance and round it off for all required situations.

#### DirectShape Categories

**Question:** I would like to create DirectShape elements to represent different types of beams and put them in different Categories. Until now, however, I can only create them using built-in categories like this:

```csharp
  ElementId id = new ElementId(
    BuiltInCategory.OST\_StructuralFraming );

  DirectShape ds = DirectShape.CreateElement(
    doc, id, "A", "B" );
```

Can I create a custom category and use that as well, e.g. like this:

```csharp
  Category cat = doc.Settings.Categories.get\_Item(
    BuiltInCategory.OST\_StructuralFraming );

  Category subCat = doc.Settings.Categories
    .NewSubcategory( cat, "Sparren" );

  ElementId elemIdSubCat = subCat.Id;

  DirectShape ds = DirectShape.CreateElement(
    doc, elemIdSubCat, "A", "B" );
```

When I try this, the DirectShape creation raises an error saying the id must be a top-level category.

**Answer:** All I can do, really, is to summarise and confirm what you say above:

1. You can create subcategories. You cannot create new top-level categories.
2. The direct shape creation requires a top-level category.

I think that just about answers your question, doesn't it?

I submitted the wish list item CF-1748 [API wish: DirectShape subcategory support] requesting direct shape subcategory support for you.

As an alternate technique that you can implement and use right away, you could consider creating one or more shared parameters and use those for differentiation in filters and schedules. From Revit 2015 onwards, such shared parameters can even be set to be read-only.

#### DirectShape Texture Assignment

**Question:** Is it possible to assign a texture (material) to a DirectShape object?

**Answer:** Materials can indeed be assigned to the faces of DirectShape solids.

One way to do this is through SolidOptions.MaterialId, which is available for every member of GeometryCreationUtilities.

Another way is through the TessellatedFace.MaterialId property that is available for faces used to populate the TessellatedShapeBuilder.

The open source
[IFC link project](http://sourceforge.net/projects/ifcexporter) provides
some live examples of this in use.