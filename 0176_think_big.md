---
post_number: "0176"
title: "Think Big in Revit"
slug: "think_big"
author: "Jeremy Tammik"
tags: ['elements', 'family', 'geometry', 'parameters', 'python', 'revit-api', 'views', 'walls']
source_file: "0176_think_big.htm"
original_url: "https://thebuildingcoder.typepad.com/blog/0176_think_big.html"
---

### Think Big in Revit

A couple of cases dealing with issues concerning small dimensions in Revit were recently raised by
Toste Wallmark of
[Tecton Limited](http://www.i-tecton.com)
and
Henrik Bengtsson of
[Lindab](http://www.lindab.se):

- [Minimal length required when calling NewLineBound](#1).
- [Minimal possible family component dimension](#2).
- [Minimum scaling factor for annotation family symbol](#3).

#### Minimal Length in NewLineBound

**Question:**
I have discovered that NewLineBound throws System.ArgumentException "Value does not fall within the expected range" when the values are close or very close.
Is that an expected behaviour in this case?
How close can the line endpoints be for NewLineBound?

**Answer:** I wrote an external command with the following implementation of its Execute method and was able to verify your assertion with it:

```python
double length = 1; // foot
try
{
  Application app = commandData.Application;
  XYZ p = new XYZ();
  XYZ q = new XYZ();
  Line line;
  while( 0 < length )
  {
    length = 0.5 \* length;
    p.X = q.X = q.Y = length;

    Debug.Print(
      "Creating model line of length {0}...",
      length );

    line = app.Create.NewLineBound( p, q );
  }
  return CmdResult.Succeeded;
}
catch( Exception e )
{
  Debug.Print(
    "Creating model line of length {0} "
    + "threw exception '{1}'",
    length, e.Message );

  message = e.Message;
  return CmdResult.Failed;
}
```

This is the result of running this command in a new Revit model:

```
Creating model line of length 0.5...
Creating model line of length 0.25...
Creating model line of length 0.125...
Creating model line of length 0.0625...
Creating model line of length 0.03125...
Creating model line of length 0.015625...
Creating model line of length 0.0078125...
Creating model line of length 0.00390625...
Creating model line of length 0.001953125...

A first chance exception of type
'System.ArgumentException' occurred in RevitAPI.dll

Creating model line of length 0.001953125 threw
exception 'Value does not fall within the expected range.'
```

So apparently, you should not try to create any model lines with a length below about 0.004 feet, i.e. ca. 0.05 inches or 1.2 millimetres.

This makes pretty good sense to me in the context of an architectural model.

#### Minimal Family Component Dimension

**Question:**
When building families, the smallest parametric dimension that can be created is about 0.7 mm.
Why?
This is causing a great deal of problems for us.
We are designing profiles whose thicknesses range from 0.4 mm to 3.0 mm.
Is there any workaround to lower the minimal allowable dimension limit?

To be exact, our thin metal profiles are produced in different thicknesses. The range is between 0.4 mm and 3.0 mm.
When I am designing my product, I want the geometries to be exactly as the physical product.

In the same way, if someone is designing a hot rolled beam like an IPE270, it would really have been a problem if it was created 200 mm high or 300 mm instead of 270 mm.
This is the same type of issue.
The difference is big for us if you talk about 0.4 mm to 0.5 mm compared to 270 and 300.

With the current limitations in Revit, when I am designing our profile which is 0.7 mm, I have to artificially make it 0.8 mm instead.
I will need to create a dummy parameter for the real thickness which is simple text, and then an extra parameter that controls the thickness of the profile.
These cannot be the same and that slows me down, as well as having the problem that the profiles are designed with an inaccurate thickness.
People will get confused.

Some things in the building process are quite thin and small and that has nothing to do with tolerances really.
The tolerances are much smaller than this, but there is no need to express that.
There is a very big difference between a profile that is 0.5 mm and 0.7 mm.
The load bearing capacity will be twice as big and that is important if someone analyses the drawing in detail.
Users will wonder why it is 0.8 instead of 0.7, especially since 0.8 mm might be available from some producers.

**Answer:**
Revit in general is not intended to work with very thin elements of ca. 1 mm.
We may have something like that to support surfaces for massing functionality, but it is not possible throughout the product.

The reason is that Revit is tuned up to address building scale modelling.
Small details are incompatible with such scale, primary because they interfere with tolerances and it requires much more memory to combine in the same model very small and very large geometrical entities, with side effect of increased processing times.

One approach to such cases is to model the external shape of the object, such as a column, and use interior layers for thin materials.

Currently no such concept is applied to column, but only to wall, ceiling, floor, and roof, unless a column is joined geometry with a wall, in which case the column takes on the layer structure of the wall.

Meanwhile, detail components could be added to such columns in views in which they are cut.
Certainly this can be done through the user interface, not sure if it is possible via the API as well.

#### Minimum Annotation Family Symbol Scaling Factor

**Question:**
Annotation families cannot scale down in size when placed in views with 1:100 or smaller-sized items, e.g. 1:200, 1:500, etc.
After setting a parameter of the annotation family which makes it too small, Revit says 'Can't make type xxx", with error in Generic Annotation.

The error seems to happen when the line is shorter than 1 mm.

Is there any way to use the API to circumvent this, e.g. to scale down the annotation to an even smaller size in the view?

**Answer:**
Unfortunately, there is no known way to achieve this.
Revit is not designed to function with elements of that size.