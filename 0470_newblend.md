---
post_number: "0470"
title: "NewBlend Sample"
slug: "newblend"
author: "Jeremy Tammik"
tags: ['csharp', 'elements', 'family', 'python', 'revit-api', 'transactions', 'views']
source_file: "0470_newblend.htm"
original_url: "https://thebuildingcoder.typepad.com/blog/0470_newblend.html"
---

### NewBlend Sample

Until now, The Building Coder included no example at all of using the NewBlend method.
A developer ran into an issue using it, and my colleague Joe Ye created an external command to resolve the issue, so I took and added it.
Here is the issue that prompted this:

**Question:** I am trying to create a blend with two circular profiles, but it is not working.
Can you please provide some solution for us?

**Answer:** After several tests, I found that the issue is due to the fact that the top and bottom profiles should contain at least two curves.

I split the circle that you were using for you profile into two semi-circle arcs.
Now the command successfully creates the new blend.

I used Joe's sample code to create a new Building Coder sample external command CmdNewBlend.
It exercises the following helper method CreateBlend for the actual new blend creation:
```csharp
static Blend CreateBlend( Document doc )
{
  Debug.Assert( doc.IsFamilyDocument,
    "this method will only work in a family document" );

  Application app = doc.Application;

  Autodesk.Revit.Creation.Application creApp
    = app.Create;

  Autodesk.Revit.Creation.FamilyItemFactory factory
    = doc.FamilyCreate;

  double startAngle = 0;
  double midAngle = Math.PI;
  double endAngle = 2 \* Math.PI;

  XYZ xAxis = XYZ.BasisX;
  XYZ yAxis = XYZ.BasisY;

  XYZ center = XYZ.Zero;
  XYZ normal = -XYZ.BasisZ;
  double radius = 0.7579;

  Arc arc1 = creApp.NewArc( center, radius,
    startAngle, midAngle, xAxis, yAxis );

  Arc arc2 = creApp.NewArc( center, radius,
    midAngle, endAngle, xAxis, yAxis );

  CurveArray baseProfile = new CurveArray();

  baseProfile.Append( arc1 );
  baseProfile.Append( arc2 );

  XYZ center2 = new XYZ( 0, 0, 1.27 );

  Arc arc3 = creApp.NewArc( center2, radius,
    startAngle, midAngle, xAxis, yAxis );

  Arc arc4 = creApp.NewArc( center2, radius,
    midAngle, endAngle, xAxis, yAxis );

  CurveArray topProfile = new CurveArray();

  topProfile.Append( arc3 );
  topProfile.Append( arc4 );

  Plane basePlane = creApp.NewPlane(
    normal, center );

  SketchPlane sketch = factory.NewSketchPlane(
    basePlane );

  Blend blend = factory.NewBlend( true,
    topProfile, baseProfile, sketch );

  return blend;
}
```

The command mainline simply checks that the current document is indeed a family document and then calls CreateBlend:
```python
[Transaction( TransactionMode.Automatic )]
[Regeneration( RegenerationOption.Manual )]
class CmdNewBlend : IExternalCommand
{
  public Result Execute(
    ExternalCommandData commandData,
    ref string message,
    ElementSet elements )
  {
    UIApplication app = commandData.Application;
    Document doc = app.ActiveUIDocument.Document;

    if( doc.IsFamilyDocument )
    {
      Blend blend = CreateBlend( doc );

      return Result.Succeeded;
    }
    else
    {
      message = "Please run this command "
        + "in a family document.";

      return Result.Failed;
    }
  }
}
```

This is what the resulting blend looks like in the family editor:

![New blend element](img/newblend.png)

I modified the code to create a more interesting shape using a skewed rectangle for the top profile like this:
```csharp
  CurveArray topProfile = new CurveArray();

  bool circular\_top = false;

  if( circular\_top )
  {
    // create a circular top profile:

    XYZ center2 = new XYZ( 0, 0, 1.27 );

    Arc arc3 = creApp.NewArc( center2, radius,
      startAngle, midAngle, xAxis, yAxis );

    Arc arc4 = creApp.NewArc( center2, radius,
      midAngle, endAngle, xAxis, yAxis );

    topProfile.Append( arc3 );
    topProfile.Append( arc4 );
  }
  else
  {
    // create a skewed rectangle top profile:

    XYZ[] pts = new XYZ[] {
      new XYZ(0,0,3),
      new XYZ(2,0,3),
      new XYZ(3,2,3),
      new XYZ(0,4,3)
    };

    for( int i = 0; i < 4; ++i )
    {
      topProfile.Append( creApp.NewLineBound(
        pts[0 == i ? 3 : i - 1], pts[i] ) );
    }
  }
```

Running that in a new family document based on the Metric Column template creates a shape like this:

![New blend element](img/newblend2.png)

Here is what it looks like in plan view:

![New blend element plan view](img/newblend3.png)

Here is
[version 2011.0.78.0](zip/bc_11_78.zip)
of The Building Coder samples including the complete source code and Visual Studio solution with the new command.