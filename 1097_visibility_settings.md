---
ai_optimized: true
author: Jeremy Tammik
complexity_score: 5.2
content_type: qa
optimization_date: '2025-12-11T11:44:15.298749'
original_url: https://thebuildingcoder.typepad.com/blog/1097_visibility_settings.html
post_number: '1097'
reading_time_minutes: 6
series: views
slug: visibility_settings
source_file: 1097_visibility_settings.htm
tags:
- csharp
- elements
- family
- geometry
- levels
- parameters
- revit-api
- views
title: The GEOM_VISIBILITY_PARAM Visibility Settings
word_count: 1208
---

### The GEOM\_VISIBILITY\_PARAM Visibility Settings

Here is an nice, useful and interesting example of a Revit API forum
[discussion thread](http://forums.autodesk.com/t5/Revit-API/Visibility-settings/m-p/4764061) and
cooperative research between
Remy van den Bor of [ICN Solutions](http://www.icnsolutions.net),
Rudolf Honke of [Mensch und Maschine acadGraph](http://www.acadgraph.de),
the Revit development team, Joe Ye and your humble scribe on retrieving an element's detail view visibility settings, before wishing you and us all the best for the
[year of the horse](#2):

**Remy:** Hello Reviteers,

Does anybody know how to get to the visibility settings in the family editor with the API?

Manually, you just select the element, click Visibility settings and change its appearance, e.g.,
Plan/RCP, Front/Back, Left/Right, Coarse, Medium or Fine.

I tried snooping using RevitLookup, but I can't find the parameter that is affected when I alter these settings.

Help greatly appreciated.

**Rudolf:** Try this:

```csharp
  FamilyElementVisibility visibility = new
    FamilyElementVisibility(
      FamilyElementVisibilityType.Model );

  visibility.IsShownInCoarse = true;
  visibility.IsShownInFine = true;
  visibility.IsShownInMedium = true;
  visibility.IsShownInFrontBack = true;
  visibility.IsShownInLeftRight = true;
  visibility.IsShownInPlanRCPCut = false;

  yourElement.SetVisibility( visibility );
```

**Remy:** Thank you very much, that is what I needed :)

Only one question remains: you say "yourElement.SetVisibility(visibility)", but I can't get it to work.

I tried calling the SetVisibility method on both FamilyInstance and Element.

Both tell me they don't have such a method.

The only thing I can find is the doc.ActiveView.SetVisibility but it doesn't have a constructor for an element, only a whole category :-(

So my next question is: how can I use the SetVisibility for a family instance (within the family editor)?

**Rudolf:** Oh, I see.

I used it for SymbolicCurve elements.

In the Family context, you can use it also with ModelCurves.

Also, each element whose class is derived from GenericForm will provide this method, meaning Extrusions, Sweeps and so on.

**Remy:** Okay, I see.

Yes, when I create an extrusion and snoop it, I see the method you mention.

But I have a different situation: I made a sweep and loaded this sweep into another family, so it's a nested family.

Snooping this family, Revit sees it as FamilyInstance, which doesn't have the FamilyElementVisibility method.

And you can't simply cast the FamilyInstance to a sweep :-)

**Rudolf:** There is another method to access the visibility.

FamilyInstances residing in a Family document provide a built-in parameter named GEOM\_VISIBILITY\_PARAM.

This is an integer typed read-write parameter, so it can be set.

The only thing you have to do is to find out the relation between the integer value and the corresponding menu entries in the user interface.

In the following dialog, the parameter is named "Überschreibungen Sichtbarkeit/Grafiken".

Here are the visibility settings in the user interface:

![](img/visibility_setting_1.png)

We can see the resulting parameter value in RevitLookup:

![](img/visibility_setting_2.png)

Changing the parameter value in the user interface:

![](img/visibility_setting_3.png)

The changes are reflected in RevitLookup:

![](img/visibility_setting_4.png)

So what you could do is to test different configurations and take note of their corresponding integer representations.

These values can be used for setting the visibility even with nested families.

So I hope ;-)

**Remy:** Super, that's it.

Normally, I don't like the "magic numbers" approach, but this time I'll make an exception.

So, to summarise:

The "FamilyElementVisibility" method can be used when creating solids in a family (sweeps, extrusions etc.).

The "magic numbers" method can be used when using families as nested families.

To name a few:

- All checked = 57406
- Plan/RCP (Grundriss/Deckenplan) unchecked = 57398
- Left/Right (Links/rechts) unchecked = 57374

Thanx again :smileyhappy:

**Rudolf:** This approach may be picked up by Jeremy, I hope.

It's a typical workaround in the Revit API environment.

It would be great not only to know all the magic numbers, but also the algorithm that calculates them.

Also, are the values stable even between different Family documents?

**Remy:** Yes, I agree 100%.

As far as I can tell these numbers are stable for 3D elements.

In a detail component family there is only the coarse/medium/fine setting and the visibility/graphics overrides parameter has value 0, so no magic numbers there.

So, it would be a nice topic for Jeremy indeed :smileyhappy: (hint)

**Jeremy:** I fully agree that this looks interesting.

Two additional observations:

- Those magic numbers are probably much less magic when you look at them in binary or hexadecimal representation.
- I would not be at all surprised if there is an enumeration value defined somewhere in the Revit API, and Boolean or combinations of the enumeration values provide these 'magic' or not-so-magic values.

What do you think?

Have you seen any likely candidates?

**Rudolf:** Yes, I also think so.

These magic numbers aren’t magic at all, there must be a way to calculate them.

They are some sort of combined enumeration, I fully agree.

As Remy pointed out, there is a constant first number for 2D families.

To finish the forum thread, it would be nice if somebody could evaluate the algorithm by trying out the different combinations.

**Remy:** I am sure there must be some enumeration behind it, only not available for us plebs :smileyhappy:

So for now, the numbers appear magical to me, but Joe Ye already submitted a wish list item to expose them officially, so who knows...

**Andrzej:** Here are the definitions of the constants used in C++:

```csharp
  unsigned int m\_bUndefined:1; // Mostly for caching errors
  unsigned int m\_bModel:1;

  // Model
  unsigned int m\_bPlanRCPCut:1;
  unsigned int m\_bTopBottom:1;
  unsigned int m\_bFrontBack:1;
  unsigned int m\_bLeftRight:1;

  // View direction specific
  unsigned int m\_bOnlyWhenCut:1;

  unsigned int m\_reserved:6;

  // Detail level
  unsigned int m\_bCoarse:1;
  unsigned int m\_bMedium:1;
  unsigned int m\_bFine:1;

  // ----- 16 bits up to here -------

  // Old stuff - for compatibility with loaded families
  unsigned int m\_bOld:1;
  unsigned int m\_bOldGeom:1;

  // Old Geom
  unsigned int m\_bOldGeomPlan:1;
  unsigned int m\_bOldGeomRCP:1;
  unsigned int m\_bOldGeomElev:1;
  unsigned int m\_bOldGeom3d:1;

  // Old Curves
  unsigned int m\_bOldCurve3d:1;
  unsigned int m\_bOldCurveParallel:1;
  unsigned int m\_bOldCurveAntiParallel:1;
  unsigned int m\_bOldCurveInFront:1;
```

For example, m\_bCoarse is at position 14, and 'reserved' is taking 6 bits, so you can check it by using a bitmask defined like this:

```csharp
  int ival = e.get\_Parameter(
    BuiltInParameter.GEOM\_VISIBILITY\_PARAM )
      .AsInteger();

  bool isCourse = 0 != (ival & (0x1 << 13));
```

**Scott:** You should also take a look at the discussion on
[setting the visibility](http://thebuildingcoder.typepad.com/blog/2013/01/set-detail-curve-visibility.html) of
a detail curve.

Normally, an add-in will only need to deal with the detail level stuff at this level, because the Revit API already exposes model level geometry visibility at a higher level as a more easily used API object.

Many thanks to all the contributors for their help in exploring and sharing this!

#### Happy Year of the Horse!

![Happy Year of the Horse!](img/year_of_the_horse_2014.png)

Happy
[Year of the Horse](http://www.theguardian.com/technology/2014/jan/30/chinese-new-year-google-doodle)!