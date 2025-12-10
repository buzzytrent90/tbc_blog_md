---
post_number: "2001"
title: "Boundingbox"
slug: "boundingbox"
author: "Jeremy Tammik"
tags: ['elements', 'family', 'filtering', 'parameters', 'revit-api', 'rooms', 'sheets', 'views', 'walls']
source_file: "2001_boundingbox.md"
original_url: "https://thebuildingcoder.typepad.com/blog/2001_boundingbox.html"
---

### Bounding Boxes Axis Alignment and Transformation
Today we look at some aspects axis-aligned bounding boxes, bounding box transformation and AI news:
- [BoundingBox is axis-aligned](#3)
- [BoundingBox transformation](#4)
- [Interactive explanation of SVG path](#5)
- [Claude.AI](#6)
- [Relativising the impact of AI](#7)
#### BoundingBox is Axis-Aligned
Let's move on to the more mundane question
of [how to get the `BoundingBox` that corresponds to the shape of the family](https://forums.autodesk.com/t5/revit-api-forum/how-to-get-the-boundingbox-that-corresponds-to-the-shape-of-the/m-p/12089970):
\*\*Question:\*\* I would like to inquire about `BoundingBox`.
In the left image, the area of the BoundingBox is similar to the shape of the pipe;
however, in the right one, the BoundingBox is much larger than its pipe:
![Pipe bounding box](img/pipe_boundingbox.png "Pipe bounding box")
It seems that this happens only for Pipes (probably for System Families).
For FamilyInstance, it seems that they always have a same size for BoundingBox, even if they are rotated.
Is there a way to know the area of the Family, which always corresponds to its shape, regardless of it being rotated or not?
\*\*Answer:\*\* No, this does not only happen for pipes.
This happens for all kinds of objects that have different dimensions in different directions and are placed at an angle to the cardinal axes.
For instance, a vertical wall at a 45-degree angle in the XY plane will exhibit the exact same behaviour.
This behaviour is intentional.
The Revit bounding box is always aligned with the cardinal axes.
That makes it extremely fast and efficient to work with.
Wikipedia calls this
an [axis-aligned bounding box](https://en.wikipedia.org/wiki/Minimum_bounding_box#Axis-aligned_minimum_bounding_box).
#### BoundingBox Transformation
That question leads over nicely to the explanation why
the [transform of linked element creates an empty outline](https://forums.autodesk.com/t5/revit-api-forum/transform-of-linked-element-creates-an-empty-outline/m-p/12089717),
solved by appropriately transforming the bounding box:
\*\*Question:\*\* I want to create an outline from a linked element's bounding box to use in a `BoundingBoxIntersectsFilter`.
My approach below works for some cases, but where a link or element is rotated beyond a certain limit, the bounding box goes askew and the `Outline(XYZ, XYZ)` method returns an empty outline.
I've tried scaling the bounding box up with an offset which fixes some cases, but not all of them.
Some advice on solutions would be appreciated.
![Non-rotated link](img/xform_boundingbox_1.png "Non-rotated link")

Non-rotated link

![Rotated link](img/xform_boundingbox_2.png "Rotated link")

Rotated link

```
# get the elements bounding box
s_BBox = element.get_BoundingBox(doc.ActiveView)

# apply the link documents transform
s_BBox_min = link_trans.OfPoint(s_BBox.Min)
s_BBox_max = link_trans.OfPoint(s_BBox.Max)

# make the outline
new_outline = Outline(s_BBox_min, s_BBox_max)

# make the filter
bb_filter = BoundingBoxIntersectsFilter(new_outline)
```

\*\*Answer:\*\* Yes.
You can corrupt the bounding box by transforming it.
I would suggest the following:
- Extract all eight corner vertices of the bounding box
- Transform all eight vertices as individual points
- Create a new bounding box from the eight transformed results
[The Building Coder samples](https://github.com/jeremytammik/the_building_coder_samples/tree/master) include
the [method `ExpandToContain`](https://github.com/jeremytammik/the_building_coder_samples/blob/master/BuildingCoder/Util.cs#L2724-L2739) to
create and enlarge a bounding box point by point that will come in handy for the last step:

```
  ///
  ///   Expand the given bounding box to include
  ///   and contain the given point.
  ///
  public static void ExpandToContain(
    this BoundingBoxXYZ bb,
    XYZ p)
  {
    bb.Min = new XYZ(Math.Min(bb.Min.X, p.X),
      Math.Min(bb.Min.Y, p.Y),
      Math.Min(bb.Min.Z, p.Z));

    bb.Max = new XYZ(Math.Max(bb.Max.X, p.X),
      Math.Max(bb.Max.Y, p.Y),
      Math.Max(bb.Max.Z, p.Z));
  }
```

\*\*Response:\*\* Hi Jeremy, this solution works well, thanks.
#### Interactive Explanation of SVG Path
I made good use of and learned to love SVG paths working on
the [room polygon and furniture picker in SVG](http://thebuildingcoder.typepad.com/blog/2012/10/room-polygon-and-furniture-picker-in-svg.html) and
implementing [2D SVG editing on mobile device with Raphael](http://thebuildingcoder.typepad.com/blog/2013/02/2d-svg-editing-on-mobile-device-with-rapha%C3%ABl.html).
If you would like to enjoy a much nicer explanation of the concepts of SVG paths than I had access to back then, take a quick dive
into [understanding SVG paths](https://www.nan.fyi/svg-paths).
#### Claude.AI
ChatGPT has a new competitor,
[claude.ai](https://claude.ai).
Eric Boehlke of [truevis BIM Consulting](https://truevis.com) pointed it out to me, saying:
> You can create an account in the USA or UK, or use VPN to get there.
A competitor to OpenAI. Can upload big files. Any better for coding?
I tested it myself by asking it to summarise
my [last blog post](https://thebuildingcoder.typepad.com/blog/2023/07/rbp-materials-their-assets-and-the-visual-api.html),
evaluate a German architectural contract without providing any hint of what it might be, and discuss
the German [HOAI](https://de.wikipedia.org/wiki/Honorarordnung_f%C3%BCr_Architekten_und_Ingenieure).
The results were completely satisfying in all three cases:
![Claude.AI analyses blog post](img/claudeai_tbc2000.png "Claude.AI analyses blog post")

Claude.AI analyses blog post

![Claude.AI analyses a German architectural contract](img/claudeai_archvertrag.png "Claude.AI analyses a German architectural contract")

Claude.AI analyses a German architectural contract

![Claude.AI discusses the HOAI](img/claudeai_hoai.png "Claude.AI discusses the HOAI")

Claude.AI discusses the HOAI

Eric suggests:
> You could batch that blog post analysis to summarise all of your blog posts programmatically for SEO.
#### Relativising the Impact of AI
Let's close with some wise and pertinent thoughts – not following the hype – on the possible impact of AI on our life and economy, presented in the explanation
of [why transformative AI is really, really hard to achieve](https://zhengdongwang.com/2023/06/27/why-transformative-ai-is-really-really-hard-to-achieve.html).