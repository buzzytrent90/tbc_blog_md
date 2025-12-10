---
post_number: "0183"
title: "Scale a Curve"
slug: "scale_curve"
author: "Jeremy Tammik"
tags: ['csharp', 'family', 'parameters', 'revit-api']
source_file: "0183_scale_curve.htm"
original_url: "https://thebuildingcoder.typepad.com/blog/0183_scale_curve.html"
---

### Scale a Curve

Now let us get back to really practical things again, after the
[practical notes on impractical things](http://thebuildingcoder.typepad.com/blog/2009/07/practical-notes-on-impractical-things.html),
and the more or less practical
[boot camp note](http://thebuildingcoder.typepad.com/blog/2009/07/revit-boot-camp-on-mac.html).

**Question:**
I have a CurveArray object containing data defined in inches.
Now I need to scale it to feet in order to feed it into a Revit API method to create an extrusion.
How can I perform this scaling operation?
I would obviously prefer not having to rebuild the entire CurveArray.
How does one apply a scale factor to an existing CurveArray?

I tried iterating over the Curve objects in the CurveArray but cannot figure out how to change the existing X, Y and Z coordinate values of the Curve objects.

There appears to be a Multiply function for the XYZ object, which doesn't actually act on the XYZ object itself, but instead acts on and returns a new copy of the original object.
No such function appears to exist for the Curve, CurveArray or CurveArrArray objects.

Even if it did, acting on a copy wouldn't achieve the desired result.
A 'Scale' method for the XYZ, Curve, CurveArray and CurveArrArray objects, which acts on the data in those objects directly, would be a great thing to have.
It is especially important due to the fact that Revit requires the data be in one specific set of database units only, for example when submitting it to functions for doing things like creating an extrusion.

**Answer:**
Unfortunately, this is not possible in the current API.
You need to create a new curve.
Here is an idea which may at least partly alleviate the problem:

Curve.get\_Transformed() can be used to simplify the process of creating a new curve scaled as desired.

```csharp
public void ScaleCurves()
{
  CurveArray cArray = PrepareCurveArray();

  Transform x = Transform.Identity;
  x = x.ScaleBasis( 1.0 / 12.0 );

  int numCurves = cArray.Size;
  for( int i = 0; i < numCurves; ++i )
  {
    Curve curve = cArray.get\_Item( i );

    Curve newCurve = curve.get\_Transformed( x );

    cArray.set\_Item( i, newCurve );
  }

  WriteProfile( "After transformation", cArray );
}
```

You are still creating new curves, but not required to figure out how to scale each curve independently based on type.

**Reply:**
That is definitely helpful code.
I've been writing some extension methods to get around some of the limitations, essentially adding my own functions to the class definitions provided by the Revit API so my methods 'look' like they are part of the API itself.
This new method for scaling curves is much cleaner than what I came up with, so I will definitely replace my extension method with this code.

On a very vaguely related note, here is a pointer to some Revit tools for both users and developers:

#### Revit Family Tools

Version 2.3.0.0 of the
[Revit Family Tools](http://www.biggestbrains.com/revit/revitfamilytools)
is available from the
[Revit Tools](http://www.biggestbrains.com/revit)
page with the following functionality useful for all flavours of Revit:

- Comparing shared parameter files.- Merging shared parameter files.- Deleting RFA backup files, optionally recursively.- Copying CSV files created for Type Catalog usage to TXT files for easy change testing.- Reporting the versions of Revit used to create family files, optionally recursively.