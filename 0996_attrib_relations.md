---
post_number: "0996"
title: "Attributes, Relationships and Other Stuff"
slug: "attrib_relations"
author: "Jeremy Tammik"
tags: ['csharp', 'elements', 'family', 'geometry', 'levels', 'revit-api', 'transactions', 'views', 'walls', 'windows']
source_file: "0996_attrib_relations.htm"
original_url: "https://thebuildingcoder.typepad.com/blog/0996_attrib_relations.html"
---

### Attributes, Relationships and Other Stuff

Before entering this wonderful new week, I have some unfinished business left over from the last one.

Last week I returned from a short holiday and was inundated with developer cases, some of which I would still like to report on, or the results will presumably be lost forever, or at least until the next person runs into the same issue.
I hope to prevent this here and now before starting with anything new.

Let us therefore take a quick look at the following topics:

- [View transparency setting](#2)
- [Determine BIM element masses](#3)
- [Stacked wall component relationships](#4)
- [Bind a link](#5)
- [Determine the colour of an element](#6)
- [Exception retrieving a bounding box](#7)
- [Happy week to all!](#8)

#### View Transparency Setting

**Question:** The graphic display options provide the following transparency setting:

![Graphic display options transparency setting](img/graphic_display_options_transparency.jpg)

How can I control that setting programmatically, please?

**Answer:** This is documented in the
[Revit 2014 API 'What's New'](http://thebuildingcoder.typepad.com/blog/2013/04/whats-new-in-the-revit-2014-api.html) section:

#### Views & Display

##### Graphic Display options

These new members expose read and write of graphic display options:

- View.GetBackground()
- View.SetBackground()
- View.ShadowIntensity
- View.SunlightIntensity
- View.SurfaceTransparency
- View.ShowEdges
- View.ShowSilhouettes
- View.SilhouetteLineStyleId

The View.SurfaceTransparency property provides an integer value controlling the amount of transparency applied to all surfaces, with 0 = opaque, 100 = fully transparent.

#### Determine BIM Element Mass

**Question:** I need to extract the masses of my various building elements and am having difficulties accessing the geometry information to achieve this.

I'm using a method similar to the ObjectViewer SDK sample.
In the project I am working on, this is only returning the roof and floor geometry.

What do I need to do to retrieve the window frame geometry as well?

Is there is a specific Mass class I can make use of?

**Answer:** You can query any Revit element for its geometry, extract the solids from that, and query them for their volume.
This should work just as well for windows and other family instances as for roofs and floors.

In some cases, you need to traverse the geometry instances as well as the main geometry object.

This is demonstrated by many samples, e.g. the
[OBJ viewer](http://thebuildingcoder.typepad.com/blog/2013/07/revit-2014-obj-exporter-and-new-sdk-samples.html#2).

However, it is probably simpler for you to use the Element methods GetMaterialIds, GetMaterialArea and GetMaterialVolume to determine the masses for different materials for any given element.
Look at the discussion of the
[material quantity extraction](http://thebuildingcoder.typepad.com/blog/2010/02/material-quantity-extraction.html) and
the MaterialQuantities SDK sample for an example of using this.

#### Stacked Wall Component Relationships

**Question:** How can I find the basic WallTypes composing a stacked wall?

I tried using the RevitLookup application and can see an unspecified "structure", but no further information.

**Answer:** I am glad you are using RevitLookup.
This important tool is a must-have for any Revit developer, and very useful for end users as well.

Currently, however, the Revit API provides no direct support for determining the relationship between a stacked wall and its subcomponent members.
There are a couple of workarounds, though.

One method to determine the relationship between a stacked wall and it basic wall components is described in the blog post on
[curtain wall geometry](http://thebuildingcoder.typepad.com/blog/2010/05/curtain-wall-geometry.html),
using
[undocumented sequential element id relationships](http://thebuildingcoder.typepad.com/blog/2011/11/undocumented-elementid-relationships.html).

A more reliable and useful method to determine this relationship is to delete the stacked wall and see which child walls go with it, a method used in a totally general way by the
[object relationship analyser](http://thebuildingcoder.typepad.com/blog/2010/03/object-relationships.html)
([VB](http://thebuildingcoder.typepad.com/blog/2010/03/object-relationships-in-vb.html)).

Pointers to several uses of that utility and the associated
[temporary transaction trick](http://thebuildingcoder.typepad.com/blog/2012/11/temporary-transaction-trick-touchup.html) are
provided in this discussion on
[purging elements](http://thebuildingcoder.typepad.com/blog/2013/03/determining-purgeable-elements.html).

#### Bind Link

**Question:** Is it possible to use the Revit API to "Bind Link", like from the user interface?

I searched for a method to achieve this, but with no luck.

**Answer:** I am not aware of any direct API support for this functionality.

However, I can imagine two different approaches to achieving it anyway.

- Make use of the existing built-in 'bind link' Revit command;
  you can check whether this command can be launched programmatically using the UIApplication.CanPostCommand method and then post it using the PostCommand method.
- Make use of the
  [copy and paste API](http://thebuildingcoder.typepad.com/blog/2013/05/copy-and-paste-api-applications-and-modeless-assertion.html).

#### Determine the Colour of an Element

**Question:** I want to change the colour of a specific family instance.

How can I achieve that, please?

**Answer:** Before you can affect the colour of a Revit element, you need to choose which method to use, and at what level this control should take place.
There are numerous choices, as we mentioned discussing the
[element display overrides](http://thebuildingcoder.typepad.com/blog/2012/04/getelement-method-and-get-element-type.html#2), e.g.:

- [Visibility hierarchy 1](http://revitclinic.typepad.com/my_weblog/2012/02/revit-visibility-hierarchy.html)
- [Visibility hierarchy 2](http://adndevblog.typepad.com/aec/2012/06/revit-visibility-hierarchy.html)
- [Controlling the graphical representation](http://www.aecbytes.com/tipsandtricks/2010/issue54-revit2.html)
- [Graphic overrides](http://blogs.rand.com/support/2011/10/revit-graphic-overrides.html)

#### Exception Retrieving a Bounding Box

**Question:** I am using the following three lines of code with hard-coded element ids to retrieve the bounding box of a panel element in my BIM:

```csharp
View view = doc.GetElement(
new ElementId(231354)) as View;
Panel panel = doc.GetElement(
new ElementId(4944423)) as Panel;
var bb = panel.get\_BoundingBox( view );
```

However, this pretty trivial operation is throwing a SEHException:

![Bounding box exception](img/bb_exception.png)

What is going wrong here, please?

**Answer:** You are requesting the bounding box of an element in a template view, as you can immediately see exploring the view using RevitLookup:

![Template view property](img/bb_exception_snoop.png)

Note the true 'Is Template' property.
Elements do not have any geometry in template views, so this is a nonsensical operation.

Please test the View.IsTemplate property before using a view to query element geometry.
If it returns true, then elements cannot return any geometry in that view.

#### Happy Week to All!

So, that wraps up a couple of things, and I feel free to enter my new week.
I wish you a happy week as well!