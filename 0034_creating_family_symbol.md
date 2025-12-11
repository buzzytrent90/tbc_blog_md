---
ai_optimized: true
author: Jeremy Tammik
complexity_score: 2.6
content_type: tutorial
optimization_date: '2025-12-11T11:44:13.265185'
original_url: https://thebuildingcoder.typepad.com/blog/0034_creating_family_symbol.html
post_number: '0034'
reading_time_minutes: 2
series: elements
slug: creating_family_symbol
source_file: 0034_creating_family_symbol.htm
tags:
- csharp
- elements
- family
- filtering
- parameters
- revit-api
- selection
- walls
title: Creating a new Family Symbol
word_count: 463
---

### Creating a new Family Symbol

This post is due to Joe Ye in our Beijing office. Many thanks, Joe! While proof reading my handouts for the Revit API tips and tricks session at AU,
[DE205-3 Enhancing Your Revit Add-In](http://au.autodesk.com/sessions/?speaker=Jeremy+Tammik&year=2008),
Joe pointed out that creating a new type or family symbol is a hot topic often asked by developers.
It is not obvious from the Revit API help documentation or samples how to achieve this.
It is simple to solve, though: one can use the Duplicate() method to create a new type, and then modify the parameters or properties required.

Here is an example in VB duplicating a wall type by doubling the thickness of each layer in its compound layer structure:

```vbnet
Public Function Execute( \_
  ByVal commandData As ExternalCommandData, \_
  ByRef message As String, \_
  ByVal elements As ElementSet) \_
As IExternalCommand.Result \_
Implements IExternalCommand.Execute

  Try
    Dim app As Application = commandData.Application
    Dim doc As Document = app.ActiveDocument
    Dim els As ElementSet = doc.Selection.Elements
    Dim e As Element
    Dim newWallTypeName As String \_
      = "NewWallType\_with\_Width\_doubled"

    For Each e In els
      If TypeOf e Is Wall Then
        Dim wall As Wall = e

        Dim wallType As WallType
        wallType = wall.WallType

        Dim newWallType As WallType
        newWallType = wallType.Duplicate(newWallTypeName)
        Dim layers As CompoundStructureLayerArray
        layers = newWallType.CompoundStructure.Layers

        Dim layer As CompoundStructureLayer
        For Each layer In layers
          layer.Thickness \*= 2
        Next

        wall.WallType = newWallType
        Exit For
      End If
    Next

    Return IExternalCommand.Result.Succeeded
  Catch ex As Exception
    message = ex.ToString()
    Return IExternalCommand.Result.Failed
  End Try
End Function
```

This command iterates over the currently selected elements. If a wall has been selected, its wall type is retrieved and duplicated to create a new wall type. In its compound layer structure, the thickness of every layer is doubled, and the new wall type is assigned to the selected wall. The command terminates as soon as the first selected wall has been processed.

Here is a different example in C# duplicating a column type, setting a new fixed value for its radius:

```csharp
public CmdResult Execute(
  ExternalCommandData commandData,
  ref string message,
  ElementSet elements )
{
  Application app = commandData.Application;
  Document doc = app.ActiveDocument;

  string familyName = "Concrete-Round-Column";
  Filter f = app.Create.Filter.NewFamilyFilter(
    familyName );

  List<RvtElement> families = new List<RvtElement>();
  doc.get\_Elements( f, families );
  if( 1 > families.Count )
  {
    message = "No suitable family found.";
    return CmdResult.Failed;
  }

  Family fam = families[0] as Family;
  FamilySymbol famSym = null;
  foreach( FamilySymbol fs in fam.Symbols )
  {
    famSym = fs;
    break;
  }

  // create a new family symbol using Duplicate:
  string newFamilyName = "NewRoundColumn 3";
  FamilySymbol newFamSym = famSym.Duplicate(
    newFamilyName ) as FamilySymbol;

  // set the radius to a new value:
  Parameter par = newFamSym.get\_Parameter( "b" );
  par.Set( 3 );

  return CmdResult.Succeeded;
}
```