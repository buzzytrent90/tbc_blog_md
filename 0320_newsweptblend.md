---
ai_optimized: true
author: Jeremy Tammik
complexity_score: 3.4
content_type: qa
optimization_date: '2025-12-11T11:44:13.739314'
original_url: https://thebuildingcoder.typepad.com/blog/0320_newsweptblend.html
post_number: '0320'
reading_time_minutes: 4
series: general
slug: newsweptblend
source_file: 0320_newsweptblend.htm
tags:
- csharp
- family
- revit-api
title: NewSweptBlend
word_count: 740
---

### NewSweptBlend

I am still awfully busy preparing for upcoming Revit API trainings and should be working more for that and less on the blog, actually.
Anyway, here an important nugget if information from another case handled by my colleagues Phil Xia and Saikat Bhattacharya on how to set up the input profile for the NewSweptBlend method.
We already described the recommended approach for
[debugging form creation problems](http://thebuildingcoder.typepad.com/blog/2009/07/debug-geometric-form-creation.html),
and it obviously applies to the NewSweptBlend method as well, but the information provided here concerning the input curves to the NewSweptBlend method is required in addition to that and not provided in the Revit API documentation.

**Question:** The following method CreateNewSweptBlend based on the GenericModelCreation SDK sample CreateGenericModel method creates a valid swept blend form:
```csharp
public void CreateNewSweptBlend( Document doc )
{
  if( !doc.IsFamilyDocument )
  {
    Util.ErrorMsg(
      "Please run this command in a family document." );
  }
  Application app = doc.Application;

  Autodesk.Revit.Creation.Application creapp
    = app.Create;

  CurveArrArray curvess0
    = creapp.NewCurveArrArray();

  CurveArray curves0 = new CurveArray();

  XYZ p00 = creapp.NewXYZ( 0, 7.5, 0 );
  XYZ p01 = creapp.NewXYZ( 0, 15, 0 );

  // changing Z to 1 in the following line fails:

  XYZ p02 = creapp.NewXYZ( -1, 10, 0 );

  curves0.Append( creapp.NewLineBound( p00, p01 ) );
  curves0.Append( creapp.NewLineBound( p01, p02 ) );
  curves0.Append( creapp.NewLineBound( p02, p00 ) );
  curvess0.Append( curves0 );

  CurveArrArray curvess1 = creapp.NewCurveArrArray();
  CurveArray curves1 = new CurveArray();

  XYZ p10 = creapp.NewXYZ( 7.5, 0, 0 );
  XYZ p11 = creapp.NewXYZ( 15, 0, 0 );

  // changing the Z value in the following line fails:

  XYZ p12 = creapp.NewXYZ( 10, -1, 0 );

  curves1.Append( creapp.NewLineBound( p10, p11 ) );
  curves1.Append( creapp.NewLineBound( p11, p12 ) );
  curves1.Append( creapp.NewLineBound( p12, p10 ) );
  curvess1.Append( curves1 );

  SweepProfile sweepProfile0
    = creapp.NewCurveLoopsProfile( curvess0 );

  SweepProfile sweepProfile1
    = creapp.NewCurveLoopsProfile( curvess1 );

  XYZ pnt10 = new XYZ( 5, 0, 0 );
  XYZ pnt11 = new XYZ( 0, 20, 0 );
  Curve curve = creapp.NewLineBound( pnt10, pnt11 );

  XYZ normal = XYZ.BasisZ;

  SketchPlane splane = CreateSketchPlane(
    doc, normal, XYZ.Zero );

  try
  {
    SweptBlend sweptBlend = doc.FamilyCreate.NewSweptBlend(
      true, curve, splane, sweepProfile0, sweepProfile1 );
  }
  catch( Exception ex )
  {
    Util.ErrorMsg( "NewSweptBlend exception: " + ex.Message );
  }
  return;
}
```

But if I change the Z coordinate value of points p02 and p12 to 1 instead of 0, it throws an invalid operation exception.
Which rule is being broken when I change the Z value of both p02 and p12, please?
Does the curve have to be perpendicular to the sweep profiles?

**Answer:** First of all, here is a summary of the suggestion we already made for
[debugging form creation problems](http://thebuildingcoder.typepad.com/blog/2009/07/debug-geometric-form-creation.html):
one way to investigate issues like this is to change the code so that it creates model lines instead of only internal lines and remove the call to NewSweptBlend.
Then you can try selecting the model lines to manually create the blend in the user interface.
If the operation succeeds through the UI and not through the API, that would indicate a problem in the API.

In this case, the problem is neither in the user interface nor the API, but simply in the definition of the input curves and the NewSweptBlend method documentation.
The Revit API help file RevitAPI.chm states the following requirements for the two input profiles bottomProfile and topProfile:

- Type: Autodesk.Revit.DB.SweepProfile.- It should consist of only one curve loop.- The input profile must be in one plane.

After some research and discussion, we determined that the input profile curve loop must be in the XY plane, implying that the Z coordinates must all be zero.
Revit will calculate the profile planes with the input path curve and then transforms the XY plane curve loop to the right profile plane internally.
In other words, you need to define the curve profiles to be 2D and located in the XY plane, i.e. set the XYZ input points' Z coordinates to zero.
Unfortunately, the documentation does not mention the XY plane, it just says that the curves should be in one plane.
Once I set all the Z coordinates to zero, it works fine.

Since we did not previously have any examples of calls to NewSeptBlend in the Building Coder samples, I added a new command CmdNewSweptBlend to it.
Here is
[version 1.1.0.64](zip/bc11064.zip)
of the complete Building Coder source code and Visual Studio solution including the new CmdNewSweptBlend external command.

Many thanks to Phil and Saikat for this solution!