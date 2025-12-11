---
ai_optimized: true
author: Jeremy Tammik
complexity_score: 3.4
content_type: code_example
optimization_date: '2025-12-11T11:44:15.570679'
original_url: https://thebuildingcoder.typepad.com/blog/1234_setout_points.html
post_number: '1234'
reading_time_minutes: 4
series: geometry
slug: setout_points
source_file: 1234_setout_points.htm
tags:
- csharp
- elements
- family
- geometry
- parameters
- python
- revit-api
- schedules
title: Concrete Setout Points for Revit Structure 2015
word_count: 738
---

### Concrete Setout Points for Revit Structure 2015

I was prompted by a Revit API forum discussion thread on
[Jeremy's setoutpoint](http://forums.autodesk.com/t5/revit-api/jeremy-s-setoutpoint/m-p/5376217) to
take another look at my
[SetoutPoints structural concrete setout point add-in](http://thebuildingcoder.typepad.com/blog/2012/08/structural-concrete-setout-point-add-in.html),
publish it on GitHub and migrate it from Revit Structure 2013 to 2015.

SetoutPoints is a Revit add-in for automatic placement and management of structural concrete setout points.

Setout point markers are automatically generated at all vertices of structural members.
They can be manually designated as minor or major:

![Minor and major structural concrete setout points](img/setout_points_7.png)

The major points are automatically numbered, scheduled and converted to global northing and easting coordinates.

The numbering can be edited and regenerated to ensure schedule consistency and readability.

Here is a collection on background and further reading on this project:

- [Detailed description](http://thebuildingcoder.typepad.com/blog/2012/08/structural-concrete-setout-point-add-in.html)
- [Original implementation](http://thebuildingcoder.typepad.com/blog/2012/03/melbourne-devlab.html)
- [Commercial use](http://thebuildingcoder.typepad.com/blog/2013/01/basic-file-info-and-rvt-file-version.html)
- [Discussion thread](http://forums.autodesk.com/t5/revit-api/jeremy-s-setoutpoint/m-p/5372337)

Sanjaymann took a look at this project, encountering and solving a transformation issue with some structural foundations:

**Question:** I was working with the setoutpoint example on structural foundations.
The code places the points on one foundation but not on the other.
Both of them are same family.

While debugging, the only difference that I saw was that in the case of the first foundation, the code directly retrieves a solid in the GetSolid function; in the case of the second one, it retrieves a geometry instance instead:

```python
  /// <summary>
  /// Retrieve the first non-empty solid found for
  /// the given element. In case the element is a
  /// family instance, it may have its own non-empty
  /// solid, in which case we use that. Otherwise we
  /// search the symbol geometry. If we use the
  /// symbol geometry, we have to keep track of the
  /// instance transform to map it to the actual
  /// instance project location.
  /// </summary>
  Solid GetSolid(
    Element e,
    Options opt,
    out Transform t )
  {
    GeometryElement geo = e.get\_Geometry( opt );

    Solid solid = null;
    GeometryInstance inst = null;
    t = Transform.Identity;

    // Some columns have no solids, and we have to
    // retrieve the geometry from the symbol;
    // others do have solids on the instance itself
    // and no contents in the instance geometry
    // (e.g. in rst\_basic\_sample\_project.rvt).

    foreach( GeometryObject obj in geo )
    {
      solid = obj as Solid;

      if( null != solid
        && 0 < solid.Faces.Size )
      {
        break;
      }

      inst = obj as GeometryInstance;
    }

    if( null == solid && null != inst )
    {
      geo = inst.GetSymbolGeometry();
      t = inst.Transform;

      foreach( GeometryObject obj in geo )
      {
        solid = obj as Solid;

        if( null != solid
          && 0 < solid.Faces.Size )
        {
          break;
        }
      }
    }
    return solid;
  }
```

As said, both foundations belong to the same family.
Why do they behave so differently?

**Answer:** The difference may be caused by the fact that one foundation can use the unmodified family symbol geometry, whereas the other one needs some kind of instance specific modification, and therefore cannot use the unmodified symbol geometry.
Does that make sense?

**Response:** Sometimes the sword fails and the needle does the job.

While looping over the corners:

```csharp
  foreach( XYZ p in corners.Keys )
```

The point p has to be transformed:

```csharp
  XYZ p1 = t.OfPoint( p );

  FamilyInstance fi
    = doc.Create.NewFamilyInstance( p1,
      symbols[1], StructuralType.NonStructural );
```

One line did the trick.

**Answer:** Congratulations on finding the right needle for the job!

Thank you very much for your research!

Motivated by your work, I created a
[GitHub repository for the SetoutPoints application](https://github.com/jeremytammik/SetoutPoints) and
migrated it to Revit Structure 2015.

I applied the change that you suggest by adding the transformation as an 'out' parameter to the GetSolid method instead of moving the code out, cf. the
[GitHub diffs](https://github.com/jeremytammik/SetoutPoints/commit/13e134375c38573c9f45d755973359b98681db7c).

The updated solution for Revit 2015 with your transformation added is published in the
[SetoutPoints GitHub repository](https://github.com/jeremytammik/SetoutPoints) and
your transformation enhancement is the content of
[release 2015.0.0.1](https://github.com/jeremytammik/SetoutPoints/releases/tag/2015.0.0.1).

Many thanks to Sanjaymann for his interest and research!

I hope this is of interest to others as well.