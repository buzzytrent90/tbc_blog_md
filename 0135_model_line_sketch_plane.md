---
post_number: "0135"
title: "Model Line Sketch Plane"
slug: "model_line_sketch_plane"
author: "Jeremy Tammik"
tags: ['csharp', 'elements', 'family', 'geometry', 'revit-api']
source_file: "0135_model_line_sketch_plane.htm"
original_url: "https://thebuildingcoder.typepad.com/blog/0135_model_line_sketch_plane.html"
---

### Model Line Sketch Plane

Model lines always reside in a sketch plane, which can give rise to some issues.
One of these issues is the question of whether it really is necessary to generate a separate sketch plane for every single model line.
Another question which recently came up is how to handle a Move request which would take the model line off the sketch plane it belongs to.
Before diving into these topics, there are two bits of recent Revit API relevant news to share.

#### Revit API Introduction Webcast Recording

The recording of the
[Revit API Introduction webcast](http://www.adskconsulting.com/adn/cs/api_course_sched.php)
held last week on Wednesday April 29th has been posted to the Developer Center and the ADN web site.

#### Updated Revit 2010 SDK

An updated version of the
[Revit 2010 SDK](http://usa.autodesk.com/adsk/servlet/index?siteID=123112&id=2484975)
is available.
The changes mostly affect the documentation.

#### Model Line must Reside in Sketch Plane

Returning to the model lines and sketch planes, here are some slightly edited musings on that by Miroslav Schonauer of Autodesk:

**Question:**
In the Revit user interface, a model line is always created on the current work plane.
Subsequently moving the line or grip-moving the endpoints always retains the line within that given plane.
This plane can be accessed via the API through the Line element SketchPlane property.

In the API, we create a new model curve using the method

```
public ModelCurve NewModelCurve(
  Curve geometryCurve,
  SketchPlane sketchPlane )
```

One might expect an exception to be thrown if the curve specified does not reside within the given SketchPlane.
This is not the case, though.
One can create any line on any plane, for instance using the Revit SDK ModelLines sample.
As an example, I created a vertical line using the first SketchPlane stored in the Revit database, which always seems to be the XY plane.

The recommended approach is to create a plane containing the desired line and use that as the sketch plane when creating the line. I would like to avoid this for two reasons:

- I want to avoid having potentially thousands of extra SketchPlane objects in my model.- I want to avoid the additional code to create or find a suitable plane for each line.

It would be convenient if we could use one single default sketch plane to host all our model lines, regardless of whether the line actually resides on that plane or not.
One would not have to care about the sketch plane, or create a sketch plane containing the line every time a new line is required.

The question is whether this behaviour is intentional or allowed by omission, in which case it would be dangerous to design any code based on these premises.
Maybe the API will throw an error in future releases, or the model will misbehave under certain circumstances.

After more experimenting in the user interface, I observe that if I grip move the endpoints of such a line after its out-of-plane API creation, the line drops onto the plane, so it starts to behave as standard in the user interface. If the line is not edited, for instance by pinning it to prevent this, the RVT file can be happily saved, re-opened and the line legally remains as created via API.

So apparently there is no absolute requirement built into the Revit geometrical parametric engine for a line to fully reside on its sketch plane. It may be different for other curve types like arcs and splines, etc. Apparently it is only the user interface layer that forces such behaviour, which of course is consistent with Revits parametric BIM concepts.

The question is, can one rely on this extended API behaviour?

**Answer:**
The answer is **no**. The NewModelCurve method should actually check if the geometry curve lies within the given sketch plane. If the curve lies outside plane, other functions relying on the sketch plane might go wrong.

In Revit 2010, when working in a mass family, you can create a CurveByPoints without having to use a sketch plane.
In any other family or project environment, however, a sketch plane is still required.

An earlier discussion on
[creating model lines](http://thebuildingcoder.typepad.com/blog/2008/11/model-line-creation.html)
provides sample code demonstrating the sketch plane creation for each model line.
Here is another method for creating a model line with an associated sketch plane to host it, also provided by Miro.
It uses the Z axis to span the sketch plane for non-vertical lines, and otherwise the Y axis:

```csharp
public static ModelLine CreateModelLine(
  Autodesk.Revit.Application app,
  XYZ p,
  XYZ q )
{
  if( p.Distance( q ) < Util.MinLineLength ) return null;
  XYZ v = q - p;

  double dxy = Math.Abs( v.X ) + Math.Abs( v.Y );

  XYZ w = ( dxy > Util.TolPointOnPlane )
    ? XYZ.BasisZ
    : XYZ.BasisY;

  XYZ norm = v.Cross( w ).Normalized;

  Plane plane = app.Create.NewPlane( norm, p );

  Autodesk.Revit.Creation.Document creDoc
    = app.ActiveDocument.Create;

  SketchPlane sketchPlane = creDoc.NewSketchPlane( plane );

  return creDoc.NewModelCurve(
    app.Create.NewLine( p, q, true ),
    sketchPlane ) as ModelLine;
}
```

With this discussion in place, let us look at a related question on moving model lines.

#### Using the Move method on model lines

**Question:**
I am attempting to use the Move method on an ElementSet containing ModelCurve instances. The Move method returns true, signifying that the elements were moved, but they are not.
Looking at the elements in 3D, they still remain at elevation 0.0, although the Move operation changed the elevation 0.0 to elevation -10.0 using the vector NewXYZ(0,0,-10).
Does Move only work of some element types?
Do I need to do something different to get it to work with ModelCurve instances?

Is there an example of using the Move method with ModelCurve instances?
Is there a better way to change the elevation of these elements?

**Answer:**
As far as I know, the Move method can be used on any Revit model elements.
However, there may be other constraints on these elements as well, which take priority over the Move request.

The case of model curves is probably one example of such behaviour.
Every model curve resides on a specific sketch plane, and cannot be moved off it.
In your case, the sketch plane hosting the model lines may well be a horizontal plane with elevation 0.0.
In that case, attempts to change the elevation of the model curves will fail, because regardless of the Z coordinate you specify, they will always be forced back into the sketch plane they reside in.

As explained above, a model curve really must lie in the sketch plane it belongs to.
An application generating model lines programmatically can take two approaches:

- The simplistic approach generates a separate sketch plane for every new model line.- A more complex approach searches for an existing sketch plane containing the new model line to use to host it, and only generates a new sketch plane if no fitting one already exists.

In any case, the application needs to ensure that the model lines generated really do lie in the sketch plane they are assigned to.

In your case, you need to check that the move operation retains the curve within the same plane.
From your description above, though, that would not appear to be the case.

Alternatively, you would have to place the model curves on a new sketch plane, which will probably require recreating them from scratch, for instance using a method similar to the ones described above.