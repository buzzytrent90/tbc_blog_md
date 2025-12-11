---
ai_optimized: true
author: Jeremy Tammik
complexity_score: 2.8
content_type: qa
optimization_date: '2025-12-11T11:44:14.262182'
original_url: https://thebuildingcoder.typepad.com/blog/0611_refresh_graphics.html
post_number: '0611'
reading_time_minutes: 3
series: general
slug: refresh_graphics
source_file: 0611_refresh_graphics.htm
tags:
- csharp
- doors
- elements
- family
- filtering
- geometry
- parameters
- revit-api
- transactions
- views
- walls
- windows
title: Refresh Element Graphics Display
word_count: 652
---

### Refresh Element Graphics Display

Here is a little issue with an interesting workaround that I encountered in the past in similar incarnations in other CAD software packages besides Revit, so it is worthwhile making a note of this for potential future use almost anywhere.

The issue has to do with refreshing the graphics display of individual elements in the Revit user interface.

Committing the transaction, explicit regeneration, and even calling RefreshActiveView does not help.
One additional thing that sometimes does help in similar situations is saving the document, but that is obviously a last recourse.

The interesting workaround presented in this case suggests moving the affected elements by a certain amount using ElementTransformUtils.MoveElement, and then moving them back again.

One place where I used similar tricks in the past was in the AutoCAD ARX API, where you could trigger a regeneration of an entity by calling transformBy with an identity matrix on it, if nothing else helped.

Here is the context of the issue:

**Question:** I'm importing a DWG into a Revit family.
It contains a solid filled polygon on one layer.

I am trying to change its line colour via its sub-category.

Using the following code in VSTA changes the Object Style properly, and I can see the change in the Object Styles dialog:
```csharp
public void ObjectStyleTest()
{
  UIDocument uiDoc = this.ActiveUIDocument;
  Document doc = this.ActiveUIDocument.Document;

  Transaction tr = new Transaction( doc );
  tr.Start( "ObjectStyles" );

  Category cat = doc.Settings.Categories.get\_Item(
    BuiltInCategory.OST\_ImportObjectStyles );

  Category subCat = cat.SubCategories.get\_Item(
    "A--A28--\_F-2--" );

  subCat.LineColor = new Color( 0, 0, 255 );

  doc.Regenerate();

  tr.Commit();

  uiDoc.RefreshActiveView();
}
```

However, the already existing ImportInstance does not change colour as expected.
If I change the ObjectStyle colour in the user interface, then the ImportInstance does change colour.

I tried calling Regenerate and RefreshActiveView, but they have no effect.
Once I move the ImportInstance in the user interface, its colour updates properly.

**Answer:** You can solve this programmatically, but you need to use a little workaround.
One possible workaround is to move the elements back and forth a bit.
Here is your sample code enhanced to show how to do this:
```csharp
public void ChangeImportedCategoryColor(
  UIDocument uiDoc )
{
  Document doc = uiDoc.Document;

  Transaction tr = new Transaction( doc );
  tr.Start( "ObjectStyles" );

  Category cat = doc.Settings.Categories.get\_Item(
    BuiltInCategory.OST\_ImportObjectStyles );

  Category subCat = cat.SubCategories.get\_Item(
    "A--A28--\_F-2--" );

  subCat.LineColor = new Color( 0, 0, 255 );

  // <workaround>
  FilteredElementCollector coll
    = new FilteredElementCollector( doc )
      .OfClass( typeof( ImportInstance ) );

  foreach( Element e in coll )
  {
    ElementTransformUtils.MoveElement(
      doc, e.Id, XYZ.BasisX );

    ElementTransformUtils.MoveElement(
      doc, e.Id, XYZ.BasisX.Negate() );
  }
  // </workaround>

  tr.Commit();
}
```

As said, in a similar situation in AutoCAD, it was sufficient to call transformBy with an identity transformation.
I tried out whether it works making just one single call to MoveElement with a XYZ.Zero argument, instead of two, but that is not the case.
I also tested changing some element parameter values back and forth, but since that does not change the geometry (I guess) it did not do the trick.

Many thanks to Harry Mattison and Adam Nagy for providing this idea!

**Addendum:** As Rudolf points out in his
[comment](http://thebuildingcoder.typepad.com/blog/2011/07/refresh-element-graphics-display.html?cid=6a00e553e16897883301538fb68718970b#comment-6a00e553e16897883301538fb68718970b) below,
this workaround may need to be adjusted for hosted elements:

Moving windows or doors may cause errors if the translation is too large, causing the element to leave its host boundaries, or if the moving direction is locked for the element. For example, windows cannot be moved in the Wall.Orientation direction.

Therefore, it would be safer to move the elements by just a tiny amount instead of a whole foot, and also ensure that the movement direction is parallel to the host face. For example, for a window in a vertical wall, you might use XYZ.BasisZ.Multiply(tinyAmount) instead of XYZ.BasisX.