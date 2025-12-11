---
ai_optimized: true
author: Jeremy Tammik
complexity_score: 2.0
content_type: qa
optimization_date: '2025-12-11T11:44:13.328813'
original_url: https://thebuildingcoder.typepad.com/blog/0079_two_levels.html
post_number: 0079
reading_time_minutes: 3
series: general
slug: two_levels
source_file: 0079_two_levels.htm
tags:
- csharp
- doors
- elements
- family
- filtering
- levels
- parameters
- revit-api
- walls
title: Walls and Doors on Two Levels
word_count: 565
---

### Walls and Doors on Two Levels

Here is a little note to answer a
[question from Berria](http://thebuildingcoder.typepad.com/blog/2008/10/relationship-in.html#comments) on creating walls and doors on two separate levels.
This is a slightly edited version of the original query:

**Question:**
I need to create a house with two floors.
I create a level and then a new wall and a door.
It draws the two objects on different levels.
When I built the first level, all is OK.
An error occurs when I build the second, because the level to define the component position is always zero.

**Answer:**
I cannot tell you off-hand what the problem is in your situation, but I can provide you with a code example that achieves your goal.
Look at the external command Lab2\_0\_CreateLittleHouse in the
[Revit API introduction labs](http://thebuildingcoder.typepad.com/blog/files/rac_labs_20081117.zip).
To demonstrate that the code can be easily adapted to work on several levels, here is a modified version which creates only the walls and doors, omitting the other elements, and repeats the process on two levels.
In order to place the second door, I do indeed modify the Z coordinate of its insertion point by setting 'midpoint.Z = levelMiddle.Elevation'.
Before running this command, ensure that you have three levels in your model, and that the brain-dead code snippet which extracts them into the three variables really gets them in the right order:

```csharp
WaitCursor waitCursor = new WaitCursor();
Application app = commandData.Application;
Document doc = app.ActiveDocument;
\_createApp = app.Create;
\_createDoc = doc.Create;
//
// determine the four corners of the rectangular house:
//
double width = 7 \* LabConstants.MeterToFeet;
double depth = 4 \* LabConstants.MeterToFeet;
List<XYZ> corners = new List<XYZ>( 4 );
corners.Add( new XYZ( 0, 0, 0 ) );
corners.Add( new XYZ( width, 0, 0 ) );
corners.Add( new XYZ( width, depth, 0 ) );
corners.Add( new XYZ( 0, depth, 0 ) );

Level levelBottom = null;
Level levelMiddle = null;
Level levelTop = null;
List<Element> levels = new List<Element>();

Filter filterType
  = \_createApp.Filter.NewTypeFilter(
    typeof( Level ) );

doc.get\_Elements( filterType, levels );
foreach( Element e in levels )
{
  if( null == levelBottom )
  {
    levelBottom = e as Level;
  }
  else if( null == levelMiddle )
  {
    levelMiddle = e as Level;
  }
  else if( null == levelTop )
  {
    levelTop = e as Level;
  }
  else
  {
    break;
  }
}

BuiltInParameter topLevelParam
  = BuiltInParameter.WALL\_HEIGHT\_TYPE;

Line line;
Wall wall;
Parameter param;

ElementId topId = levelMiddle.Id;
List<Wall> walls = new List<Wall>( 8 );
for( int i = 0; i < 4; ++i )
{
  line = \_createApp.NewLineBound(
    corners[i], corners[3 == i ? 0 : i + 1] );

  wall = \_createDoc.NewWall(
    line, levelBottom, false );

  param = wall.get\_Parameter( topLevelParam );
  param.Set( ref topId );
  walls.Add( wall );
}

topId = levelTop.Id;
for( int i = 0; i < 4; ++i )
{
  line = \_createApp.NewLineBound(
    corners[i], corners[3 == i ? 0 : i + 1] );

  wall = \_createDoc.NewWall(
    line, levelMiddle, false );

  param = wall.get\_Parameter( topLevelParam );
  param.Set( ref topId );
  walls.Add( wall );
}

List<Element> doorSymbols
  = LabUtils.GetAllFamilySymbols(
    app, BuiltInCategory.OST\_Doors );

Debug.Assert(
  0 < doorSymbols.Count,
  "expected at least one door symbol"
  + " to be loaded into project" );

FamilySymbol door
  = doorSymbols[0] as FamilySymbol;

XYZ midpoint = LabUtils.Midpoint(
  corners[0], corners[1] );

FamilyInstance inst0
  = \_createDoc.NewFamilyInstance(
    midpoint, door, walls[0], levelBottom,
    StructuralType.NonStructural );

midpoint.Z = levelMiddle.Elevation;

FamilyInstance inst1
  = \_createDoc.NewFamilyInstance(
    midpoint, door, walls[4], levelMiddle,
    StructuralType.NonStructural );
return CmdResult.Succeeded;
```

Here is the result of running the command:

![Two level house](img/two_level_house.png)

I hope this helps resolve your problem.