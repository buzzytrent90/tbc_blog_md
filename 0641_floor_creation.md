---
ai_optimized: true
author: Jeremy Tammik
complexity_score: 3.1
content_type: code_example
optimization_date: '2025-12-11T11:44:14.314748'
original_url: https://thebuildingcoder.typepad.com/blog/0641_floor_creation.html
post_number: '0641'
reading_time_minutes: 5
series: general
slug: floor_creation
source_file: 0641_floor_creation.htm
tags:
- csharp
- elements
- family
- filtering
- geometry
- levels
- parameters
- python
- revit-api
- selection
- transactions
- walls
title: Floor Creation
word_count: 1083
---

### Floor Creation

I already presented examples on using the NewFloor method to
[edit a floor profile](http://thebuildingcoder.typepad.com/blog/2008/11/editing-a-floor-profile.html) and
[perform Boolean operations on two-dimensional polygons](http://thebuildingcoder.typepad.com/blog/2009/02/boolean-operations-for-2d-polygons.html).

Now Guming submitted a
[simpler question](http://thebuildingcoder.typepad.com/blog/2011/08/modeless-loose-connector-navigator-update.html?cid=6a00e553e168978833015391044035970b#comment-6a00e553e168978833015391044035970b), on
how to use the NewFloor method to create a floor from scratch,
[twice](http://thebuildingcoder.typepad.com/blog/2011/08/modeless-loose-connector-navigator-update.html?cid=6a00e553e168978833015434f762a5970c#comment-6a00e553e168978833015434f762a5970c) in
fact, so let's have a look at that:

**Question:** How can I use certain points to create a new floor?
The function NewFloor doesn't work for me.

**Answer:** You create a new floor using the NewFloor method.
You need to provide the profile curves and specify whether it is structural or not.
Other optional arguments can determine the floor type, level and a normal vector in case it is not horizontal.
According to the help file, the profile curves must always be specified horizontally, regardless of the desired floor slope.
Here are the argument lists of the three NewFloor overloads:

- CurveArray profile, Boolean structural: Creates a floor within the project with the given horizontal profile using the default floor style.- CurveArray profile, FloorType, Level, Boolean structural: Creates a floor within the project with the given horizontal profile and floor style on the specified level.- CurveArray profile, FloorType, Level, Boolean structural, XYZ normal: Creates a floor within the project with the given horizontal profile and floor style on the specified level with the specified normal vector.

Further documentation on the use of this method is provided in the developer guide section 11.2 'Floors and Foundations'.

The Building Coder samples include the command
[CmdEditFloor](http://thebuildingcoder.typepad.com/blog/2008/11/editing-a-floor-profile.html) which
exercises the second one of the NewFloor overloads, and so does the Revit SDK sample application GenerateFloor.

CmdEditFloor reads a profile from an existing floor and modifies that, whereas GenerateFloor demonstrates how to generate a floor using the closed outline made by a selection of walls.

Guming, one problem with the input points that you list is that they are not horizontal.
Some have a Z coordinate of 0, others 10.
That might be causing the method to fail for you.

Here is another related issue and interesting sample from a case handled by my colleagues Saikat Bhattacharya and Harry Mattison:

**Question:** I wish to create a whole set of floors by automatically generating slabs in every level of a project based on profiles.

When creating a set of floors using profiles, all the floors seem to be created with a non-zero value for the 'Height Offset from Level' parameter.
Why is this parameter value being set to a non-zero value?

In spite of this parameter value being set and the Level property showing the correct information, all the floors are being created on the same level.
I can manually set this parameter value to zero to get the expected results, but I am not sure why it is being automatically set in the first place.

**Answer:** The floors are created at the position of the profile curves.
The floors are associated with the specified level, but the level does not set the floor's height.

Therefore, all floors are created with the same set of profiles and each floor is associated with a different level.
All the profile curves have a Z coordinate equal to zero.
So when a floor is associated with a level at 28 feet and its curves are at 0, the offset will be -28.

You have two options:

- Leave the Z value of all curve points at zero, create the floors, and set their FLOOR\_HEIGHTABOVELEVEL\_PARAM parameter value to zero.- Modify each and every curve Z coordinate before creating the floors to ensure that the parameter is set to zero to begin with.

Which method to choose depends on how you want your model built.
In some cases, you will want to associate with the nearest level and not use the offsets.
In other cases, you might not care about what level is related to the floor and position them using offsets only.

Presumably, if you set all the horizontal curve profile Z coordinates to the elevation of the level, the floor height above level parameter value will be automatically set to zero.

I implemented a sample application CreateFloors to demonstrate this.
It performs the following steps:

- Create a list of all the levels in the project.- Determine the floor type to use, Generic - 12".- Retrieve the first generic model named WP1 to use as a profile definition element.- Traverse the profile definition element geometry and collect all its curves.- Use these curves to generate a floor on each of the levels.- Set the height above level parameter on each floor to zero, to that it resides on the same height as its associated level.

Here is the source code to achieve these steps:
```python
[Transaction( TransactionMode.Manual )]
public class Command : IExternalCommand
{
  public Result Execute(
    ExternalCommandData commandData,
    ref string message,
    ElementSet elements )
  {
    UIApplication uiapp = commandData.Application;
    UIDocument uidoc = uiapp.ActiveUIDocument;
    Application app = uiapp.Application;
    Document doc = uidoc.Document;

    FilteredElementCollector levels
      = new FilteredElementCollector( doc )
        .OfClass( typeof( Level ) );

    FloorType floorType
      = new FilteredElementCollector( doc )
        .OfClass( typeof( FloorType ) )
        .First<Element>(
          e => e.Name.Equals( "Generic - 12\"" ) )
          as FloorType;

    Element profileElement
      = new FilteredElementCollector( doc )
        .OfClass( typeof( FamilyInstance ) )
        .OfCategory( BuiltInCategory.OST\_GenericModel )
        .First<Element>(
          e => e.Name.Equals( "WP1" ) );

    CurveArray slabCurves = new CurveArray();

    GeometryElement geo = profileElement.get\_Geometry( new Options() );

    foreach( GeometryInstance inst in geo.Objects )
    {
      foreach( GeometryObject obj in inst.SymbolGeometry.Objects )
      {
        if( obj is Curve )
        {
          slabCurves.Append( obj as Curve );
        }
      }
    }

    XYZ normal = XYZ.BasisZ;

    Transaction trans = new Transaction( doc );
    trans.Start( "Create Floors" );

    foreach( Level level in levels )
    {
      Floor newFloor = doc.Create.NewFloor( slabCurves,
        floorType, level, false, normal );

      newFloor.get\_Parameter(
        BuiltInParameter.FLOOR\_HEIGHTABOVELEVEL\_PARAM ).Set( 0 );
    }

    trans.Commit();

    return Result.Succeeded;
  }
}
```

These are some sample levels to use to create floors on:
![Floor creation levels](img/create_floor_levels.png)

This is the generic model that the floor profile is read from:
![Floor creation generic model](img/create_floor_generic_model.png)

Here are the resulting floors:
![Floor creation result](img/create_floor_floors.png)

Finally, here is
[CreateFloors.zip](zip/CreateFloors.zip) containing
the complete Visual Studio solution, C# source code, and add-in manifest file of the CreateFloors external command.