---
ai_optimized: true
author: Jeremy Tammik
complexity_score: 2.5
content_type: qa
optimization_date: '2025-12-11T11:44:13.966398'
original_url: https://thebuildingcoder.typepad.com/blog/0450_place_detail_inst.html
post_number: '0450'
reading_time_minutes: 2
series: general
slug: place_detail_inst
source_file: 0450_place_detail_inst.htm
tags:
- csharp
- elements
- family
- revit-api
- views
title: Place Detail Instance
word_count: 337
---

### Place Detail Instance

Here is a little note on a problem regarding the placement of a detail family instance in the active view that has appeared several times lately, and its solution by Robert Pleysier of
[ALFA Development](http://www.alfadevelopment.nl):

I want to place a detail family instance in a section view.
In my initial attempts, the call to NewFamilyInstance was placing the instance in the wrong view.
After using RevitLookup, we found that the instance was actually hosted in the section view.
We began experimenting with the other NewFamilyInstance options and finally found the following option which solved our problem:
```csharp
  List<Element> docViews = util.GetElemOfType(
    doc, typeof( View ) );

  View v1 = getView( docViews, "Exterior" );

  FamilyInstance fin = factory.NewFamilyInstance(
    XYZ.Zero, famSymbol, v1.SketchPlane,
    StructuralType.NonStructural );
```

This code is using the following helper method to find a view by name:
```csharp
View getView( List<Element> views, string name )
{
  foreach( View v in views )
  {
    if( v.ViewName == name )
    {
      return v;
    }
  }
  return null;
}
```

This approach is making use of the overload of NewFamilyInstance taking the following arguments:

- XYZ location- FamilySymbol symbol- Element host- StructuralType structuralType

So the sketch plane of the view is actually being regarded as the host of the detail instance.
I find this pretty interesting.

**Important Note:** Here is an important note from Scott Conover, Revit API Software Development Manager, on the suggestion above:

In Revit 2011, NewFamilyInstance(XYZ, FamilySymbol, View) is the overload designed for the use case.

I'm surprised the other one works.

I am just in the process of exploring another case dealing with a similar issue, and will post again with a more complete solution for this issue as soon as it is fully clarified and tested.

**Response:** Robert responds to Scott's suggestion and says:

We tried the method recommended by Scott Conover in the latest Revit 2011 but we did not succeed. After placing a detail family instance in a view like elevation or section, the detail family was placed in the plan view.

The exploration continues...