---
ai_optimized: true
author: Jeremy Tammik
complexity_score: 2.1
content_type: qa
optimization_date: '2025-12-11T11:44:15.073585'
original_url: https://thebuildingcoder.typepad.com/blog/0992_view_location.html
post_number: 0992
reading_time_minutes: 3
series: views
slug: view_location
source_file: 0992_view_location.htm
tags:
- csharp
- elements
- family
- revit-api
- sheets
- views
title: Setting the Exact Same Location for two Views on a Sheet
word_count: 529
---

### Setting the Exact Same Location for two Views on a Sheet

A recent developer query requests a method to define the exact same location for two views on a sheet.

This was possible to achieve in Revit 2013 and previous releases, and some related API behaviour apparently changed in Revit 2014.

A simple but clever trick enables achieving this reliably in Revit 2014 as well.

**Question:** I noticed that sometimes when I place views on a sheet the location is somewhat shifted.

I have a Revit sample model and C# add-in that places ViewPlans on a newly created sheet.

The two views are placed at exactly the same location on the sheet, but they do not end up on top of each other anyway.

Am I doing something wrong, and if that is the case, how can I fix this?

**Answer:** We investigated this and found that there is an issue with the calculation of the viewport position.

The good news is that there is a workaround: you can temporarily set the crop box for both views to be suitably large and identical.

That way, the viewport boundaries will be identical, and you can align them easily on the sheet.

Then restore the previous crop box settings.

Here is a function with hard-wired element ids that does this for the two views in the sample project you provided:

```csharp
  private void test4( Document doc )
  {
    BoundingBoxXYZ savedBox1 = null;
    BoundingBoxXYZ savedBox2 = null;

    FamilySymbol typ = doc.GetElement(
      new ElementId( 143899 ) ) as FamilySymbol;

    ViewPlan go1 = doc.GetElement(
      new ElementId( 205387 ) ) as ViewPlan;

    ViewPlan vg1 = doc.GetElement(
      new ElementId( 205424 ) ) as ViewPlan;

    // Save current crop box

    if( go1.CropBoxActive )
      savedBox1 = go1.CropBox;

    if( vg1.CropBoxActive )
      savedBox2 = vg1.CropBox;

    if( savedBox1 != null )
    {
      // Set crop box for 2nd view = 1st

      vg1.CropBox = savedBox1;
      vg1.CropBoxActive = true;
    }
    else
    {
      // Set both views to semi-random
      // but large crop box

      BoundingBoxXYZ newBox = new BoundingBoxXYZ();
      newBox.set\_MinEnabled( 0, true );
      newBox.set\_MinEnabled( 1, true );
      newBox.set\_MinEnabled( 2, true );
      newBox.Min = new XYZ( -2000, -2000, 0 );
      newBox.set\_MaxEnabled( 0, true );
      newBox.set\_MaxEnabled( 1, true );
      newBox.set\_MaxEnabled( 2, true );
      newBox.Max = new XYZ( 2000, 2000, 0 );

      vg1.CropBox = newBox;
      go1.CropBox = newBox;
      doc.Regenerate();
      vg1.CropBoxActive = true;
      go1.CropBoxActive = true;
    }

    doc.Regenerate();

    // Create sheets and viewports

    var vSheet = ViewSheet.Create( doc, typ.Id );
    vSheet.Name = "test3";
    var location2 = new XYZ( 0, 0, 0 );

    var v12 = Viewport.Create(
      doc, vSheet.Id, go1.Id, location2 );

    var v22 = Viewport.Create(
      doc, vSheet.Id, vg1.Id, location2 );

    doc.Regenerate();

    // Align lower left - works because
    // crop boxes are same

    Outline outline1 = v12.GetBoxOutline();
    Outline outline2 = v22.GetBoxOutline();

    XYZ min1 = outline1.MinimumPoint;
    XYZ min2 = outline2.MinimumPoint;

    XYZ diffToMove = min1 - min2;

    ElementTransformUtils.MoveElement(
      doc, v22.Id, diffToMove );

    doc.Regenerate();

    // Restore view crop boxes

    if( savedBox1 == null )
    {
      go1.CropBoxActive = false;
    }
    else
    {
      go1.CropBox = savedBox1;
      go1.CropBoxActive = true;
      go1.CropBoxVisible = false;
    }

    if( savedBox2 == null )
    {
      vg1.CropBoxActive = false;
    }
    else
    {
      vg1.CropBox = savedBox2;
      vg1.CropBoxActive = true;
      vg1.CropBoxVisible = false;
    }
  }
```

Now the result looks correct. Hope this helps.

**Response:** I am happy to inform you that I tested your workaround in a couple of scenarios in our main application and it continues to work well. Thank you!