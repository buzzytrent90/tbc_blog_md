---
post_number: "1240"
title: "Rotation by π and NewSweptBlend Using Arcs"
slug: "newsweptblend_arc"
author: "Jeremy Tammik"
tags: ['csharp', 'elements', 'family', 'filtering', 'geometry', 'levels', 'python', 'revit-api', 'transactions', 'views']
source_file: "1240_newsweptblend_arc.htm"
original_url: "https://thebuildingcoder.typepad.com/blog/1240_newsweptblend_arc.html"
---

### Rotation by π and NewSweptBlend Using Arcs

Let me address two questions concerning form generation in family documents raised by Alex Hearn:

- [Rotation by π](#2)
- [NewSweptBlend using arcs](#3)

Among other things, the answers also demonstrate some trivial migration steps of the form generation code from Revit 2012 to 2015 and, yet again, the occasional
[crucial importance of regeneration](http://thebuildingcoder.typepad.com/blog/about-the-author.html#5.33).

#### Rotation by π

**Question:** I am exploring and really enjoying your blog site.
Thanks for all the detailed information.

I have been attempting to build a Revit translator that takes in model data from SolidWorks (my usual programming environment) and redraws the elements in Revit. The idea is that I can then recreate SolidWorks models in Revit.

I have come across a very odd behaviour in ElementTransformUtils.RotateElement and I was wondering if you have ever seen it or have any advice. I’m using Revit 2012 with C#. Here’s some sample code:

```csharp
  Autodesk.Revit.Creation.Application creapp
    = fdoc.Application.Create;

  XYZ normal = XYZ.BasisZ;

  SketchPlane sketchPlane
    = CreateSketchPlane( normal, XYZ.Zero );

  XYZ p0 = XYZ.Zero;
  XYZ p1 = new XYZ( 10, 0, 0 );
  XYZ p2 = new XYZ( 10, 10, 0 );
  XYZ p3 = new XYZ( 0, 10, 0 );
  XYZ p4 = new XYZ( 10, 20, 0 );
  Line line1 = creapp.NewLineBound( p0, p1 );
  Line line2 = creapp.NewLineBound( p1, p2 );
  Line line3 = creapp.NewLineBound( p2, p3 );
  Line line4 = creapp.NewLineBound( p3, p0 );
  Line line5 = creapp.NewLineBound( p2, p4 );
  CurveArray curveArray1 = new CurveArray();
  curveArray1.Append( line1 );
  curveArray1.Append( line2 );
  curveArray1.Append( line3 );
  curveArray1.Append( line4 );
  CurveArrArray curveArrArray = new CurveArrArray();
  curveArrArray.Append( curveArray1 );

  Revolution aRevolution
    = fdoc.FamilyCreate.NewRevolution( true,
      curveArrArray, sketchPlane, line5,
      Math.PI / -2, 0 );

  // THE FOLLOWING DOES NOT WORK!

  ElementTransformUtils.RotateElement(
    fdoc, aRevolution.Id, line5, Math.PI );
```

The problem is apparently with the Math.PI value used by RotateElement. I have messed around with it and it seems that you ***cannot*** use a value that is greater than ***or equal to*** Math.PI for this argument. As a workaround, I am using Math.PI \* 0.995 which means the resulting component is veeeery slightly off from the true square.

Any suggestions welcome but don’t be too concerned because as I’ve said above I do have an acceptable (for now) workaround.

**Answer:** The following implementation based on your sample code migrated to Revit 2015 works perfectly well for me:

```python
[Transaction( TransactionMode.Manual )]
public class Command : IExternalCommand
{
  public Result Execute(
    ExternalCommandData commandData,
    ref string message,
    ElementSet elements )
  {
    UIApplication uiapp = commandData.Application;
    UIDocument uidoc = uiapp.ActiveUIDocument;
    Application app = uiapp.Application;
    Document doc = uidoc.Document;

    XYZ p0 = XYZ.Zero;
    XYZ p1 = new XYZ( 10, 0, 0 );
    XYZ p2 = new XYZ( 10, 10, 0 );
    XYZ p3 = new XYZ( 0, 10, 0 );
    XYZ p4 = new XYZ( 10, 20, 0 );
    Line line1 = Line.CreateBound( p0, p1 );
    Line line2 = Line.CreateBound( p1, p2 );
    Line line3 = Line.CreateBound( p2, p3 );
    Line line4 = Line.CreateBound( p3, p0 );
    Line line5 = Line.CreateBound( p2, p4 );
    CurveArray curveArray1 = new CurveArray();
    curveArray1.Append( line1 );
    curveArray1.Append( line2 );
    curveArray1.Append( line3 );
    curveArray1.Append( line4 );
    CurveArrArray curveArrArray = new CurveArrArray();
    curveArrArray.Append( curveArray1 );

    using( Transaction tx = new Transaction( doc ) )
    {
      tx.Start( "Create Revolution and Rotate" );

      //XYZ normal = XYZ.BasisZ;
      //Plane plane = new Plane( normal, XYZ.Zero );
      //SketchPlane sketchPlane = SketchPlane.Create( doc, plane );

      SketchPlane sketchPlane
        = new FilteredElementCollector( doc )
          .OfClass( typeof( SketchPlane ) )
          .Cast<SketchPlane>()
          .Where<SketchPlane>( x
            => x.Name.Equals( "Ref. Level" ) )
          .FirstOrDefault<SketchPlane>();

      Revolution aRevolution
        = doc.FamilyCreate.NewRevolution(
          true, curveArrArray, sketchPlane,
          line5, Math.PI / -2, 0 );

      doc.Regenerate();

      ElementTransformUtils.RotateElement(
        doc, aRevolution.Id, line5, Math.PI ); // WORKs FINE!

      tx.Commit();
    }
    return Result.Succeeded;
  }
}
```

There are only a few differences, besides the fact that I migrated the code from Revit 2012 to 2015.

I also replaced the generation of a new geometric plane and sketch plane by a filtered element collector retrieving the existing sketch plane from the family document instead.

Most significantly: I added a call to
[regenerate the document](http://thebuildingcoder.typepad.com/blog/about-the-author.html#5.33) before
calling the RotateElement method.

Please note that it takes an element id as an argument. That is a hint that it relies on the element being a valid resident of the Revit database, which it does not become until after it has been regenerated.

Here is the result of running this command:

![Revolution rotated by π](img/rotate_by_pi.png)

#### NewSweptBlend Using Arcs

The next query comes from this
[comment](http://thebuildingcoder.typepad.com/blog/2014/10/adva-webinar-free-student-software-and-au.html?cid=6a00e553e16897883301bb07a94cba970d#comment-6a00e553e16897883301bb07a94cba970d):

**Question:** Thank you for your response. I have another issue, this time with SweptBlend.

Is it possible to have the sweep path be an arc?
I would think yes, but I cannot get it to work.
All the examples show the path as a line.

Here’s my code:

```csharp
  XYZ pnt1 = new XYZ( 0, -1, 0 );
  XYZ pnt2 = new XYZ( 1, 0, 0 );
  XYZ pnt3 = new XYZ( 0, 1, 0 );
  XYZ pnt4 = new XYZ( -1, 0, 0 );
  Arc aArc1 = creapp.NewArc( pnt1, pnt3, pnt2 );
  Arc aArc2 = creapp.NewArc( pnt3, pnt1, pnt4 );
  CurveArrArray arrarr1 = new CurveArrArray();

  SweepProfile bottomProfile
    = creapp.NewCurveLoopsProfile( arrarr1 );

  CurveArray arr1 = new CurveArray();
  arr1.Append( aArc1 );
  arr1.Append( aArc2 );
  XYZ pnt6 = new XYZ( 0, -2, 0 );
  XYZ pnt7 = new XYZ( 2, 0, 0 );
  XYZ pnt8 = new XYZ( 0, 2, 0 );
  XYZ pnt9 = new XYZ( -2, 0, 0 );
  Arc aArc3 = creapp.NewArc( pnt6, pnt8, pnt7 );
  Arc aArc4 = creapp.NewArc( pnt8, pnt6, pnt9 );
  CurveArrArray arrarr2 = new CurveArrArray();
  CurveArray arr2 = new CurveArray();
  arr2.Append( aArc3 );
  arr2.Append( aArc4 );
  arrarr2.Append( arr2 );

  SweepProfile topProfile
    = creapp.NewCurveLoopsProfile( arrarr2 );

  XYZ pnt10 = new XYZ( 0, 0, 0 );
  XYZ pnt11 = new XYZ( 0, 5, 0 );
  XYZ pnt122 = new XYZ( 2.5, 2.5, 0 );
  Arc testArc = creapp.NewArc( pnt10, pnt11, pnt122 );
  Curve curve = (Curve) testArc;

  Plane geometryPlane = creapp.NewPlane(
    XYZ.BasisZ, XYZ.Zero );

  SketchPlane sketchPlane = doc.NewSketchPlane(
    geometryPlane );

  SweptBlend aSweptBlend = doc.NewSweptBlend(
    true, curve, sketchPlane, bottomProfile,
    topProfile );
```

**Answer:** The curve defining the swept blend extrusion path can be any valid curve and is definitely not limited to just straight lines.

Just ensure that you are not generating any self-intersecting solids.

If you run into any problems, remove the final solid generation call, replace it by code to generate model curves instaed, and see what happens when you try to generate the same solid manually in the user interface, as described for
[debugging geometric form creation](http://thebuildingcoder.typepad.com/blog/2009/07/debug-geometric-form-creation.html).

The top and bottom profiles must be planar and lie in the XY plane.

I added a new helper method CreateNewSweptBlendArc to the existing CmdNewSweptBlend external command in The Building Coder samples to create a second solid shape to answer your query.

This external command implementation was still using automatic transaction mode until now, so I went ahead and converted it to use manual transaction mode, while I was at it anyway.

Here is the new CreateNewSweptBlendArc method based on your sample code:

```csharp
  /// <summary>
  /// Create a new swept blend form using arcs to
  /// define circular start and end profiles and an
  /// arc path. The NewSweptBlend method requires
  /// the input profiles to be in the XY plane.
  /// </summary>
  public void CreateNewSweptBlendArc( Document doc )
  {
    Debug.Assert( doc.IsFamilyDocument,
      "this method will only work in a family document" );

    Application app = doc.Application;

    Autodesk.Revit.Creation.Application creapp
      = app.Create;

    Autodesk.Revit.Creation.FamilyItemFactory credoc
      = doc.FamilyCreate;

    XYZ px = XYZ.BasisX;
    XYZ py = XYZ.BasisY;
    Arc arc1 = Arc.Create( -px, px, -py );
    Arc arc2 = Arc.Create( px, -px, py );
    CurveArray arr1 = new CurveArray();
    arr1.Append( arc1 );
    arr1.Append( arc2 );
    CurveArrArray arrarr1 = new CurveArrArray();
    arrarr1.Append( arr1 );

    SweepProfile bottomProfile
      = creapp.NewCurveLoopsProfile( arrarr1 );

    px += px;
    py += py;
    Arc arc3 = Arc.Create( -px, px, -py );
    Arc arc4 = Arc.Create( px, -px, py );
    CurveArray arr2 = new CurveArray();
    arr2.Append( arc3 );
    arr2.Append( arc4 );
    CurveArrArray arrarr2 = new CurveArrArray();
    arrarr2.Append( arr2 );

    SweepProfile topProfile
      = creapp.NewCurveLoopsProfile( arrarr2 );

    XYZ p0 = XYZ.Zero;
    XYZ p5 = 5 \* XYZ.BasisY;
    XYZ pmid = new XYZ( 2.5, 2.5, 0 );
    Arc testArc = Arc.Create( p0, p5, pmid );

    Plane geometryPlane = creapp.NewPlane(
      XYZ.BasisZ, XYZ.Zero );

    SketchPlane sketchPlane = SketchPlane.Create(
      doc, geometryPlane );

    SweptBlend aSweptBlend = credoc.NewSweptBlend(
      true, testArc, sketchPlane, bottomProfile,
      topProfile );
  }
```

This creates a swept blend with circular start and end profiles with radius one and two respectively, swept along an arc with a diameter of five.

I added the call to this method to the existing one that generates a shape with straight edges, which I left unchanged from the original CmdNewSweptBlend implementation.

I fixed a few obvious small but crucial errors in your code, such as mixing up the endpoint and the midpoint in the calls to generate some of the arcs and passing in an empty CurveArrArray arrarr1 as an argument to the NewCurveLoopsProfile method when defining the SweepProfile bottomProfile...

Here is a screen snapshot of the two swept blends now generated by this command:

![New swept blend using arcs](img/newsweptblendarc.png)

Here is another snapshot from a different point of view:

![New swept blend using arcs](img/newsweptblendarc2.png)

As always, the most up to date version of The Building Coder samples is provided in
[its GitHub repository](https://github.com/jeremytammik/the_building_coder_samples),
and the version described above is
[release 2015.0.115.2](https://github.com/jeremytammik/the_building_coder_samples/releases/tag/2015.0.115.2).