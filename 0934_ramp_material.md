---
ai_optimized: true
author: Jeremy Tammik
complexity_score: 2.7
content_type: qa
optimization_date: '2025-12-11T11:44:14.927477'
original_url: https://thebuildingcoder.typepad.com/blog/0934_ramp_material.html
post_number: 0934
reading_time_minutes: 2
series: materials
slug: ramp_material
source_file: 0934_ramp_material.htm
tags:
- csharp
- elements
- family
- filtering
- geometry
- parameters
- revit-api
- selection
- views
- materials
title: Accesssing and Filtering by Ramp Material
word_count: 367
---

### Accesssing and Filtering by Ramp Material

I had a chat with Ning Zhou, who was away from the Revit API for a while and is now happily back in the fold.

He explored how to access the material of a ramp element.

#### Access to Ramp Material

**Question:** Is there a way to get the ramp material information using API?
I tried lots of paths and could not find anything.

**Answer (by Ning himself):** I searched again using RevitLookup snoop.

It turns out that basic material info is accessible after all.
I found it us under 'Object type' instead of 'Parameters'.
Apparently only the material name is stored there, in the built-in parameter 'RAMP\_ATTR\_MATERIAL':

![Snoop ramp material](img/snoop_ramp_material.jpg)

I have not seen anything providing the material volume, so I guess I'll have to use the geometry access and calculate that myself instead.

At least I can now implement a filter selection using the material name!
```csharp
  FilteredElementCollector concreteRamps
    = new FilteredElementCollector( doc )
      .WhereElementIsNotElementType()
      .OfCategory( BuiltInCategory.OST\_Ramps )
      .Where( e =>
      {
        ElementId id = e.GetValidTypes().First(
          id2 => id2.Equals( e.GetTypeId() ) );

        Material m = doc.GetElement( doc.GetElement( id )
          .get\_Parameter(
            BuiltInParameter.RAMP\_ATTR\_MATERIAL )
          .AsElementId() ) as Material;

        return m.Name.Contains( "Concrete" );
      } );
```

Many thanks to Ning for his research and sharing this helpful result.

**Addendum:** Simpler:

```csharp
  FilteredElementCollector concreteRamps
    = new FilteredElementCollector( doc )
      .WhereElementIsNotElementType()
      .OfCategory( BuiltInCategory.OST\_Ramps )
      .Where( e =>
      {
        ElementId id = e.GetTypeId();

        Material m = doc.GetElement( doc.GetElement( id )
          .get\_Parameter(
            BuiltInParameter.RAMP\_ATTR\_MATERIAL )
          .AsElementId() ) as Material;

        return m.Name.Contains( "Concrete" );
      } );
```

Thank you again, Ning, for you your additional comment below.

Before closing, here is another useful pointer on family instance placement and rotation:

#### Rotate a Family in Three Different Axes

Here is a pretty neat article on family instance placement strategies from a user point of view, describing how to
[rotate a family in three different axes](http://wikihelp.autodesk.com/Revit/enu/Community/Tips_and_Tricks/Families,_Parameters,_Formulas/Rotate_a_Family_in_3_Different_Axes),
which is certainly useful for developers as well.

As always in the Revit API, knowing the best practice from a user and product point of view is of paramount importance before putting any thoughts or efforts at all into API development issues.