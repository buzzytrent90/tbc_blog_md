---
post_number: "0395"
title: "Set Tag Type"
slug: "set_tag_type"
author: "Jeremy Tammik"
tags: ['csharp', 'doors', 'elements', 'family', 'filtering', 'levels', 'parameters', 'revit-api', 'views', 'walls']
source_file: "0395_set_tag_type.htm"
original_url: "https://thebuildingcoder.typepad.com/blog/0395_set_tag_type.html"
---

### Set Tag Type

The exploration of a question on setting the tag type for a newly created tag prompted me to put together a new little Building Coder sample command which may be useful for other purposes as well, since it can be started in an empty architectural project and performs the following steps:

- Determine bottom and top level for placing a wall.- Create a new wall element and set its top bounding level.- Retrieve the wall thickness to later calculate an offset for the tag.- Retrieve a door symbol to use to insert a door in the wall.- Create a new door element at the wall midpoint.- Create a new door tag element associated with the door.- Retrieve an existing door tag type to duplicate.- Create a new door tag type by calling the Duplicate method.- Change the door tag's type to the new door tag type.

The code to implement the first six steps has been extracted from the Revit API introduction Lab2\_0\_CreateLittleHouse external command, which we used repeatedly in the past to demonstrate various aspects, such as
[selecting all walls](http://thebuildingcoder.typepad.com/blog/2008/09/selecting-all-w.html),
[determining wall dimensions](http://thebuildingcoder.typepad.com/blog/2008/09/wall-dimensions.html),
creating
[walls and doors on two levels](http://thebuildingcoder.typepad.com/blog/2009/01/walls-and-doors-on-two-levels.html),
and in Revit 2011 showing the
[immutability of the XYZ class](http://thebuildingcoder.typepad.com/blog/2010/04/xyz-immutable.html),
[risks of the manual regeneration option](http://thebuildingcoder.typepad.com/blog/2010/04/manual-regeneration-mode-danger.html),
and a temporary problem requiring an explicit call to the
[AutoJoinElements](http://thebuildingcoder.typepad.com/blog/2010/05/autojoinelements.html) method.

We already looked at the use of the Duplicate method to create a new family symbol for
[walls and columns](http://thebuildingcoder.typepad.com/blog/2008/11/creating-a-new-family-symbol.html),
later again for
[columns](http://thebuildingcoder.typepad.com/blog/2009/02/inserting-a-column.html) and
[beams](http://thebuildingcoder.typepad.com/blog/2009/02/inserting-a-beam.html),
[family instances](http://thebuildingcoder.typepad.com/blog/2009/04/revit-api-cases-1.html#3),
[dimensioning](http://thebuildingcoder.typepad.com/blog/2009/12/distinguish-different-dimension-types.html),
and to create a new
[material](http://thebuildingcoder.typepad.com/blog/2009/05/new-material.html).

The code also demonstrates some neat little element filtering collector usages.

So it is well worth extracting it and placing it into a self-contained Building Coder sample for easy immediate access.

Here is the question that prompted this update:

**Question:** I am writing some code to automatically tag an element.
Here is the line I am using to create the new tag:
```csharp
  doc.Create.NewTag(
    doc.ActiveView,
    elem,
    False,
    TagMode.TM\_ADDBY\_CATEGORY,
    TagOrientation.TAG\_HORIZONTAL,
    panelCenter );
```

This method allows me to choose the tagging mode, but not the type of tag that I want.

How can I specify the tag type to be used for the newly created tag?

In the user interface, I can define it in this dialogue:

![Default tag type per category](img/tag_list.png)

**Answer:** You can set the tag type by calling ChangeTypeId with the desired tag type element id on the tag returned by the NewTag method.

I implemented a new Building Coder sample command CmdSetTypeTag to answer this and provide an executable example of implementing it.
It executes all of the steps listed in the introduction above, the last four of which address your specific question:

- Create a door tag.- Retrieve an existing door tag type.- Duplicate it to create a new door tag type.- Assign the new type to the door tag.

The command makes use of a couple of constants and helper methods.

First of all, we define a conversion unit for converting meters to feet, since all length measurements within the Revit database make use of the latter.
Our recent exploration of the
[voltage units](http://thebuildingcoder.typepad.com/blog/2010/06/voltage-units.html) includes
an extensive list of back pointers to posts on this subject:
```csharp
  const double MeterToFeet = 3.2808399;
```

This geometric helper method returns the midpoint between two points:
```csharp
public static XYZ Midpoint( XYZ p, XYZ q )
{
  return p + 0.5 \* ( q - p );
}
```

Then we have a group of filtered element collector routines to effectively retrieve the following sets of elements from the Revit database:

- GetElementsOfType:
  Return all elements of the requested class,
  i.e. System.Type, matching the given built-in
  category in the given document.- GetFamilySymbols:
    Return all family symbols in the given document
    matching the given built-in category.- GetFirstFamilySymbol:
      Return the first family symbol found in the given document
      matching the given built-in category, or null if none is found.- GetBottomAndTopLevels:
        Determine bottom and top levels for creating walls.
        In a default empty Revit Architecture project,
        'Level 1' and 'Level 2' will be returned.
        Returns true if the two levels are successfully determined.

```csharp
static FilteredElementCollector
  GetElementsOfType(
    Document doc,
    Type type,
    BuiltInCategory bic )
{
  FilteredElementCollector collector
    = new FilteredElementCollector( doc );

  collector.OfCategory( bic );
  collector.OfClass( type );

  return collector;
}

static FilteredElementCollector
  GetFamilySymbols(
    Document doc,
    BuiltInCategory bic )
{
  return GetElementsOfType( doc,
    typeof( FamilySymbol ), bic );
}

static FamilySymbol GetFirstFamilySymbol(
  Document doc,
  BuiltInCategory bic )
{
  FamilySymbol s = GetFamilySymbols( doc, bic )
    .FirstElement() as FamilySymbol;

  Debug.Assert( null != s, string.Format(
    "expected at least one {0} symbol in project",
    bic.ToString() ) );

  return s;
}

static bool GetBottomAndTopLevels(
  Document doc,
  ref Level levelBottom,
  ref Level levelTop )
{
  FilteredElementCollector levels
    = GetElementsOfType( doc, typeof( Level ),
      BuiltInCategory.OST\_Levels );

  foreach( Element e in levels )
  {
    if( null == levelBottom )
    {
      levelBottom = e as Level;
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

  if( levelTop.Elevation < levelBottom.Elevation )
  {
    Level tmp = levelTop;
    levelTop = levelBottom;
    levelBottom = tmp;
  }
  return null != levelBottom && null != levelTop;
}
```

Putting these all together, here is the implementation of the external command mainline Execute method:
```csharp
public Result Execute(
  ExternalCommandData commandData,
  ref string message,
  ElementSet elements )
{
  UIApplication app = commandData.Application;
  Document doc = app.ActiveUIDocument.Document;

  Autodesk.Revit.Creation.Application createApp
    = app.Application.Create;

  Autodesk.Revit.Creation.Document createDoc
    = doc.Create;

  // determine the wall endpoints:

  double length = 5 \* MeterToFeet;

  XYZ [] pts = new XYZ[2];

  pts[0] = XYZ.Zero;
  pts[1] = new XYZ( length, 0, 0 );

  // determine the levels where
  // the wall will be located:

  Level levelBottom = null;
  Level levelTop = null;

  if( !GetBottomAndTopLevels( doc,
    ref levelBottom, ref levelTop ) )
  {
    message = "Unable to determine "
      + "wall bottom and top levels";

    return Result.Failed;
  }

  // create a wall:

  BuiltInParameter topLevelParam
    = BuiltInParameter.WALL\_HEIGHT\_TYPE;

  ElementId topLevelId = levelTop.Id;

  Line line = createApp.NewLineBound(
    pts[0], pts[1] );

  Wall wall = createDoc.NewWall(
    line, levelBottom, false );

  Parameter param = wall.get\_Parameter(
    topLevelParam );

  param.Set( topLevelId );

  // determine wall thickness for tag
  // offset and profile growth:

  double wallThickness = wall.WallType
    .CompoundStructure.Layers.get\_Item( 0 )
    .Thickness;

  // add door to wall;
  // note that the NewFamilyInstance method
  // does not automatically add a door tag,
  // like the ui command does:

  FamilySymbol doorSymbol = GetFirstFamilySymbol(
    doc, BuiltInCategory.OST\_Doors );

  if( null == doorSymbol )
  {
    message = "No door symbol found.";
    return Result.Failed;
  }

  XYZ midpoint = Midpoint( pts[0], pts[1] );

  FamilyInstance door = createDoc
    .NewFamilyInstance( midpoint, doorSymbol,
      wall, levelBottom,
      StructuralType.NonStructural );

  // create door tag:

  View view = doc.ActiveView;

  double tagOffset = 3 \* wallThickness;

  midpoint += tagOffset \* XYZ.BasisY;

  IndependentTag tag = createDoc.NewTag(
    view, door, false, TagMode.TM\_ADDBY\_CATEGORY,
    TagOrientation.TAG\_HORIZONTAL, midpoint );

  // create and assign new door tag type:

  FamilySymbol doorTagType
    = GetFirstFamilySymbol(
      doc, BuiltInCategory.OST\_DoorTags );

  doorTagType = doorTagType.Duplicate(
    "New door tag type" ) as FamilySymbol;

  tag.ChangeTypeId( doorTagType.Id );

  return Result.Succeeded;
}
```

The result of running this in a new empty project is a wall, door, and door tag with the newly created door tag type assigned to it, looking like this:

![New door tag type](img/new_door_tag_type.png)

Here is
[version 2011.0.73.0](zip/bc_11_73.zip)
of The Building Coder samples including the complete source code and Visual Studio solution and the new command.