---
post_number: "0727"
title: "Updating the Wall Compound Layer Structure"
slug: "balaji_setcompoundstr"
author: "Jeremy Tammik"
tags: ['csharp', 'elements', 'references', 'revit-api', 'selection', 'transactions', 'walls']
source_file: "0727_balaji_setcompoundstr.htm"
original_url: "https://thebuildingcoder.typepad.com/blog/0727_balaji_setcompoundstr.html"
---

### Updating the Wall Compound Layer Structure

We have a new Revit API supporter on the DevTech team!

Balaji joined ADN DevTech a year or two ago and dived straight into supporting AutoCAD with great zest, setting new records in speed and efficiency.
Now he addressed his first Revit API case, on how to update a wall compound layer structure.

We touched on this topic in the past when discussing the new
[changes made to the CompoundStructure class](http://thebuildingcoder.typepad.com/blog/2011/03/many-issues-resolved.html#1264438) in
Revit 2012.

An example is also given by the CmdNewWallLayer external command in The Building Coder samples, which required a complete rewrite due to these changes when
[migrating The Building Coder samples to Revit 2012](http://thebuildingcoder.typepad.com/blog/2011/04/migrating-the-building-coder-samples-to-revit-2012.html).

Here is Balaji's case:

**Question:** I have been updating wall layers through the API, but I am having trouble with the layer thicknesses in a wall with a compound structure. What might I be doing wrong?

**Answer:** As explained in the ADN technical solution on
[Changes to compound layer don't apply to wall type](http://adn.autodesk.com/adn/servlet/devnote?siteID=4814862&id=16779684&linkID=4901650),
accessible to ADN members only, GetCompoundStructure works on a copy and the modifications to the compound structure do not change the original type until SetCompoundStructure is called.

Here is a snippet of sample code that uses the CompoundStructure SetLayerWidth method and works ok:
```csharp
  UIApplication uiApp = commandData.Application;
  UIDocument uiDoc = uiApp.ActiveUIDocument;
  Document doc = uiDoc.Document;

  Transaction trans = new Transaction( doc );
  trans.Start( "changeLayerWidth" );

  Reference ref1 = uiDoc.Selection.PickObject(
    ObjectType.Element, "Pick a wall" );

  ElementId id = ref1.ElementId;
  Element elem = doc.get\_Element( id );
  Wall wall = elem as Wall;
  if( wall == null )
    return Result.Failed;

  CompoundStructure cs
    = wall.WallType.GetCompoundStructure();

  double layerWidth = 0.2;
  int layerIndex = cs.GetFirstCoreLayerIndex();

  IList<CompoundStructureLayer> cslayers
    = cs.GetLayers();

  foreach( CompoundStructureLayer csl in cslayers )
  {
    cs.SetLayerWidth( layerIndex, layerWidth );
    layerIndex++;
  }
  wall.WallType.SetCompoundStructure( cs );
  trans.Commit();
```

I verified the total wall thickness and it does add up correctly.

Many thanks to Balaji for this solution, and congratulations on answering your first Revit API ADN case!
Welcome to the club!