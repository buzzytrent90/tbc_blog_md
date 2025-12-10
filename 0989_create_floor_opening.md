---
post_number: "0989"
title: "Create a Floor with an Opening or Complex Boundary"
slug: "create_floor_opening"
author: "Jeremy Tammik"
tags: ['csharp', 'elements', 'filtering', 'geometry', 'references', 'revit-api', 'rooms', 'selection', 'transactions', 'views']
source_file: "0989_create_floor_opening.htm"
original_url: "https://thebuildingcoder.typepad.com/blog/0989_create_floor_opening.html"
---

### Create a Floor with an Opening or Complex Boundary

Here is another question raised and solved by Victor Chekalin, Виктор Чекалин:

**Question:** I need to programmatically create a floor.
The problem is that the floor is complex with different boundary loops, like this:

![Complex floor boundaries](img/vc_complex_floor_boundaries.png)

The API provides only one single method to create a new floor – Document.NewFloor.
I have to pass a CurveArray to this method, and the floor will be created on these curves.

But using this method I cannot create a complex floor as shown, with openings.
If I create an array containing eight lines for the outer and inner boundary loops, it produces the following:

![Erroneous resulting floor](img/vc_complex_floor_creating_result.png)

Actually, I also cannot find the way retrieve the existing floor boundaries.
The ones displayed above were obtained from the floor geometry solid, which does not produce exactly the same results as its boundary.

Is there way to create a floor programmatically with several boundary loops?

**Answer:** Here are the examples of using the NewFloor method that I am aware of:

- SDK samples: FoundationSlab, GenerateFloor- RevitLookup: Utils/Elements.cs and TestElements.cs- Blog posts:
      - [Editing a Floor Profile](http://thebuildingcoder.typepad.com/blog/2008/11/editing-a-floor-profile.html)
      - [Boolean Operations for 2D Polygons](http://thebuildingcoder.typepad.com/blog/2009/02/boolean-operations-for-2d-polygons.html)
      - [Hole in a Floor](http://thebuildingcoder.typepad.com/blog/2009/05/hole-in-a-floor.html)
      - [Floor Creation](http://thebuildingcoder.typepad.com/blog/2011/08/floor-creation.html)
      - [Pick Corners and Create Floor](http://thebuildingcoder.typepad.com/blog/2011/11/pick-corners-and-create-floor.html)
      - [Validate Roof Type and View OBJ on Android](http://thebuildingcoder.typepad.com/blog/2012/08/validate-roof-type-and-view-obj-on-android.html)

Have you looked at all of these?

The blog post discussing a hole in a floor states that you have to use an opening to generate the hole.

Is there any other way to create a floor with a hole manually, or does it also require a shaft or an opening of some kind?

If that is the case, then the API will impose the same requirement.

Mostly the API will not allow you to model things that you cannot also create manually.

**Response:** In fact I already could copy floor without openings like in your sample to
[edit a floor profile](http://thebuildingcoder.typepad.com/blog/2008/11/editing-a-floor-profile.html).

After reading the posts you listed, I created the function to copy existing floor with openings.
I assumed that the first EdgeArray in the Face.EdgeLoops is the outer boundary of the floor and the following ones are opening boundaries.
As it turns out, this is not always the case.

Here is the
[code implementing this](http://pastebin.com/sWXvLyKc):

```csharp
[Transaction( TransactionMode.Manual )]
public class ElementRoomInfoCommand : IExternalCommand
{
  const double \_eps = 1.0e-9;

  public Result Execute(
    ExternalCommandData commandData,
    ref string message,
    ElementSet elements )
  {
    UIApplication uiapp = commandData.Application;
    UIDocument uidoc = uiapp.ActiveUIDocument;
    Application app = uiapp.Application;
    Document doc = uidoc.Document;

    Reference r;
    try
    {
      r = uidoc.Selection.PickObject( ObjectType.Element,
        new FloorSelectionFilter(), "Select a floor" );
    }
    catch( Exception )
    {
      return Result.Cancelled;
    }

    var floor = doc.GetElement( r.ElementId ) as Floor;

    using( Transaction t = new Transaction( doc ) )
    {
      t.Start( "Copy Floor" );

      var newFloor = CopyFloor( floor );

      var moveRes =
        newFloor.Location.Move( new XYZ( 0, 0, 10 ) );

      t.Commit();

      t.Start( "Create new floor openings" );

      CreateFloorOpenings( floor, newFloor );

      var res = t.Commit();
    }
    return Result.Succeeded;
  }

  private void CreateFloorOpenings(
    Floor sourceFloor,
    Floor destFloor )
  {
    // Looking if source floor has openings

    var floorGeometryElement =
        sourceFloor.get\_Geometry( new Options() );

    foreach( var geometryObject in floorGeometryElement )
    {
      var floorSolid =
        geometryObject as Solid;

      if( floorSolid == null )
        continue;

      var topFace =
        GetTopFace( floorSolid );

      if( topFace == null )
        throw new NotSupportedException(
          "Floor does not have top face" );

      if( topFace.EdgeLoops.IsEmpty )
        throw new NotSupportedException(
          "Floor top face does not have edges" );

      // If source floor has openings

      if( topFace.EdgeLoops.Size > 1 )
      {
        for( int i = 1; i < topFace.EdgeLoops.Size; i++ )
        {
          var openingEdges =
            topFace.EdgeLoops.get\_Item( i );

          var openingCurveArray =
            GetCurveArrayFromEdgeArary( openingEdges );

          var opening =
            sourceFloor
              .Document
              .Create
              .NewOpening( destFloor,
                    openingCurveArray,
                    true );
        }
      }
    }
  }

  private Floor CopyFloor( Floor sourceFloor )
  {
    var floorGeometryElement =
      sourceFloor.get\_Geometry( new Options() );

    foreach( var geometryObject in floorGeometryElement )
    {
      var floorSolid =
        geometryObject as Solid;

      if( floorSolid == null )
        continue;

      var topFace =
        GetTopFace( floorSolid );

      if( topFace == null )
        throw new NotSupportedException(
          "Floor does not have top face" );

      if( topFace.EdgeLoops.IsEmpty )
        throw new NotSupportedException(
          "Floor top face does not have edges" );

      var outerBoundary =
        topFace.EdgeLoops.get\_Item( 0 );

      // Create new floor using source
      // floor outer boundaries

      CurveArray floorCurveArray =
        GetCurveArrayFromEdgeArary( outerBoundary );

      var newFloor =
        sourceFloor
          .Document
          .Create
          .NewFloor( floorCurveArray, false );

      return newFloor;
    }
    return null;
  }

  private CurveArray GetCurveArrayFromEdgeArary(
    EdgeArray edgeArray )
  {
    CurveArray curveArray =
      new CurveArray();

    foreach( Edge edge in edgeArray )
    {
      var edgeCurve =
          edge.AsCurve();

      curveArray.Append( edgeCurve );
    }
    return curveArray;
  }

  PlanarFace GetTopFace( Solid solid )
  {
    PlanarFace topFace = null;
    FaceArray faces = solid.Faces;
    foreach( Face f in faces )
    {
      PlanarFace pf = f as PlanarFace;
      if( null != pf
        && ( Math.Abs( pf.Normal.X - 0 ) < \_eps
        && Math.Abs( pf.Normal.Y - 0 ) < \_eps ) )
      {
        if( ( null == topFace )
          || ( topFace.Origin.Z < pf.Origin.Z ) )
        {
          topFace = pf;
        }
      }
    }
    return topFace;
  }
}

public class FloorSelectionFilter : ISelectionFilter
{
  public bool AllowElement( Element elem )
  {
    return elem is Floor;
  }

  public bool AllowReference( Reference r, XYZ p )
  {
    throw new NotImplementedException();
  }
}
```

Unfortunately, this produces an error when I try to copy a floor with openings:

![Error message](img/vc_complex_floor1.png)

The reason is that Revit cannot create an opening in the floor until after the floor creation transaction has been committed.
That means that you cannot copy a floor with openings in one single transaction.

My next attempt consists in copying the floor without its openings, committing the transaction, and then creating the openings in a second step.

This method works.
Here is the main implementation source code:

```csharp
  private Floor CopyFloor( Floor sourceFloor )
  {
    var floorGeometry =
      sourceFloor.get\_Geometry( new Options() );

    foreach( var geometryObject in floorGeometry )
    {
      var floorSolid =
        geometryObject as Solid;

      if( floorSolid == null )
        continue;

      var topFace =
        GetTopFace( floorSolid );

      if( topFace == null )
        throw new NotSupportedException(
          "Floor does not have top face" );

      if( topFace.EdgeLoops.IsEmpty )
        throw new NotSupportedException(
          "Floor top face does not have edges" );

      var outerBoundary =
        topFace.EdgeLoops.get\_Item( 0 );

      // Create new floor using source
      // floor outer boundaries

      CurveArray floorCurveArray =
        GetCurveArrayFromEdgeArary( outerBoundary );

      var newFloor =
        sourceFloor
          .Document
          .Create
          .NewFloor( floorCurveArray, false );

      // If source floor has openings

      if( topFace.EdgeLoops.Size > 1 )
      {
        for( int i = 1; i < topFace.EdgeLoops.Size; i++ )
        {
          var openingEdges =
            topFace.EdgeLoops.get\_Item( i );

          var openingCurveArray =
            GetCurveArrayFromEdgeArary( openingEdges );

          var opening =
            sourceFloor
              .Document
              .Create
              .NewOpening( newFloor,
                openingCurveArray,
                true );
        }
      }
      return newFloor;
    }
    return null;
  }

  private CurveArray GetCurveArrayFromEdgeArary(
    EdgeArray edgeArray )
  {
    CurveArray curveArray =
      new CurveArray();

    foreach( Edge edge in edgeArray )
    {
      var edgeCurve =
          edge.AsCurve();

      curveArray.Append( edgeCurve );
    }
    return curveArray;
  }

  PlanarFace GetTopFace( Solid solid )
  {
    PlanarFace topFace = null;
    FaceArray faces = solid.Faces;
    foreach( Face f in faces )
    {
      PlanarFace pf = f as PlanarFace;
      if( null != pf
        && ( Math.Abs( pf.Normal.X - 0 ) < \_eps
        && Math.Abs( pf.Normal.Y - 0 ) < \_eps ) )
      {
        if( ( null == topFace )
          || ( topFace.Origin.Z < pf.Origin.Z ) )
        {
          topFace = pf;
        }
      }
    }
    return topFace;
  }
```

Here is the result, with the original floor at the bottom and the copy at the top:

![Floor with openings](img/vc_complex_floor2.png)

But, as you pointed out, the copied floor is not an exact replica of the original floor:

![Floor with openings](img/vc_complex_floor3.png)

They have only the same visualization:

![Floor with openings](img/vc_complex_floor4.png)

The original floor does not have an opening. It has complex multiply boundaries.

Also, the problem is that my approach is not right.
The first item in the Face.EdgeLoops is not always the outer boundary.
A floor may contain several EdgeArrays without boundaries, for example like this single floor consisting of multiple disjoint pieces:

![Floor with several pieces](img/vc_complex_floor5.png)

This is one single floor, whose geometry contains one single solid.
The top face of this solid has four separate EdgeArrays and the floor doesn't have any openings.
So, I have to implement a method to determine whether a given EdgeArray represents an opening or not.
I think I can do it by just checking if each edge of the EdgeArray is inside any previous EdgeArray, which would mean that it represents an opening.

The other problem: even if I determine all openings and create a 'copy' of the floor, I cannot create a copy of the copy using the same method.

Conclusion:

- There is no way to create an exact copy of a floor with holes using API as in the UI.
- There is a workaround to create holes in the floor using openings instead.

The entire source code and Visual Studio project implementing this command to copy a floor with openings is available from Victor's
[RevitFloorCopy GitHub repository](https://github.com/vchekalin/RevitFloorCopy),
which also includes this direct link to the
[complete zip archive](https://github.com/vchekalin/RevitFloorCopy/archive/master.zip).

Thank you very much, Victor, for your persistent and insightful research and sharing this valuable solution!