---
post_number: "0672"
title: "Pick Corners and Create Floor"
slug: "pick_create_floor"
author: "Jeremy Tammik"
tags: ['csharp', 'elements', 'filtering', 'levels', 'references', 'revit-api', 'selection', 'transactions', 'views']
source_file: "0672_pick_create_floor.htm"
original_url: "https://thebuildingcoder.typepad.com/blog/0672_pick_create_floor.html"
---

### Pick Corners and Create Floor

I presented an example demonstrating
[floor creation](http://thebuildingcoder.typepad.com/blog/2011/08/floor-creation.html) and
pointed out that
[setting a view](http://thebuildingcoder.typepad.com/blog/2011/09/activate-a-3d-view.html) using
the ActiveView property cannot be achieved within a transaction.

Ishwar Nagwani, Technical Consultant in our Bangalore organisation, contributed a sample command to create a new a floor which demonstrates both of these aspects in conjunction with the interactive PickBox method to determine the floor corners.

It makes use of the FindElement method, which we have seen in various incarnations to select an element of a given type with a specified name:
```csharp
/// <summary>
/// Helper function to find an element of
/// the given type and the name.
/// For example, use this to find a Reference
/// or Level with the given name.
/// </summary>
Element findElement(
  Document doc,
  Type targetType,
  string targetName )
{
  // Get the elements of the given type

  FilteredElementCollector col
    = new FilteredElementCollector( doc )
      .OfClass( targetType );

  // Return first one with the given name

  Element targetElement = null;

  foreach( Element e in col )
  {
    if( e.Name.Equals( targetName ) )
    {
      targetElement = e;
      break;
    }
  }

  return targetElement;
}
```

By the way, here is another more succinct implementation to achieve the same thing using generics.
I first implemented it using the generic First method, which has the disadvantage of throwing an exception if no element matching the requirements is found.
Luckily, right after the first publication of this, Jon Smith pointed out in a comment below that it can easily be remedied by using FirstOrDefault instead, in which case the implementation is nice and short with no disadvantages at all:
```csharp
/// <summary>
/// Return the first element found of the
/// specific target type with the given name.
/// </summary>
Element FindElement(
  Document doc,
  Type targetType,
  string targetName )
{
  return new FilteredElementCollector( doc )
    .OfClass( targetType )
    .FirstOrDefault<Element>(
      e => e.Name.Equals( targetName ) );
}
```

That method is used in the mainline Execute method of the external command to select a level and a plan view to create the floor on, before switching to that view in order to pick the floor corner points:
```csharp
/// <summary>
/// External command to create a floor element
/// by selecting two points in plan view.
/// </summary>
public Result Execute(
  ExternalCommandData commandData,
  ref string message,
  ElementSet elements )
{
  Result rc = Result.Failed;
  UIApplication uiapp = commandData.Application;
  UIDocument uidoc = uiapp.ActiveUIDocument;
  Application app = uiapp.Application;
  Document doc = uidoc.Document;

  // Get the level for floor

  Level floorLevel = findElement(
    doc, typeof( Level ), "Level 1" ) as Level;

  // Get the plan view

  View planView = findElement(
    doc, typeof( View ), "Level 1" ) as View;

  // Set the active view to plan

  uidoc.ActiveView = planView;

  // Create the floor in transaction

  Transaction tr = new Transaction( doc );

  if( TransactionStatus.Started
    == tr.Start( "Create Floor" ) )
  {
    try
    {
      PickCreateFloor( uiapp, floorLevel );

      tr.Commit();

      rc = Result.Succeeded;
    }
    catch( Exception ex )
    {
      tr.RollBack();

      message = ex.Message;
    }
  }
  return rc;
}
```

Once the level has been selected and the view has been set, the PickCreateFloor method is called to select the floor type to use, prompt the user to pick two corner points, use those to create the floor profile, and generate the floor element based on that:
```csharp
const string \_floorTypeName = "Generic - 12\"";

/// <summary>
/// Create a rectangular floor on the given level,
/// asking the user to interactively specify its
/// corner points.
/// </summary>
Floor PickCreateFloor(
  UIApplication uiapp,
  Level level )
{
  UIDocument uidoc = uiapp.ActiveUIDocument;

  Document doc = uidoc.Document;

  Autodesk.Revit.Creation.Application appCreation
    = uiapp.Application.Create;

  // Get a floor type for floor creation,
  // e.g. Generic - 12"

  FloorType floorType = null;

  foreach( FloorType ft in doc.FloorTypes )
  {
    if( ft.Name.Equals( \_floorTypeName ) )
    {
      floorType = ft;
      break;
    }
  }

  PickedBox box = uidoc.Selection.PickBox(
      PickBoxStyle.Directional,
      "Click and drag to define two corners of a floor" );

  // Get the two user selected points on screen

  XYZ first = box.Min;
  XYZ third = box.Max;

  // Build a floor profile for the floor creation

  XYZ second = new XYZ( third.X, first.Y, 0 );
  XYZ fourth = new XYZ( first.X, third.Y, 0 );

  CurveArray profile = new CurveArray();

  profile.Append( appCreation.NewLineBound( first, second ) );
  profile.Append( appCreation.NewLineBound( second, third ) );
  profile.Append( appCreation.NewLineBound( third, fourth ) );
  profile.Append( appCreation.NewLineBound( fourth, first ) );

  // The normal vector must be perpendicular
  // to the profile

  XYZ normal = XYZ.BasisZ;

  return doc.Create.NewFloor(
    profile, floorType, level, true, normal );
}
```

Here is
[PickCreateFloor.zip](zip/PickCreateFloor.zip) including
the complete source code and Visual Studio solution of this command.

Many thanks to Ishwar for sharing this!