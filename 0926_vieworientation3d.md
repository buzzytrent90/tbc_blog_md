---
ai_optimized: true
author: Jeremy Tammik
complexity_score: 5.1
content_type: qa
optimization_date: '2025-12-11T11:44:14.909363'
original_url: https://thebuildingcoder.typepad.com/blog/0926_vieworientation3d.html
post_number: 0926
reading_time_minutes: 4
series: views
slug: vieworientation3d
source_file: 0926_vieworientation3d.htm
tags:
- csharp
- elements
- family
- filtering
- references
- revit-api
- sheets
- views
title: Setting up your ViewOrientation3D
word_count: 716
---

### Setting up your ViewOrientation3D

Here is a contribution from Mario Guttman of
[CASE](http://case-inc.com),
who already made various contributions here in the past.
He says:

I have been purging my 2013 code of deprecated functions in preparation for my 2014 upgrade.
One group of statements I have needed to replace are the view creations.
They were previously using the Document.Create.NewView3D method and needed converting to the View3D.CreateIsometric with a separate ViewOrientation3D object defining the view direction.

Typically, in these kind of cases, I just search your blog for the answer.
In this case I found a few references, for example your March 04, 2013 item on
[What's New in the Revit 2013 API – View API – View Creation](http://thebuildingcoder.typepad.com/blog/2013/03/whats-new-in-the-revit-2013-api.html).
This, together with the
[wikihelp entry on the View3D class](http://wikihelp.autodesk.com/Revit/enu/2013/Help/00006-API_Developer's_Guide/0039-Basic_In39/0067-Views67/0069-The_View69),
describe the new syntax, but they don't really explain how you would create the values.

After searching the inner reaches of my brain for some ancient math skills I figured out the following:

Assuming that your user interface has produced two angular values (in degrees):
```csharp
  /// <summary>
  /// The angle in the XY plane (azimuth),
  /// typically 0 to 360.
  /// </summary>
  double angleHorizD;

  /// <summary>
  /// The vertical tilt (altitude),
  /// typically -90 to 90.
  /// </summary>
  double angleVertD;
```

Then this utility function returns a unit vector in the specified direction:
```csharp
  /// <summary>
  /// Return a unit vector in the specified direction.
  /// </summary>
  /// <param name="angleHorizD">Angle in XY plane
  /// in degrees</param>
  /// <param name="angleVertD">Vertical tilt between
  /// -90 and +90 degrees</param>
  /// <returns>Unit vector in the specified
  /// direction.</returns>
  private XYZ VectorFromHorizVertAngles(
    double angleHorizD,
    double angleVertD )
  {
    // Convert degreess to radians.

    double degToRadian = Math.PI \* 2 / 360;
    double angleHorizR = angleHorizD \* degToRadian;
    double angleVertR = angleVertD \* degToRadian;

    // Return unit vector in 3D

    double a = Math.Cos( angleVertR );
    double b = Math.Cos( angleHorizR );
    double c = Math.Sin( angleHorizR );
    double d = Math.Sin( angleVertR );

    return new XYZ( a \* b, a \* c, d );
  }
```

From this it is easy to create the ViewOrientation3D object as follows:
```csharp
  XYZ eye = XYZ.Zero;

  XYZ forward = VectorFromHorizVertAngles(
    angleHorizD, angleVertD );

  XYZ up = VectorFromHorizVertAngles(
    angleHorizD, angleVertD + 90 );

  ViewOrientation3D viewOrientation3D
    = new ViewOrientation3D( eye, up, forward );
```

Although it is already explained in one of your other posts, just for completeness, here is the final step is to apply the orientation to the view:
```csharp
  ViewFamilyType viewFamilyType3D
    = new FilteredElementCollector( doc )
      .OfClass( typeof( ViewFamilyType ) )
      .Cast<ViewFamilyType>()
      .FirstOrDefault<ViewFamilyType>(
        x => ViewFamily.ThreeDimensional
          == x.ViewFamily );

  View3D view3d = View3D.CreateIsometric(
    doc, viewFamilyType3D.Id );

  view3d.SetOrientation( viewOrientation3D );
```

I hope you find this useful.

Personally, I do indeed.

Many thanks to Mario for sharing!

#### Addendum

Colmag adds in
his [comment](http://thebuildingcoder.typepad.com/blog/2013/04/setting-up-your-vieworientation3d.html#comment-2904374676) below:

Thank you for sharing this info, it's made life a whole lot easier in setting 3D view orientation.

It did take me a short while to work out the values to replicate top and bottom isometric views from each corner of the view cube, so I thought I'd share the values here for others. Looking at them listed out they are pretty obvious, but faced with a blank sheet things didn't seem so straight forward!

- Horizontal Angles:

- Left Front = 45
- Front Right = 135
- Right Back = 225
- Back Left = 310

- Vertical Angles:

- -30 = Top
- 30 = Bottom

Many thanks to Colmag for this useful addition!

#### Addendum 2

K C Tang added a further simplification in
his [comment](http://thebuildingcoder.typepad.com/blog/2013/04/setting-up-your-vieworientation3d.html#comment-3137491963) below:

I found that the formula above using `( a * b, a * c, d )` to calculate the return value from `VectorFromHorizVertAngles` can be further simplified, because:

```
  ( a * b, a * c, d )
    ≡ ( a * b / a , a * c / a, d / a )
    = (b, c, d/a)
    = (b, c, e),
```

where

```
  double e = Math.Tan( angleVertR ).
```

In words, the return value from `VectorFromHorizVertAngles` can be defined as:

- cos (horizontal angle) for X
- sin (horizontal angle) for Y
- tan (vertical angle) for Z

Many thanks to K C Tang for this simplification and explanation highlighting the basic trigonometric relationships between the angles and the vectors involved so much better than the original version!