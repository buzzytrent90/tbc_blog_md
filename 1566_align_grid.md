---
post_number: "1566"
title: "Align Grid"
slug: "align_grid"
author: "Jeremy Tammik"
tags: ['elements', 'revit-api', 'sheets', 'transactions', 'views']
source_file: "1566_align_grid.md"
original_url: "https://thebuildingcoder.typepad.com/blog/1566_align_grid.html"
---

### Aligning a Slightly Off-Axis Grid
Here is a follow-up on the recent discussion on how
to [modify a grid curve end point](http://thebuildingcoder.typepad.com/blog/2017/05/sdk-update-rvtsamples-and-modifying-grid-end-point.html#4) using
`Grid.SetCurveInView`.
In the [Revit API discussion forum](http://forums.autodesk.com/t5/revit-api-forum/bd-p/160) thread
on [off-axis grids](https://forums.autodesk.com/t5/revit-api-forum/grids-off-axis/m-p/7129065) causing
warnings in Revit, an attempt to use the same approach fails.
Instead, Fair59 presents a solution using `RotateElement` to align an almost horizontal or almost vertical grid that is slightly off-axis:
\*\*Question:\*\* I have models with `Grid` elements that produce off-axis warnings.
Snooping the `Grid.Lines` shows direction vectors that are microscopically off horizontal/vertical.
I have been unsuccessful in changing the `XYZ` `Direction`, which is read-only.
Is it possible to adjust the direction vector, or the start and end points, to resolve the off-axis error?
\*\*Answer:\*\* The solution
to [modify a grid curve end point](http://thebuildingcoder.typepad.com/blog/2017/05/sdk-update-rvtsamples-and-modifying-grid-end-point.html#4) might
be exactly what you need.
\*\*Response:\*\* That looks like precisely what is required.
However, on grids that are microscopically off-axis, it fails, saying "The curve is unbound or not coincident with the original one of the datum plane".
\*\*Answer:\*\* The exception message basically says that your new curve doesn't lie on the old curve, meaning that you can't change the direction this way.
The solution is to rotate the `Grid` with `ElementTransformUtils.RotateElement`.
\*\*Response:\*\* That would be a reasonable solution. I suspect that the error is not from user input, but a result of Revit's accuracy limitations and rounding issues. In my case the variation is microscopic, so determining the relative rotation increment, rather than being able to set the absolute angle will just push the rounding error further out. This is the off-axis grid:
![Grid off-axis error](img/grid_off_axis_error.png)
The highlighted value should be 1.0 but this `Direction` property is read-only.
\*\*Answer:\*\* It is maybe a rounding issue, but Revit gives you a warning in the Warning-Review list, so it would be prudent to correct the issue.
Here is the method `AlignOffAxisGrid` that does so:
```csharp
///
/// Align the given grid horizontally or vertically
/// if it is very slightly off axis, by Fair59 in
/// https://forums.autodesk.com/t5/revit-api-forum/grids-off-axis/m-p/7129065
/// summary>
void AlignOffAxisGrid(
Grid grid )
{
//Grid grid = doc.GetElement(
// sel.GetElementIds().FirstOrDefault() ) as Grid;
Document doc = grid.Document;
XYZ direction = grid.Curve
.GetEndPoint( 1 )
.Subtract( grid.Curve.GetEndPoint( 0 ) )
.Normalize();
double distance2hor = direction.DotProduct( XYZ.BasisY );
double distance2vert = direction.DotProduct( XYZ.BasisX );
double angle = 0;
// Maybe use another criterium then <0.0001
double max_distance = 0.0001;
if( Math.Abs( distance2hor ) < max_distance )
{
XYZ vector = direction.X < 0
? direction.Negate()
: direction;
angle = Math.Asin( -vector.Y );
}
if( Math.Abs( distance2vert ) < max_distance )
{
XYZ vector = direction.Y < 0
? direction.Negate()
: direction;
angle = Math.Asin( vector.X );
}
if( angle.CompareTo( 0 ) != 0 )
{
using( Transaction t = new Transaction( doc ) )
{
t.Start( "correctGrid" );
ElementTransformUtils.RotateElement( doc,
grid.Id,
Line.CreateBound( grid.Curve.GetEndPoint( 0 ),
grid.Curve.GetEndPoint( 0 ).Add( XYZ.BasisZ ) ),
angle );
t.Commit();
}
}
}
```
\*\*Response:\*\* Many thanks, that has indeed resolved the error.
Many thanks to Fair59 for this efficient solution!
I added it
to [The Building Coder samples](https://github.com/jeremytammik/the_building_coder_samples)
module [CmdSetGridEndpoint.cs](https://github.com/jeremytammik/the_building_coder_samples/blob/master/BuildingCoder/BuildingCoder/CmdSetGridEndpoint.cs)
in [release 2018.0.133.1](https://github.com/jeremytammik/the_building_coder_samples/releases/tag/2018.0.133.1),
cf. the [diff to the previous version](https://github.com/jeremytammik/the_building_coder_samples/compare/2018.0.133.0...2018.0.133.1).