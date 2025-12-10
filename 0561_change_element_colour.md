---
post_number: "0561"
title: "Change Element Colour"
slug: "change_element_colour"
author: "Jeremy Tammik"
tags: ['csharp', 'elements', 'references', 'revit-api', 'selection', 'transactions', 'views']
source_file: "0561_change_element_colour.htm"
original_url: "https://thebuildingcoder.typepad.com/blog/0561_change_element_colour.html"
---

### Change Element Colour

We once looked at
[changing the colour of model or detail curves](http://thebuildingcoder.typepad.com/blog/2010/01/model-and-detail-curve-colour.html) by
setting their category line style.

Here is another simple little question, on setting the colour of an element, with a clear and simple answer, also by Joe Ye:

**Question:** How can I programmatically change the colour of an annotation symbol, e.g. a tag, in a Revit project?

**Answer:** You can change the colour of individual elements in a specified view using the View class ProjColorOverrideByElement property.
It takes a collection of elements as an argument and overrides their projection lines' colour in the given view.

Here is a VSTA code snippet showing how it can be used:
```csharp
public void ChangeElementColor()
{
  Application app = this.ActiveUIDocument
    .Application.Application;

  UIDocument uidoc = this.ActiveUIDocument;
  Document doc = uidoc.Document;

  Color color = app.Create.NewColor();
  color.Blue = ( byte ) 150;
  color.Red = ( byte ) 200;
  color.Green = ( byte ) 200;

  Selection sel = uidoc.Selection;

  Reference ref1 = sel.PickObject(
    ObjectType.Element,
    "Pick element to change its colour" );

  Element elem = ref1.Element;

  List<ElementId> ids = new List<ElementId>( 1 );
  ids.Add( elem.Id );

  Transaction trans = new Transaction( doc );
  trans.Start( "ChangeColor" );

  doc.ActiveView.set\_ProjColorOverrideByElement(
    ids, color );

  trans.Commit();
}
```

Many thanks to Joe for providing this solution!