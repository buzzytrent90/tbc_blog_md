---
post_number: "0301"
title: "Detail Curve on Level"
slug: "detail_curve_level"
author: "Jeremy Tammik"
tags: ['csharp', 'elements', 'geometry', 'levels', 'parameters', 'revit-api', 'views']
source_file: "0301_detail_curve_level.htm"
original_url: "https://thebuildingcoder.typepad.com/blog/0301_detail_curve_level.html"
---

### Detail Curve on Level

In the past, we discussed
[detail curve creation](http://thebuildingcoder.typepad.com/blog/2009/09/detail-lines.html) and
[modifying their colour](http://thebuildingcoder.typepad.com/blog/2010/01/model-and-detail-curve-colour.html).
Several parties including
[Shifali](http://thebuildingcoder.typepad.com/blog/2008/12/linked-files.html?cid=6a00e553e168978833012876fee5bf970c#comment-6a00e553e168978833012876fee5bf970c)
and
[Dharanidharan](http://thebuildingcoder.typepad.com/blog/2008/08/the-revit-sdk-c.html?cid=6a00e553e1689788330120a555869e970c#comment-6a00e553e1689788330120a555869e970c) inquired
about creating detail curves on a specified level.
Here is a solution to that issue by Joe Ye.

**Question:** I am using the following code to draw some circles using arcs:
```csharp
  XYZ p = XYZ.Zero;
  XYZ norm = XYZ.BasisZ;
  double startAngle = 0;
  double endAngle = 2 \* Math.PI;
  double radius = 1.23;

  Plane plane = new Plane( norm, p );

  Arc arc = app.Create.NewArc(
    plane, radius, startAngle, endAngle );

  DetailArc detailArc
    = doc.Create.NewDetailCurve(
      doc.ActiveView, arc ) as DetailArc;
```

This code draws an arc in the active level of the active view.
I would like to specify the level as a parameter and draw the arc on that level independently of the current active level.
In the code above, I am setting the Z coordinate to the height of the level, hoping the curves will be drawn on the desired level.
Instead, they are drawn on the active level of the active view of the document.
Is there any way to draw the arcs on a specified level independent of the current active level?

**Answer:** You just need to set the first argument of the NewDetailCurve method to the specific level ViewPlan.
For example, to draw an arc on Level 2, first find the ViewPlan instance associated with that level.
To find the target view plan, you can iterate over all ViewPlan instances and check their GenLevel property.
If it matches the desired target level, this is the one we need.
Here is some sample code illustrating this:
```csharp
using System;
using Autodesk.Revit;
using Autodesk.Revit.Elements;
using Autodesk.Revit.Geometry;

public class RevitCommand : IExternalCommand
{
  public IExternalCommand.Result Execute(
    ExternalCommandData commandData,
    ref string messages,
    ElementSet elements )
  {
    Application app = commandData.Application;
    Document doc = app.ActiveDocument;

    // Create an arc on the plane whose
    // center is at the plane origin:

    XYZ end0 = new XYZ( 0, 0, 1 );
    XYZ end1 = new XYZ( 1, 3, 2 );
    XYZ norm;

    if( end0.X == end1.X )
    {
      norm = XYZ.BasisZ;
    }
    else if ( end0.Y == end1.Y )
    {
      norm = XYZ.BasisZ;
    }
    else
    {
      norm = XYZ.BasisZ;
    }

    double startAngle = 0;
    double endAngle = 2 \* Math.PI;
    double radius = 5;

    Plane objPlane
      = app.Create.NewPlane( norm, XYZ.Zero );

    // ViewPlan of "Level 2"

    ViewPlan vp2 = null;

    ElementIterator ei
      = doc.get\_Elements( typeof( ViewPlan ) );

    while( ei.MoveNext() )
    {
      ViewPlan vp = ei.Current as ViewPlan;

      if( vp.GenLevel.Name.Equals( "Level 2" ) )
      {
        vp2 = vp;
        break;
      }
    }

    if( null == vp2 )
    {
      vp2 = doc.ActiveView as ViewPlan;
    }

    if( null != vp2 )
    {
      // draw the circle:

      Arc arc = app.Create.NewArc( objPlane,
        radius, startAngle, endAngle );

      DetailArc detailArc
        = doc.Create.NewDetailCurve(
          vp2, arc ) as DetailArc;
    }
    return (null == vp2)
      ? IExternalCommand.Result.Failed
      : IExternalCommand.Result.Succeeded;
  }
}
```

Another point I would like to mention:
I noticed in your code that you create the Plane instance using one of its constructor member methods.
This works fine as long as you are not working in VSTA.
You can also use the dedicated Autodesk.Revit.Creation.Application method NewPlane to create the plane object.
UV and XYZ instances can also be created both ways, either using their constructor member methods or dedicated creation application methods.
If you are working in VSTA, you have to use the application creation methods, because the constructors will not work.

For completeness sake, here is the complete
[detail\_curve\_level](zip/detail_curve_level.zip) source code and Visual Studio solution.
Many thanks to Joe for providing it!