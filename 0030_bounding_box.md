---
ai_optimized: true
author: Jeremy Tammik
complexity_score: 5.6
content_type: code_example
optimization_date: '2025-12-11T11:44:13.259092'
original_url: https://thebuildingcoder.typepad.com/blog/0030_bounding_box.html
post_number: '0030'
reading_time_minutes: 5
series: geometry
slug: bounding_box
source_file: 0030_bounding_box.htm
tags:
- csharp
- elements
- geometry
- levels
- parameters
- python
- revit-api
- views
- walls
- windows
title: Element Bounding Box
word_count: 902
---

### Element Bounding Box

Here is a topic suggested by
[Michael Raps](mailto:michael.raps@fhnw.ch)
of the
[Institut 4D-Technologies and DataSpaces](www.i4Ds.ch)
at the
[Fachhochschule Nordwestschweiz](www.fhnw.ch)
regarding the Element BoundingBox property.

This property is documented in the Revit API help file, which says that it retrieves a box that circumscribes all geometry of the element. It takes a parameter of type View, from the Revit Elements namespace. According to the help file, it returns a bounding box for the model geometry if the view parameter is null, and otherwise the view specific geometry, for instance in a section or cut view. If the view bounding box is not known, it will return the model box; if that is not known, it will return null.

Like several other Revit API properties, this one is not accessible in C# directly as BoundingBox. If you try to call that property on an element instance, Visual Studio will return a compile time error: Property, indexer, or event 'BoundingBox' is not supported by the language; try directly calling accessor method 'get\_BoundingBox( View )'. Accordingly, Intellisense will not display any such property, only the method listed in the error message instead.

The BoundingBoxXYZ class is also used in other contexts as well as by the Element BoundingBox property, such as for defining a section view or crop box in a 3D view.

The main properties of the bounding box class are Min, Max and Transform, to retrieve or modify its upper and lower bounds and position it in space. It has other methods as well, which are not relevant to the usage by the Element BoundingBox property.

The bounding box returned by the Element BoundingBox property is parallel to the cardinal coordinate axes in the project space, so it has no relationship to the element coordinate system and is not necessarily the smallest possible circumscribing box, which would generally not be aligned with the cardinal axes. Therefore, the Transform property of a bounding box returned in this context will always be the identity transformation.

Some more information on the Geometry BoundingBoxXYZ class is provided in section 17.3.5 of the
[Revit 2009 API Developer Guide](http://thebuildingcoder.typepad.com/blog/files/Revit_2009_API_Developer_Guide.zip).

Here is a simple command that explores the bounding box properties of a selected element:

```python
public CmdResult Execute(
  ExternalCommandData commandData,
  ref string message,
  ElementSet elements )
{
  Application app = commandData.Application;
  Document doc = app.ActiveDocument;

  Element e = Util.SelectSingleElement(
    doc, "an element" );

  if( null == e )
  {
    message = "No element selected";
    return CmdResult.Failed;
  }

  BoundingBoxXYZ b = e.get\_BoundingBox( null );

  Debug.Assert( b.Transform.IsIdentity,
    "expected identity Element bounding box transform" );

  Debug.WriteLine( string.Format(
    "Element bounding box of {0}"
    + " extends from {1} to {2}.",
    Util.ElementDescription( e ),
    Util.PointString( b.Min ),
    Util.PointString( b.Max ) ) );

  return CmdResult.Succeeded;
}
```

We examine the bounding boxes of two 10 metre long and 20 cm thick walls, i.e. a bit over 32 feet long and 0.65 thick, one parallel to the X axis, the other at 45 degrees to it. Both of them are connected to level 1 at the bottom and level 2 at the top, 4 metres high, a bit more than 13 feet:

```
Element bounding box of Walls <127876 Generic - 200mm>
  extends from (11.77,35.66,0) to (45.38,36.32,26.25).
Element bounding box of Walls <127933 Generic - 200mm>
  extends from (44.35,35.76,0) to (68.01,59.42,26.25).
```

As you can see, the bounding boxes are both aligned with the principal axes, so the one for the wall parallel to the X axis is optimal, i.e. minimal, whereas the one at a 45 degree angle is not.

Inserting a window into the horizontal wall and examining that returns:

```
Element bounding box of Windows M_Fixed <127959 0406 x 0610mm>
  extends from (27.51,35.66,3) to (28.84,37.49,5).
```

Converting the values returned from feet to metric returns more or less the expected window dimensions:

```
  inch = 2.54 cm
  foot = 12 * inch = 30.48 cm
  (28.84 - 27.51) * foot = 40.5384 cm
  2 * foot = 60.96 cm
```

I hope this helps clarify this simple Element property somewhat.

I am adding a new version 1.0.0.8 of the complete Visual Studio solution
[here](http://thebuildingcoder.typepad.com/blog/files/bc1008.zip),
including the new CmdBoundingBox class as well as all other commands discussed so far.

Die Frage von mir stammt aus einem solchen Projekt, wo der Kunde selber Bibliothekselemente aufbaut. Er hat ca. 25 Elemente mit vielen Optionen, wobei alle erlaubten Kombinationen eine Artikelnummer haben. Damit er nun in Revit seine Elemente platzieren kann und auch gleich die Optionen (im Revit Parameter) richtig gesetzt sind, mache ich ein Dialog, welcher eine Datenbank anbindet mit den Definitionen der Elemente und Optionen. Dieser Dialog setzt dann die Elemente im Revit ab. Nun sind aber die Elemente unterschiedlich breit, je nachdem welche Optionen ausgewhlt sind, und da der Dialog die gewhlten Artikel nebeneinander platzieren soll, muss ich rausfinden wie breit das Element ist (eben abhngig der gewhlten Parameter). Bei eigenen Test habe ich nun rausgefunden, dass die BoundingBox anscheinend von der Bezugsebenen "vorn", "hinten", "rechts", "links", "oben" und "unten" abhngt und nicht die Geometrie selbst abfragt. Stimmt das? D.h. dass wenn im Familieneditor die erwhnten Bezugsebenen nicht alle Teile einschliessen, dann bekomme ich nicht die BoundingBox.