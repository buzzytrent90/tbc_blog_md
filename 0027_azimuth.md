---
ai_optimized: true
author: Jeremy Tammik
complexity_score: 3.5
content_type: code_example
optimization_date: '2025-12-11T11:44:13.253635'
original_url: https://thebuildingcoder.typepad.com/blog/0027_azimuth.html
post_number: '0027'
reading_time_minutes: 4
series: general
slug: azimuth
source_file: 0027_azimuth.htm
tags:
- elements
- filtering
- geometry
- python
- revit-api
- selection
- walls
- windows
title: Azimuth
word_count: 768
---

### Azimuth

Several developers have asked how to determine the
[azimuth](http://en.wikipedia.org/wiki/Azimuth)
of an element, i.e. the angle between the element and true north.
This calculation is obviously strongly affected by the
[project location](http://thebuildingcoder.typepad.com/blog/2010/01/project-location.html).

Determining the angle of an element is pretty simple.
All Revit elements have a Location property.
For some, the location is a point, while others can be line or curve based, in which case their location is an instance of the LocationCurve class.
One of the properties of the LocationCurve is Curve, which returns the actual geometry curve, which may be one of the derived classes Arc, Line, Ellipse or NurbSpline.
In addition to providing access to the underlying geometry curve, the location curve class also provides methods to move and rotate the element whose location it defines, and to determine or modify the order of the neighbouring elements joining to the end of the element's location curve.

In order to determine the angle of an element, you can convert the two endpoints of its location curve to a vector and then measure the angle of that vector relative to some other vector, for instance the X axis or the north direction.

Revit stores true north in the ProjectPosition object stored in the ProjectLocation, accessible through the ProjectLocations property on the document object. You can query the ProjectPosition.Angle property, which returns the angle difference between the project north and true north measured in radians. It can have a value from -π to +π.

To determine the element azimuth, you can determine the element angle and the angle between project north and true north, and then simply add the two.

Some notes on the Revit geometry angle methods:
The Angle() method of the XYZ class treats its XYZ instances as vectors, not points.
This is what the Revit API help has to say about this:

> The XYZ.Angle() method returns the angle between this vector and the specified vector.
>
> Return Value: The real number between 0 and π equal to the angle between the two vectors.
>
> Remarks: The angle between the two vectors is measured in the plane spanned by them.

If you are interested in angles around the full circle, i.e. in the interval [0, 2π], you should use AngleTo() instead of Angle(), which returns a value in the interval [0, π].

Here is a code example measuring various kinds of element angles and printing the project north deviation:

```python
class CmdAzimuth : IExternalCommand
{
  Element SelectSingleElement( Document doc )
  {
    Selection sel = doc.Selection;
    Element e = null;
    sel.Elements.Clear();
    sel.StatusbarTip = "Please select a line";
    if( sel.PickOne() )
    {
      ElementSetIterator elemSetItr
        = sel.Elements.ForwardIterator();
      elemSetItr.MoveNext();
      e = elemSetItr.Current as Element;
    }
    return e;
  }

  public CmdResult Execute(
    ExternalCommandData commandData,
    ref String message,
    ElementSet elements )
  {
    Application app = commandData.Application;
    Document doc = app.ActiveDocument;
    Element e = SelectSingleElement( doc );

    if( null == e )
    {
      message = "No element selected";
      return CmdResult.Failed;
    }

    LocationCurve curve
      = e.Location as LocationCurve;

    if( null == curve )
    {
      message = "No curve available";
      return CmdResult.Failed;
    }

    XYZ p = curve.Curve.get\_EndPoint( 0 );
    XYZ q = curve.Curve.get\_EndPoint( 1 );

    Debug.WriteLine( "Start point "
      + Util.PointString( p ) );

    Debug.WriteLine( "End point "
      + Util.PointString( q ) );

    double a = p.Angle( q );
    Debug.WriteLine(
      "Angle between start and end points = "
      + Util.AngleString( a ) );

    XYZ v = q - p;
    a = XYZ.BasisX.Angle( v );
    Debug.WriteLine(
      "Angle between points measured from X axis = "
      + Util.AngleString( a ) );

    XYZ z = XYZ.BasisZ;
    a = XYZ.BasisX.AngleAround( v, z );
    Debug.WriteLine(
      "Angle around measured from X axis = "
      + Util.AngleString( a ) );

    foreach( ProjectLocation location
      in doc.ProjectLocations )
    {
      ProjectPosition projectPosition
        = location.get\_ProjectPosition( XYZ.Zero );
      double pna = projectPosition.Angle;
      Debug.WriteLine(
        "Angle between project north and true north "
        + Util.AngleString( pna ) );
    }
    return CmdResult.Succeeded;
  }
}
```

Running this command in a project and selecting a model line with an angle of 35 degrees to the X axis generates the following result in the Visual Studio debug output window:

```
Start point (-43.84,31.27,0)
End point (-28.79,41.81,0)
Angle between start and end points = 19.95 degrees
Angle between points measured from X axis = 35 degrees
Angle around measured from X axis = 35 degrees
Angle between project north and true north 0 degrees
```

You might also have a look at the Revit SDK SharedCoordinateSystem sample to see another example of using the ProjectPosition object.

I am adding the complete Visual Studio solution [here](http://thebuildingcoder.typepad.com/blog/files/bc1007.zip). This version 1.0.0.7 includes all commands discussed so far: CmdListWalls, CmdRelationshipInverter, CmdWallDimensions, CmdFilterPerformance, CmdGetMaterials and the new CmdAzimuth.