---
ai_optimized: true
author: Jeremy Tammik
complexity_score: 2.2
content_type: code_example
optimization_date: '2025-12-11T11:44:14.271984'
original_url: https://thebuildingcoder.typepad.com/blog/0616_create_gable_wall.html
post_number: '0616'
reading_time_minutes: 2
series: general
slug: create_gable_wall
source_file: 0616_create_gable_wall.htm
tags:
- elements
- filtering
- levels
- python
- revit-api
- transactions
- walls
title: Create Gable Wall
word_count: 407
---

### Create Gable Wall

I discussed
[creating a wall with a sloped profile](http://thebuildingcoder.typepad.com/blog/2009/02/creating-a-wall-with-a-sloped-profile.html) using
Revit 2009.
Now Saikat Bhattacharya created a similar command to answer a similar question in Revit 2012:

**Question:** Using the Revit user interface, I can create walls that consist of more then four edges, i.e. non-rectangular.
Usually they would represent some sort of gable wall in a building.
How can I achieve this using the API, please?

**Answer:** To answer your question, I wrote some quick code which creates a gable wall with seven faces, instead of the usual six faces of a rectangular wall.
It creates the following wall:

![Gable wall](img/gable_wall.png)

**Jeremy adds:** Many thanks to Saikat for setting this up!

I created a new Building Coder sample command CmdCreateGableWall based on Saikat's code.
It is similar to the existing external command
[CmdSlopedWall](http://thebuildingcoder.typepad.com/blog/2009/02/creating-a-wall-with-a-sloped-profile.html),
updated to use new Revit API functionality to use manual transaction mode and filtered element collectors and LINQ to determine a suitable wall type and level:
```python
[Transaction( TransactionMode.Manual )]
class CmdCreateGableWall : IExternalCommand
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

    // Build a wall profile for the wall creation

    XYZ [] pts = new XYZ[] {
      XYZ.Zero,
      new XYZ( 20, 0, 0 ),
      new XYZ( 20, 0, 15 ),
      new XYZ( 10, 0, 30 ),
      new XYZ( 0, 0, 15 )
    };

    // Get application creation object

    Autodesk.Revit.Creation.Application appCreation
      = app.Create;

    // Create wall profile

    CurveArray profile = new CurveArray();

    XYZ q = pts[ pts.Length - 1 ];

    foreach( XYZ p in pts )
    {
      profile.Append( appCreation.NewLineBound(
        q, p ) );

      q = p;
    }

    XYZ normal = XYZ.BasisY;

    //WallType wallType
    //  = new FilteredElementCollector( doc )
    //    .OfClass( typeof( WallType ) )
    //    .First<Element>( e
    //      => e.Name.Contains( "Generic" ) )
    //    as WallType;

    WallType wallType
      = new FilteredElementCollector( doc )
        .OfClass( typeof( WallType ) )
        .First<Element>()
          as WallType;

    Level level
      = new FilteredElementCollector( doc )
        .OfClass( typeof( Level ) )
        .First<Element>( e
          => e.Name.Equals( "Level 1" ) )
        as Level;

    Transaction trans = new Transaction( doc );

    trans.Start( "Create Gable Wall" );

    Wall wall = doc.Create.NewWall(
      profile, wallType, level, true, normal );

    trans.Commit();

    return Result.Succeeded;
  }
}
```

Here is
[version 2012.0.88.0](zip/bc_12_88.zip) of
The Building Coder samples including the new command CmdCreateGableWall as well as the existing simpler CmdSlopedWall one.