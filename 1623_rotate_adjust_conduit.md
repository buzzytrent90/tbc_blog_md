---
post_number: "1623"
title: "Rotate Adjust Conduit"
slug: "rotate_adjust_conduit"
author: "Jeremy Tammik"
tags: ['elements', 'family', 'revit-api', 'sheets', 'transactions']
source_file: "1623_rotate_adjust_conduit.md"
original_url: "https://thebuildingcoder.typepad.com/blog/1623_rotate_adjust_conduit.html"
---

### Rotation Adjusts and Fixes Conduit End
Quite a while ago, I listed a bunch of possible approaches
to [refresh element graphics display](http://thebuildingcoder.typepad.com/blog/2014/06/refresh-element-graphics-display.html).
Here comes a new one to join the gang:
#### Applying Element Rotation to Adjust and Fix Conduit End
\*\*Question:\*\* I am creating a series of several conduit elements.
One conduit is in a different size and conduit type.
When I modify the conduit diameters before the elbow is created, one of the conduit ends exceeds the elbow.
The observed result looks like this:
![Conduit end extending too far](img/conduit_end_observed.jpg)
I am expecting it to look like this:
![Conduit end adjusted](img/conduit_end_expected.jpg)
If I remove the code to set the different size and type, and change diameters as usual, then this issue does not appear.
\*\*Answer by Paolo Serra:\*\* This happened to me as well on pipes and valves.
I fixed this after the fitting creation by applying a call to `ElementTransformUtils.Rotate` to the family instance, passing in a zero radians angle.
```csharp
using( Transaction trans = new Transaction( doc ) )
{
trans.Start( "虚假旋转线管" );
var ids = conduits.Select( c => c.Id ).ToList();
var axis = Line.CreateBound( XYZ.Zero, XYZ.BasisX );
ElementTransformUtils.RotateElements( doc, ids, axis, 0 );
trans.Commit();
}
```