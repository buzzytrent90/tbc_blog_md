---
post_number: "1264"
title: "Happy New Year and New Beginnings!"
slug: "happy_new_year"
author: "Jeremy Tammik"
tags: ['csharp', 'elements', 'family', 'geometry', 'revit-api', 'views', 'windows']
source_file: "1264_happy_new_year.htm"
original_url: "https://thebuildingcoder.typepad.com/blog/1264_happy_new_year.html"
---

### Happy New Year and New Beginnings!

Happy New Year!

Welcome back!

I'll dive straight into today's topics:

- [The twelve nights](#2)
- [More wrapping up and letting go](#3)
- [A bunch of Revit API discussion forum threads](#4)
- [Determining the total family instance transformation](#5)

#### The Twelve Nights

Once again, we survived the
[Twelfth Night](https://en.wikipedia.org/wiki/Twelfth_Night_(holiday)),
the end of this special twelve-day turning point of the year, also known as
[Rauhnächte](https://de.wikipedia.org/wiki/Rauhnacht) or
'raw nights' in German, full of special depth and significance, related to the differences between the
[lunar](https://en.wikipedia.org/wiki/Lunar_calendar) and
solar cycles, beginning with
[Christmas](https://en.wikipedia.org/wiki/Christmas),
[Hanukkah](https://en.wikipedia.org/wiki/Hanukkah),
Celtic [Samhain](https://en.wikipedia.org/wiki/Samhain),
Druid [Alban Arthan](https://en.wikipedia.org/wiki/Alban_Arthan),
and many other sacred traditions.

A time of confusion, breaking things, going wrong, calming down, going slowly, contemplation, relaxing into peace and quiet.

![Rauhnacht](img/rauhnacht.jpg)

Congratulations on coming through intact, and a Happy and Harmonious New Year to you!

#### More Wrapping Up and Letting Go

I left you last year very busy
[wrapping up](http://thebuildingcoder.typepad.com/blog/2014/12/back-from-the-conference-tour-and-wrapping-up.html) and
citing the
[wise words by Victor Hugo](http://thebuildingcoder.typepad.com/blog/2014/12/the-building-coder-wishes-you-a-merry-christmas.html#8) on
facing the future and letting go...

I have been busy with both during the break, discussing with my colleagues about starting a new blog focussed on 3D web topics.

We have been preaching to you about the
[DevDays 10x theme](http://thebuildingcoder.typepad.com/blog/2014/12/devdays-conference-at-autodesk-university.html#2),
and, as always, it seems appropriate to me to eat our own dog food.

Several happy changes make this easier for me: new colleagues joined the AEC workgroup,
[starting to blog busily](http://thebuildingcoder.typepad.com/blog/2014/12/the-building-coder-wishes-you-a-merry-christmas.html#6) in
their own right, many questions on the Revit API have been answered here and elsewhere, the number of cases seem stable to me in the Western hemisphere, growing most in the Eastern, and the
[Revit API discussion forum](http://forums.autodesk.com/t5/revit-api/bd-p/160) is
steadily providing more answers still.

#### A Bunch of Revit API Discussion Forum Threads

In fact, I spent part of yesterday chipping in there myself as well, addressing many topics:

- [Revit build version to update release number](http://forums.autodesk.com/t5/revit-api/revit-build-version-to-update-release-number/m-p/5453896)
- [Mouse over context help](http://forums.autodesk.com/t5/revit-api/mouse-over-context-help/m-p/5455910)
- [Dockable panel using Windows Forms C#](http://forums.autodesk.com/t5/revit-api/dockable-panel-using-windowsforms-c/m-p/5454252)
- [Compare and combine data tables](http://forums.autodesk.com/t5/revit-api/compare-and-combine-datatables/m-p/5457950)
- [How to export an image from a specific view using Revit API in C#](http://forums.autodesk.com/t5/revit-api/how-to-export-an-image-from-a-specific-view-using-revit-api-c/m-p/5457965)
- [Revit 2015 seems incomplete](http://forums.autodesk.com/t5/revit-api/revit-2015-seems-incomplete/td-p/5457627)
- [Revit API regarding walkthroughs](http://forums.autodesk.com/t5/revit-api/revit-api-regarding-walkthroughs/td-p/5456450)
- [Selecting a element and viewing its properties in a text box](http://forums.autodesk.com/t5/revit-api/select-a-element-and-view-properties-in-a-textbox/m-p/5455712)
- [Determine outer surface of element](http://forums.autodesk.com/t5/revit-api/determine-outer-surface-of-element/m-p/5447235)
- [CAD import within family and how to delete the associated import](http://forums.autodesk.com/t5/revit-api/cad-import-within-family-gt-how-to-delete-associated-import/m-p/5452306)
- [API access to a void form in a conceptual model](http://forums.autodesk.com/t5/revit-api/api-access-to-void-form-in-conceptual-model/m-p/5458686)

#### Determining the Total Family Instance Transformation

I also discussed a couple of issues off-line, although I always try to avoid that as much as possible, in order to optimise the global sharing of information that we are all contributing to.

Here is one of them:

**Question:** I am traversing all the Revit database elements and determining their geometry.

For this, I need to retrieve the transformation matrix, especially for family instance elements, e.g. like this:

```csharp
  FamilyInstance inst = e as FamilyInstance;
  Transform transform = inst.GetTransform();
```

I am converting this transformation to a 4x4 matrix for further use in my add-in.

I tried lots of ways to retrieve the proper transformation from the transform above with no success.

The instance from the model gets misplaced after processing.

If I set the identity matrix on each instance, it works fine, but for supporting instancing I need to take the proper transform into consideration.

Can you please guide me how to determine the proper transformation for the Revit instance so that I can support instancing in my add-in?

**Answer:** There are several different possible ways in which family instance geometry may be transformed into the placement in the model.

The only sample that I fully trust to properly handle all the different combinations is the
[Revit SDK ElementViewer sample](http://thebuildingcoder.typepad.com/blog/2009/03/transform-instance-coordinates.html).

I would suggest testing that on your specific family instances and models and debugging what exact steps it traverses to achieve the required transformation.

I worked though parts of the same process when implementing my OBJ exporter, and also document some problems that I ran into that sound similar to your issue:

- [OBJ Model Export Considerations](http://thebuildingcoder.typepad.com/blog/2012/06/obj-model-export-considerations.html)
- [OBJ Model Exporter Take One](http://thebuildingcoder.typepad.com/blog/2012/06/obj-model-exporter-take-one.html)
- [OBJ Model Exporter with Colours](http://thebuildingcoder.typepad.com/blog/2012/07/obj-model-exporter-with-colours.html)
- [OBJ Model Exporter with Multiple Solid Support](http://thebuildingcoder.typepad.com/blog/2012/07/obj-model-exporter-with-multiple-solid-support.html)
- [OBJ Model Exporter with Transparency Support](http://thebuildingcoder.typepad.com/blog/2012/07/obj-model-exporter-with-transparency-support.html)

However, all of these complexities can be avoided completely by using the
[custom exporter framework instead](http://thebuildingcoder.typepad.com/blog/about-the-author.html#5.1).

Have you taken a look at that?

Would that be more suitable and easier to use to fulfil your needs?