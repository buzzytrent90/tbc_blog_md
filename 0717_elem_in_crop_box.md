---
ai_optimized: true
author: Jeremy Tammik
complexity_score: 2.4
content_type: qa
optimization_date: '2025-12-11T11:44:14.455581'
original_url: https://thebuildingcoder.typepad.com/blog/0717_elem_in_crop_box.html
post_number: '0717'
reading_time_minutes: 3
series: general
slug: elem_in_crop_box
source_file: 0717_elem_in_crop_box.htm
tags:
- csharp
- elements
- filtering
- geometry
- parameters
- revit-api
- views
title: Element in View Crop Box Predicate
word_count: 516
---

### Element in View Crop Box Predicate

We had a look at one aspect of the interaction between element geometry and the view crop box last week when noting that
[section view geometry is not cropped](http://thebuildingcoder.typepad.com/blog/2012/02/section-view-geometry-not-cropped.html).

Here is another aspect: a useful little predicate function developer by Frode Tørresdal of
[Norconsult Informasjonssystemer AS](http://www.nois.no) to
determine whether an element is contained in the crop box of a view.

**Question:** Using the API I'm trying to determine if a detail item element is inside or outside the cropbox of a view.

This code works fine in a plan view, but not in a elevation view:
```csharp
  bool IsElementOutsideCropBox( Element e, View v )
  {
    bool rc = v.CropBoxActive;

    if( rc )
    {
      BoundingBoxXYZ vBox = v.CropBox;
      BoundingBoxXYZ eBox = e.get\_BoundingBox( v );
      rc = ( eBox.Min.X > vBox.Max.X )
        || ( eBox.Max.X < vBox.Min.X )
        || ( eBox.Min.Y > vBox.Max.Y )
        || ( eBox.Max.Y < vBox.Min.Y )
        || ( eBox.Min.Z > vBox.Max.Z )
        || ( eBox.Max.Z < vBox.Min.Z );
    }
    return rc;
  }
```

What might I be doing wrong, please?

**Answer:** In what way does it not work in an elevation view?
Are some (or all) of the six logical checks incorrect, or what?

In any case, comparing the bounding box min and max will never be precise, because the element's rectangular bounding box will not match the geometry of any non-rectangular element.

Instead, how about creating a FilteredElementCollector with the view so that it returns only the elements visible in the view, and then check to see if the element of interest is found by the collector?

**Response:** I tried to create the FilteredElementCollector with the view as an input parameter, but that collector also collects the elements outside the crop box.

I think I found the solution to the problem, however:
The transforms of the view and the element are different.
On all the cases I've tested, it seems to work OK if I use the inverse transform of the crop box of the view on the element.

I now use the following code to transform the bounding box:
```csharp
  bool IsElementOutsideCropBox( Element e, View v )
  {
    bool rc = v.CropBoxActive;

    if( rc )
    {
      BoundingBoxXYZ vBox = v.CropBox;
      BoundingBoxXYZ eBox = e.get\_BoundingBox( v );

      Transform tInv = v.CropBox.Transform.Inverse;
      eBox.Max = tInv.OfPoint( eBox.Max );
      eBox.Min = tInv.OfPoint( eBox.Min );

      rc = ( eBox.Min.X > vBox.Max.X )
        || ( eBox.Max.X < vBox.Min.X )
        || ( eBox.Min.Y > vBox.Max.Y )
        || ( eBox.Max.Y < vBox.Min.Y );
    }
    return rc;
  }
```

I left out the Z coordinate in this version, because the transform is not 100% accurate.
Therefore, if the element box is on the edge of the view box, the comparison might be slightly off and not return the desired result.
I should probably include a tolerance in the test, but in my case the Z test is not important anyway.

As said, the new system resolves my issue and I will use that in the future.

Many thanks to Frode for this useful hint!