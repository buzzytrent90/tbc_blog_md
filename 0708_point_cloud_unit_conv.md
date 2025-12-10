---
post_number: "0708"
title: "Point Cloud Unit Conversion"
slug: "point_cloud_unit_conv"
author: "Jeremy Tammik"
tags: ['python', 'references', 'revit-api']
source_file: "0708_point_cloud_unit_conv.htm"
original_url: "https://thebuildingcoder.typepad.com/blog/0708_point_cloud_unit_conv.html"
---

### Point Cloud Unit Conversion

Happy New Year of the Dragon!

![Happy New Year of the Dragon](img/year_of_the_dragon.png)

Top: Happy New Year!

Bottom: Lucky Peace!

The Chinese dragon is the symbol of emperor and power.

Thanks to Joe Ye and [nipic.com](http://www.nipic.com) for the image and translation!

Besides programming Revit and climbing mountains, I also like climbing trees:

Returning to the Revit API, the topic of
[units](http://thebuildingcoder.typepad.com/blog/units) is
a recurring theme, and here it is again rearing its beautiful head in the context of point cloud data access:

**Question:** I am working on a point cloud engine implementation for Revit.
In the IPointCloudAccess interface, the GetUnitsToFeetConversionFactor method seems to define the factor between the point cloud units and feet.
It is not working properly for me, though.
The only way I am able to use it so far is to convert all my points into feet myself and have this method return a constant factor of 1.0.

My point clouds always have a unit of measure, which is currently defined in meters, so an
internal data item of 1 means 1 meter.
If the Revit model's internal data also uses a fixed built-in length unit and not the unit preference to visible in the user interface, then the conversion between the point cloud data and the Revit model is fixed.
Otherwise, if the Revit model's internal data is unitless (like AutoCAD DWG), the customer has to determine what unit is to be used to map the points into the model.

In the Revit user interface, I experimented with creating a circle and saving it.
When it is selected, it displays the radius in the current user interface unit preference.
If I change the unit preference, it displays a new value, showing that the actual radius did not change.
This makes me believe that the internal data has a unit of measure.
What is it?
Is there a difference between the US English and German versions of Revit?

**Answer:** The explorations you describe toward the end of your query are leading in the absolutely right direction, and the solution is very simple, but not obvious.

Revit does indeed use a fixed internal database unit for all length measurements, and that unit is the imperial foot.
There is thus no difference between the different versions and languages of Revit; they all use the same internal database unit.
This fact is special enough to deserve a
[separate discussion](http://thebuildingcoder.typepad.com/blog/2011/03/internal-imperial-units.html) of its own.

A test very similar to the one you describe can easily be expanded to reliably determine the conversion factor between any given user interface unit and the internal Revit database one, as explained in the discussion of
[unit conversion](http://thebuildingcoder.typepad.com/blog/2011/01/unit-conversion-and-new-blogs.html) and
preceding posts.
You can also look at the entire collection of
[unit topics](http://thebuildingcoder.typepad.com/blog/units).

Here is a suggestion on how to implement the IPointCloudAccess interface GetUnitsToFeetConversionFactor method, assuming the point cloud data is given in millimetres:
```python
  class PointCloudAccess : IPointCloudAccess
  {
    const double \_mm\_per\_inch = 25.4;
    const double \_mm\_per\_foot = 12 \* \_mm\_per\_inch;
    const double \_mm\_to\_feet = 1.0 / \_mm\_per\_foot;

    public double GetUnitsToFeetConversionFactor()
    {
      return \_mm\_to\_feet;
    }

    // . . .

  }
```