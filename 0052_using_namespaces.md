---
ai_optimized: true
author: Jeremy Tammik
complexity_score: 5.6
content_type: qa
optimization_date: '2025-12-11T11:44:13.290447'
original_url: https://thebuildingcoder.typepad.com/blog/0052_using_namespaces.html
post_number: '0052'
reading_time_minutes: 4
series: general
slug: using_namespaces
source_file: 0052_using_namespaces.htm
tags:
- csharp
- doors
- elements
- geometry
- references
- revit-api
- walls
- windows
title: Using Namespaces
word_count: 740
---

### Using Namespaces

Autodesk University is done and finished, but no rest for the wicked ... I am now on the Western European stage of the
[Devdays](http://www.autodesknews.de/2008_developer_days)
conference tour, with a first stop in London.
On the trip here from Las Vegas, I started to write about polygon area calculation and determining the outer boundary loop for floor slabs and walls for the next post or two,
and ran into another little issue that I thought might be worth discussing first.
Like many other aspects of life, it has to do with the principle of KISS ... keep it simple, stupid.
This is mostly very sound advice for every human being, and especially applicable to programming.
The temptation to introduce complexity in software development is huge and mostly detrimental.
The best solution is mostly the simplest.
One important starting point is keeping source code minimal and easy to read,
and one aspect related to readability in .NET is namespace handling.

In .NET programming, every class has a name, which is defined within a namespace.
The fully qualified class name is the class name itself together with the namespace prefix, using the dot '.' as a separator.
For instance, the entire Revit API is encapsulated in the Autodesk.Revit namespace.
That namespace defines some classes and interfaces directly, such as the CommandData class and the IExternalCommand interface.
It also defines additional nested namespaces, for instance the Geometry one, and so on.
One class inside the Autodesk.Revit.Geometry namespace is the class XYZ, whose fully qualified class name is Autodesk.Revit.Geometry.XYZ.

You may have noticed that I avoid explicitly using fully qualified class names in the code.
Instead, I make use of `using` statements in the module header, enabling me to make local use of the unqualified class names from those namespaces.
In some cases, we need to make use of a class that has an ambiguous name, i.e. two classes with the same name occur in different namespaces, and we would like to make use of both namespaces at the same time.
In this case, we can disambiguate the two classes by defining different aliases for them.
For instance, we have done this for the Element classes residing in the Autodesk.Revit and Autodesk.Revit.Geometry namespaces in the CmdWallProfile.cs module:

```csharp
using RvtElement = Autodesk.Revit.Element;
using GeoElement = Autodesk.Revit.Geometry.Element;
```

We could avoid the need for this disambiguation by avoiding making simultaneous global use of both these namespaces.
That is the approach we used so far in the Util.cs module, which currently has the following namespace header:

```csharp
using System;
using System.Collections.Generic;
using System.Diagnostics;
using Autodesk.Revit;
using Autodesk.Revit.Elements;
using Curve = Autodesk.Revit.Geometry.Curve;
using CylindricalFace = Autodesk.Revit.Geometry.CylindricalFace;
using Edge = Autodesk.Revit.Geometry.Edge;
using PlanarFace = Autodesk.Revit.Geometry.PlanarFace;
using Transform = Autodesk.Revit.Geometry.Transform;
using XYZ = Autodesk.Revit.Geometry.XYZ;
using XYZArray = Autodesk.Revit.Geometry.XYZArray;
```

Instead of including the entire Geometry namespace and disambiguating the duplicate Element class,
we defined individual aliases for each single geometry class that we make use of.
To begin with, there were just one or two of these required, but the list started growing and has now reached a size
at which I prefer to eliminate it, include the entire Geometry namespace instead, which introduces the Element class ambiguity,
and add the same disambiguation aliases for that instead, so I am replacing the lines above by

```csharp
using System;
using System.Collections.Generic;
using System.Diagnostics;
using Autodesk.Revit;
using Autodesk.Revit.Elements;
using Autodesk.Revit.Geometry;
using RvtElement = Autodesk.Revit.Element;
```

Quite a bit shorter than the list above. This initially produces an error message

'Element' is an ambiguous reference between 'Autodesk.Revit.Element' and 'Autodesk.Revit.Geometry.Element'

This expected error is obviously fixed by replacing the occurrences of Element by RvtElement.

So much for that.
As said, I am working on an area calculation algorithm for the floor slab and wall boundary loops.
Initially, we thought that the outer loop was always listed first in the Revit Face class EdgeLoops property, followed by the inner loops representing holes, i.e. openings such as shafts, doors or windows.
We now have a case where this is not true, so we will calculate each boundary loop polygon's area in order to determine which is the largest one.
Stay tuned and walk in beauty.