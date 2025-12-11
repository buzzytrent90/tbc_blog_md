---
ai_optimized: true
author: Jeremy Tammik
complexity_score: 3.3
content_type: qa
optimization_date: '2025-12-11T11:44:16.410962'
original_url: https://thebuildingcoder.typepad.com/blog/1624_brepbuilder.html
post_number: '1624'
reading_time_minutes: 10
series: general
slug: brepbuilder
source_file: 1624_brepbuilder.md
tags:
- elements
- geometry
- revit-api
- sheets
- transactions
- views
title: Brepbuilder
word_count: 1917
---

### DirectShape from BrepBuilder and Boolean
Here is an interesting code snippet illustrating the use of `BRepBuilder` and Boolean operations to generate a `DirectShape`.
It might come in useful somewhere, even though this approach is non-optimal to address the task at hand, as explained below.
\*\*Question:\*\* I want to create a `DirectShape` object which is cut by void geometry.
I found that
the [`BRepBuilder` constructor](http://www.revitapidocs.com/2018.1/b3eb95b6-2297-44dc-df94-38aed1940b8c.htm) accepts
both `BRepType.Void` and `BRepType.Solid`, so I tested if I could use that.
However, my attempts so far lead to an internal error.
Is there any mistake in my test code?
Does the `DirectShape.AppendShape` method not accept a void shape?
![DirectShape from BrepBuilder Boolean](img/directshape_brepbuilder_boolean_2.png)
\*\*Answer:\*\* It seems to me that you actually want to do is create two solid geometries, and then use a Boolean operation to subtract one from the other.
This will give the desired shape.
Taking a look at the code, I notice that the function `CreateBrepVoid` is incorrect – it tells the `BRepBuilder` that it wants to create a void, but the geometry defines a solid, not a void. This is determined by the face and edge loop orientations. I assume that `BRepBuilder` complains about invalid input at some point in the process.
The orientation conventions are discussed in the descriptions of
BRepBuilder [AddFace](http://www.revitapidocs.com/2018.1/cb899f6d-c4e0-0983-ab70-bae0a620dc8d.htm)
and [AddCoEdge](http://www.revitapidocs.com/2018.1/c4713a48-712b-e293-6745-a266af97e195.htm) methods.
Please note that `BRepBuilder` wasn’t really meant for 'manually' constructing geometry. Its interface is very cumbersome for that purpose. It was meant for translating existing geometry into Revit, with rather thorough validation of the input geometry.
Here is the code that constructs two solid geometries and then applies the Boolean operation to find the difference.
The code constructs the desired `DirectShape` shown in the image above:
```csharp
[Transaction( TransactionMode.Manual )]
class CmdBrepBuilder : IExternalCommand
{
///
/// Create a cube 100 x 100 x 100, from
/// (0,0,0) to (100, 100, 100).
/// summary>
public BRepBuilder CreateBrepSolid()
{
BRepBuilder b = new BRepBuilder( BRepType.Solid );
// 1. Planes.
// naming convention for faces and planes:
// We are looking at this cube in an isometric view.
// X is down and to the left of us, Y is horizontal
// and points to the right, Z is up.
// front and back faces are along the X axis, left
// and right are along the Y axis, top and bottom
// are along the Z axis.
Plane bottom = Plane.CreateByOriginAndBasis( new XYZ( 50, 50, 0 ), new XYZ( 1, 0, 0 ), new XYZ( 0, 1, 0 ) ); // bottom. XY plane, Z = 0, normal pointing inside the cube.
Plane top = Plane.CreateByOriginAndBasis( new XYZ( 50, 50, 100 ), new XYZ( 1, 0, 0 ), new XYZ( 0, 1, 0 ) ); // top. XY plane, Z = 100, normal pointing outside the cube.
Plane front = Plane.CreateByOriginAndBasis( new XYZ( 100, 50, 50 ), new XYZ( 0, 0, 1 ), new XYZ( 0, 1, 0 ) ); // front side. ZY plane, X = 0, normal pointing inside the cube.
Plane back = Plane.CreateByOriginAndBasis( new XYZ( 0, 50, 50 ), new XYZ( 0, 0, 1 ), new XYZ( 0, 1, 0 ) ); // back side. ZY plane, X = 0, normal pointing outside the cube.
Plane left = Plane.CreateByOriginAndBasis( new XYZ( 50, 0, 50 ), new XYZ( 0, 0, 1 ), new XYZ( 1, 0, 0 ) ); // left side. ZX plane, Y = 0, normal pointing inside the cube
Plane right = Plane.CreateByOriginAndBasis( new XYZ( 50, 100, 50 ), new XYZ( 0, 0, 1 ), new XYZ( 1, 0, 0 ) ); // right side. ZX plane, Y = 100, normal pointing outside the cube
// 2. Faces.
BRepBuilderGeometryId faceId_Bottom = b.AddFace( BRepBuilderSurfaceGeometry.Create( bottom, null ), true );
BRepBuilderGeometryId faceId_Top = b.AddFace( BRepBuilderSurfaceGeometry.Create( top, null ), false );
BRepBuilderGeometryId faceId_Front = b.AddFace( BRepBuilderSurfaceGeometry.Create( front, null ), true );
BRepBuilderGeometryId faceId_Back = b.AddFace( BRepBuilderSurfaceGeometry.Create( back, null ), false );
BRepBuilderGeometryId faceId_Left = b.AddFace( BRepBuilderSurfaceGeometry.Create( left, null ), true );
BRepBuilderGeometryId faceId_Right = b.AddFace( BRepBuilderSurfaceGeometry.Create( right, null ), false );
// 3. Edges.
// 3.a (define edge geometry)
// walk around bottom face
BRepBuilderEdgeGeometry edgeBottomFront = BRepBuilderEdgeGeometry.Create( new XYZ( 100, 0, 0 ), new XYZ( 100, 100, 0 ) );
BRepBuilderEdgeGeometry edgeBottomRight = BRepBuilderEdgeGeometry.Create( new XYZ( 100, 100, 0 ), new XYZ( 0, 100, 0 ) );
BRepBuilderEdgeGeometry edgeBottomBack = BRepBuilderEdgeGeometry.Create( new XYZ( 0, 100, 0 ), new XYZ( 0, 0, 0 ) );
BRepBuilderEdgeGeometry edgeBottomLeft = BRepBuilderEdgeGeometry.Create( new XYZ( 0, 0, 0 ), new XYZ( 100, 0, 0 ) );
// now walk around top face
BRepBuilderEdgeGeometry edgeTopFront = BRepBuilderEdgeGeometry.Create( new XYZ( 100, 0, 100 ), new XYZ( 100, 100, 100 ) );
BRepBuilderEdgeGeometry edgeTopRight = BRepBuilderEdgeGeometry.Create( new XYZ( 100, 100, 100 ), new XYZ( 0, 100, 100 ) );
BRepBuilderEdgeGeometry edgeTopBack = BRepBuilderEdgeGeometry.Create( new XYZ( 0, 100, 100 ), new XYZ( 0, 0, 100 ) );
BRepBuilderEdgeGeometry edgeTopLeft = BRepBuilderEdgeGeometry.Create( new XYZ( 0, 0, 100 ), new XYZ( 100, 0, 100 ) );
// sides
BRepBuilderEdgeGeometry edgeFrontRight = BRepBuilderEdgeGeometry.Create( new XYZ( 100, 100, 0 ), new XYZ( 100, 100, 100 ) );
BRepBuilderEdgeGeometry edgeRightBack = BRepBuilderEdgeGeometry.Create( new XYZ( 0, 100, 0 ), new XYZ( 0, 100, 100 ) );
BRepBuilderEdgeGeometry edgeBackLeft = BRepBuilderEdgeGeometry.Create( new XYZ( 0, 0, 0 ), new XYZ( 0, 0, 100 ) );
BRepBuilderEdgeGeometry edgeLeftFront = BRepBuilderEdgeGeometry.Create( new XYZ( 100, 0, 0 ), new XYZ( 100, 0, 100 ) );
// 3.b (define the edges themselves)
BRepBuilderGeometryId edgeId_BottomFront = b.AddEdge( edgeBottomFront );
BRepBuilderGeometryId edgeId_BottomRight = b.AddEdge( edgeBottomRight );
BRepBuilderGeometryId edgeId_BottomBack = b.AddEdge( edgeBottomBack );
BRepBuilderGeometryId edgeId_BottomLeft = b.AddEdge( edgeBottomLeft );
BRepBuilderGeometryId edgeId_TopFront = b.AddEdge( edgeTopFront );
BRepBuilderGeometryId edgeId_TopRight = b.AddEdge( edgeTopRight );
BRepBuilderGeometryId edgeId_TopBack = b.AddEdge( edgeTopBack );
BRepBuilderGeometryId edgeId_TopLeft = b.AddEdge( edgeTopLeft );
BRepBuilderGeometryId edgeId_FrontRight = b.AddEdge( edgeFrontRight );
BRepBuilderGeometryId edgeId_RightBack = b.AddEdge( edgeRightBack );
BRepBuilderGeometryId edgeId_BackLeft = b.AddEdge( edgeBackLeft );
BRepBuilderGeometryId edgeId_LeftFront = b.AddEdge( edgeLeftFront );
// 4. Loops.
BRepBuilderGeometryId loopId_Bottom = b.AddLoop( faceId_Bottom );
BRepBuilderGeometryId loopId_Top = b.AddLoop( faceId_Top );
BRepBuilderGeometryId loopId_Front = b.AddLoop( faceId_Front );
BRepBuilderGeometryId loopId_Back = b.AddLoop( faceId_Back );
BRepBuilderGeometryId loopId_Right = b.AddLoop( faceId_Right );
BRepBuilderGeometryId loopId_Left = b.AddLoop( faceId_Left );
// 5. Co-edges.
// Bottom face. All edges reversed
b.AddCoEdge( loopId_Bottom, edgeId_BottomFront, true ); // other direction in front loop
b.AddCoEdge( loopId_Bottom, edgeId_BottomLeft, true ); // other direction in left loop
b.AddCoEdge( loopId_Bottom, edgeId_BottomBack, true ); // other direction in back loop
b.AddCoEdge( loopId_Bottom, edgeId_BottomRight, true ); // other direction in right loop
b.FinishLoop( loopId_Bottom );
b.FinishFace( faceId_Bottom );
// Top face. All edges NOT reversed.
b.AddCoEdge( loopId_Top, edgeId_TopFront, false ); // other direction in front loop.
b.AddCoEdge( loopId_Top, edgeId_TopRight, false ); // other direction in right loop
b.AddCoEdge( loopId_Top, edgeId_TopBack, false ); // other direction in back loop
b.AddCoEdge( loopId_Top, edgeId_TopLeft, false ); // other direction in left loop
b.FinishLoop( loopId_Top );
b.FinishFace( faceId_Top );
// Front face.
b.AddCoEdge( loopId_Front, edgeId_BottomFront, false ); // other direction in bottom loop
b.AddCoEdge( loopId_Front, edgeId_FrontRight, false ); // other direction in right loop
b.AddCoEdge( loopId_Front, edgeId_TopFront, true ); // other direction in top loop.
b.AddCoEdge( loopId_Front, edgeId_LeftFront, true ); // other direction in left loop.
b.FinishLoop( loopId_Front );
b.FinishFace( faceId_Front );
// Back face
b.AddCoEdge( loopId_Back, edgeId_BottomBack, false ); // other direction in bottom loop
b.AddCoEdge( loopId_Back, edgeId_BackLeft, false ); // other direction in left loop.
b.AddCoEdge( loopId_Back, edgeId_TopBack, true ); // other direction in top loop
b.AddCoEdge( loopId_Back, edgeId_RightBack, true ); // other direction in right loop.
b.FinishLoop( loopId_Back );
b.FinishFace( faceId_Back );
// Right face
b.AddCoEdge( loopId_Right, edgeId_BottomRight, false ); // other direction in bottom loop
b.AddCoEdge( loopId_Right, edgeId_RightBack, false ); // other direction in back loop
b.AddCoEdge( loopId_Right, edgeId_TopRight, true ); // other direction in top loop
b.AddCoEdge( loopId_Right, edgeId_FrontRight, true ); // other direction in front loop
b.FinishLoop( loopId_Right );
b.FinishFace( faceId_Right );
// Left face
b.AddCoEdge( loopId_Left, edgeId_BottomLeft, false ); // other direction in bottom loop
b.AddCoEdge( loopId_Left, edgeId_LeftFront, false ); // other direction in front loop
b.AddCoEdge( loopId_Left, edgeId_TopLeft, true ); // other direction in top loop
b.AddCoEdge( loopId_Left, edgeId_BackLeft, true ); // other direction in back loop
b.FinishLoop( loopId_Left );
b.FinishFace( faceId_Left );
b.Finish();
return b;
}
///
/// Create a cylinder to subtract from the cube.
/// summary>
public BRepBuilder CreateBrepVoid()
{
// Naming convention for faces and edges: we
// assume that x is to the left and pointing down,
// y is horizontal and pointing to the right,
// z is up.
BRepBuilder b = new BRepBuilder( BRepType.Solid );
// The surfaces of the four faces.
Frame basis = new Frame( new XYZ( 50, 0, 0 ), new XYZ( 0, 1, 0 ), new XYZ( -1, 0, 0 ), new XYZ( 0, 0, 1 ) );
CylindricalSurface cylSurf = CylindricalSurface.Create( basis, 40 );
Plane top1 = Plane.CreateByNormalAndOrigin( new XYZ( 0, 0, 1 ), new XYZ( 0, 0, 100 ) ); // normal points outside the cylinder
Plane bottom1 = Plane.CreateByNormalAndOrigin( new XYZ( 0, 0, 1 ), new XYZ( 0, 0, 0 ) ); // normal points inside the cylinder
// Add the four faces
BRepBuilderGeometryId frontCylFaceId = b.AddFace( BRepBuilderSurfaceGeometry.Create( cylSurf, null ), false );
BRepBuilderGeometryId backCylFaceId = b.AddFace( BRepBuilderSurfaceGeometry.Create( cylSurf, null ), false );
BRepBuilderGeometryId topFaceId = b.AddFace( BRepBuilderSurfaceGeometry.Create( top1, null ), false );
BRepBuilderGeometryId bottomFaceId = b.AddFace( BRepBuilderSurfaceGeometry.Create( bottom1, null ), true );
// Geometry for the four semi-circular edges and two vertical linear edges
BRepBuilderEdgeGeometry frontEdgeBottom = BRepBuilderEdgeGeometry.Create( Arc.Create( new XYZ( 10, 0, 0 ), new XYZ( 90, 0, 0 ), new XYZ( 50, 40, 0 ) ) );
BRepBuilderEdgeGeometry backEdgeBottom = BRepBuilderEdgeGeometry.Create( Arc.Create( new XYZ( 90, 0, 0 ), new XYZ( 10, 0, 0 ), new XYZ( 50, -40, 0 ) ) );
BRepBuilderEdgeGeometry frontEdgeTop = BRepBuilderEdgeGeometry.Create( Arc.Create( new XYZ( 10, 0, 100 ), new XYZ( 90, 0, 100 ), new XYZ( 50, 40, 100 ) ) );
BRepBuilderEdgeGeometry backEdgeTop = BRepBuilderEdgeGeometry.Create( Arc.Create( new XYZ( 10, 0, 100 ), new XYZ( 90, 0, 100 ), new XYZ( 50, -40, 100 ) ) );
BRepBuilderEdgeGeometry linearEdgeFront = BRepBuilderEdgeGeometry.Create( new XYZ( 90, 0, 0 ), new XYZ( 90, 0, 100 ) );
BRepBuilderEdgeGeometry linearEdgeBack = BRepBuilderEdgeGeometry.Create( new XYZ( 10, 0, 0 ), new XYZ( 10, 0, 100 ) );
// Add the six edges
BRepBuilderGeometryId frontEdgeBottomId = b.AddEdge( frontEdgeBottom );
BRepBuilderGeometryId frontEdgeTopId = b.AddEdge( frontEdgeTop );
BRepBuilderGeometryId linearEdgeFrontId = b.AddEdge( linearEdgeFront );
BRepBuilderGeometryId linearEdgeBackId = b.AddEdge( linearEdgeBack );
BRepBuilderGeometryId backEdgeBottomId = b.AddEdge( backEdgeBottom );
BRepBuilderGeometryId backEdgeTopId = b.AddEdge( backEdgeTop );
// Loops of the four faces
BRepBuilderGeometryId loopId_Top = b.AddLoop( topFaceId );
BRepBuilderGeometryId loopId_Bottom = b.AddLoop( bottomFaceId );
BRepBuilderGeometryId loopId_Front = b.AddLoop( frontCylFaceId );
BRepBuilderGeometryId loopId_Back = b.AddLoop( backCylFaceId );
// Add coedges for the loop of the front face
b.AddCoEdge( loopId_Front, linearEdgeBackId, false );
b.AddCoEdge( loopId_Front, frontEdgeTopId, false );
b.AddCoEdge( loopId_Front, linearEdgeFrontId, true );
b.AddCoEdge( loopId_Front, frontEdgeBottomId, true );
b.FinishLoop( loopId_Front );
b.FinishFace( frontCylFaceId );
// Add coedges for the loop of the back face
b.AddCoEdge( loopId_Back, linearEdgeBackId, true );
b.AddCoEdge( loopId_Back, backEdgeBottomId, true );
b.AddCoEdge( loopId_Back, linearEdgeFrontId, false );
b.AddCoEdge( loopId_Back, backEdgeTopId, true );
b.FinishLoop( loopId_Back );
b.FinishFace( backCylFaceId );
// Add coedges for the loop of the top face
b.AddCoEdge( loopId_Top, backEdgeTopId, false );
b.AddCoEdge( loopId_Top, frontEdgeTopId, true );
b.FinishLoop( loopId_Top );
b.FinishFace( topFaceId );
// Add coedges for the loop of the bottom face
b.AddCoEdge( loopId_Bottom, frontEdgeBottomId, false );
b.AddCoEdge( loopId_Bottom, backEdgeBottomId, false );
b.FinishLoop( loopId_Bottom );
b.FinishFace( bottomFaceId );
b.Finish();
return b;
}
public Result Execute(
ExternalCommandData commandData,
ref string message,
ElementSet elements )
{
UIApplication app = commandData.Application;
UIDocument uidoc = app.ActiveUIDocument;
Document doc = uidoc.Document;
// Execute the BrepBuilder methods.
BRepBuilder brepBuilderSolid = CreateBrepSolid();
BRepBuilder brepBuilderVoid = CreateBrepVoid();
Solid cube = brepBuilderSolid.GetResult();
Solid cylinder = brepBuilderVoid.GetResult();
// Determine their Boolean difference.
Solid difference
= BooleanOperationsUtils.ExecuteBooleanOperation(
cube, cylinder, BooleanOperationsType.Difference );
IList list = new List();
list.Add( difference );
using( Transaction tr = new Transaction( doc ) )
{
tr.Start( "Create a DirectShape" );
// Create a direct shape.
DirectShape ds = DirectShape.CreateElement( doc,
new ElementId( BuiltInCategory.OST_GenericModel ) );
ds.SetShape( list );
tr.Commit();
}
return Result.Succeeded;
}
}
```
Note that it’s much easier to use `GeometryCreationUtilities` functions to create extrusions than it is to use `BRepBuilder`, unless the whole point of this exercise is to use the latter.
As said, using `BRepBuilder` to 'manually' construct geometry is very inconvenient and error-prone, and that isn’t its purpose.
Many thanks to my colleagues Ryuji Ogasawara, Angel Velez, Boris Shafiro and John Mitchell for sharing, correcting, and commenting on this nice example!
I added it
to [The Building Coder Samples](https://github.com/jeremytammik/the_building_coder_samples)
[release 2018.0.136.0](https://github.com/jeremytammik/the_building_coder_samples/releases/tag/2018.0.136.0) in
the new [module CmdBrepBuilder.cs](https://github.com/jeremytammik/the_building_coder_samples/blob/master/BuildingCoder/BuildingCoder/CmdBrepBuilder.cs).