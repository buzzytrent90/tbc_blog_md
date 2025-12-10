---
post_number: "0707"
title: "Curved Analytical Model Approximation and Etabs Structural Link"
slug: "etabs_rst_link"
author: "Jeremy Tammik"
tags: ['csharp', 'elements', 'family', 'geometry', 'levels', 'revit-api', 'walls']
source_file: "0707_etabs_rst_link.htm"
original_url: "https://thebuildingcoder.typepad.com/blog/0707_etabs_rst_link.html"
---

### Curved Analytical Model Approximation and Etabs Structural Link

Here is a note on how to retrieve approximate straight line segments for the analytical model of curved structural elements, and on a free Revit Structural link tool to the
[Computers and Structures, Inc. (CSI)](http://www.csiberkeley.com)
[ETABS](http://www.csiberkeley.com/etabs)
building analysis and design environment.

A couple of weeks ago Nasser emailed me about a method he was trying to use for his Revit Structure link add-in.
He was trying to export floor elements with curved edges to the structural analysis software ETABS.
Like many other structural analysis applications, ETABS does not handle arcs and splines, so linear segmentation of curved elements is required.

The Revit API AnalyticalModel class provides an Approximate method which promises to achieve exactly that.
Unfortunately, it is not yet implemented for this case.

A simple workaround is to use the Tessellate method instead.
In addition, by skipping some of the intermediate points, the precision of the approximation can be lowered if needed.

Here is an example of how the Tessellate method can be used instead of the Approximate.
The minimum line segment length required is defined by the LineSegmentLength argument:
```csharp
  public IList<Curve>
    GetStraightLineCurvesFromFloorAnalyticalModel(
      Document doc,
      AnalyticalModel analyticalmodel,
      double lineSegmentLength )
  {
    IList<Curve> Curves;
    IList<Curve> SegmentedCurves = new List<Curve>();

    // If no analytical model then skip

    if( analyticalmodel == null )
    {
      return null;
    }

    Curves = analyticalmodel.GetCurves(
      AnalyticalCurveType.ActiveCurves );

    // This does not work:
    //Curves = analyticalmodel.GetCurves(
    //  AnalyticalCurveType.ApproximatedCurves);

    foreach( Curve curve in Curves )
    {
      IList<XYZ> pts = curve.Tessellate();

      int ibefore = 0;

      for( int i = 1; i < pts.Count; i++ )
      {
        double distance
          = GeometryUtility.Get3DDistance(
            pts[ibefore], pts[i] );

        if( pts.Count - 1 == i )
        {
          SegmentedCurves.Add(
            doc.Application.Create.NewLineBound(
              pts[ibefore], pts[i] ) );
        }
        else
        {
          if( distance < lineSegmentLength )
          {
            continue;
          }
          else
          {
            SegmentedCurves.Add(
              doc.Application.Create.NewLineBound(
                pts[ibefore], pts[i] ) );

            ibefore = i;
          }
        }
      }
    }
    return SegmentedCurves;
  }
```

This method is part of
[Nasser's Revit Tools](http://www.nassermarafi.com/?page_id=69),
a Revit Structural link add-in to integrate with the
[Computers and Structures, Inc. (CSI)](http://www.csiberkeley.com)
[ETABS](http://www.csiberkeley.com/etabs)
building analysis and design environment.

Nasser's add-in exports the following elements to ETABS: Columns, Braces, Beams, Walls, Slabs, Openings, Grids, Levels and Rigid Links.
It recognizes most family elements.
Custom created structural family instance elements will be exported as null, and their properties can later be adjusted in ETABS.

Here is a sample model in Revit:

![Revit sample model](img/nasser_example_revit.png)

This is the result of exporting it to ETABS:

![Revit model exported to ETABS](img/nasser_example_etabs.png)

Visit
[www.nassermarafi.com](http://www.nassermarafi.com) for
more information or to access the free download.
Feel free to
[contact Nasser directly](mailto:me@nassermarafi.com) for
any questions or other issues.