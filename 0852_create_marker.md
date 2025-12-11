---
ai_optimized: true
author: Jeremy Tammik
complexity_score: 1.9
content_type: code_example
optimization_date: '2025-12-11T11:44:14.737057'
original_url: https://thebuildingcoder.typepad.com/blog/0852_create_marker.html
post_number: 0852
reading_time_minutes: 2
series: general
slug: create_marker
source_file: 0852_create_marker.htm
tags:
- csharp
- geometry
- python
- references
- revit-api
- transactions
- views
title: Display a Geometry Debugging Point in the Model
word_count: 378
---

### Display a Geometry Debugging Point in the Model

Here are some pointers from a conversation I had with Daren Thomas on how to display a reference point in the model for debugging purposes that may be on general interest.

**Question:** I am playing around with the FindReferencesWithContextByDirection method and would like to display the references found as coloured blobs or something in the model.

How can I achieve this, please?

This question could also be simplified into: how to create an easily visible dot at a 3D point in the model?

Can such a "dot" exist on its own or must it be drawn onto a face? How?

I never really got into actually **creating** geometry in Revit.

**Answer** by Daren: I figured it out on my own now:

I can create a SketchPlane and draw a ModelCurve on it.

Given two points a and b, this
[RevitPythonShell](http://code.google.com/p/revitpythonshell)
code will create a green line between them in the 3D view:
```csharp
transaction = Transaction(doc)
transaction.Start('draw a line')
c = XYZ(0, 0, 0)
v0 = XYZ(a.X - b.X, a.Y - b.Y, a.Z - b.Z)
v1 = XYZ(a.X - c.X, a.Y - c.Y, a.Z - c.Z)
plane = Plane(v0.CrossProduct(v1), a)
sketchPlane = doc.Create.NewSketchPlane(plane)
line = doc.Application.Create.NewLineBound(a, b)
doc.Create.NewModelCurve(line, sketchPlane)
transaction.Commit()
```

**Answer** by Jeremy: Yes, model lines is probably the simplest way to go.

You need model curves in 3D, and can use detail curves in a 2D view.

The Creator class included in The Building Code samples includes several usage examples implemented in C#.
It was used to graphically highlight geometry in numerous previous posts, and last updated to
[display the boundaries of a slab](http://thebuildingcoder.typepad.com/blog/2012/10/slab-boundary-revisited.html).

All it does is package what you describe above nicely, though.

For more complex geometry and transient display, you can also use the analysis visualisation framework AVF.
It could be used to draw a full 3D sphere as a marker, if you prefer, as I showed when displaying
[Apollonian spheres using AVF](http://thebuildingcoder.typepad.com/blog/2012/09/apollonian-packing-of-spheres-via-web-service-and-avf.html).