---
post_number: "0855"
title: "Geometry Object Subcategory"
slug: "geom_obj_category"
author: "Jeremy Tammik"
tags: ['doors', 'elements', 'family', 'filtering', 'geometry', 'python', 'references', 'revit-api', 'schedules', 'selection', 'sheets', 'transactions', 'views']
source_file: "0855_geom_obj_category.htm"
original_url: "https://thebuildingcoder.typepad.com/blog/0855_geom_obj_category.html"
---

﻿

### Geometry Object Subcategory

I received and have started installing and learning to use my new laptop, which is a Mac, I'm finishing up my preparations for Autodesk University, and I'm working on preparations for upcoming developer conferences, so you can imagine there is rather a lot to do and 24 hours per day seem very limiting.
I love the Unix shell!
Goodbye, DOS box.

Thank goodness my colleague Joe Ye answered a case that I found interesting and would like to share with you, since Joe is busy preparing his own Chinese developer meetings and conferences:

**Question:** I am trying to determine the subcategory of each solid element in a family.

Can this be done using the API?

My family that has two different solids and they have different subcategories.

I want to know which solid I am looking at and figured that subcategories would be a good way to go, since they can be applied to solids in a family.

Please note that the family with the subcategories must be nested as a non-shared family and I have to find out what element geometry is what in the supercomponent family.

I know that I can find out what subcategory each solid element belongs to when I open the family, but can I do so when its loaded as well?

Is there any other way of identifying different geometry objects once a family instance has been placed in the project?
Besides looking at the geometrical aspects such as volume, areas etc.

**Answer:** Yes, you can retrieve the solid category from the family instance in the model.

The category information is stored in the Solid object's GraphicStyle property.

The following code retrieves all the solid category names from a selected family instance:
```python
[Transaction( TransactionMode.ReadOnly )]
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

    Selection sel = uidoc.Selection;

    Reference r = sel.PickObject( ObjectType.Element,
      "Please pick a family instance" );

    Element e = doc.GetElement( r );

    GeometryElement geoElem = e.get\_Geometry(
      new Options() );

    int n = 0;
    string s = string.Empty;

    foreach( GeometryObject obj in geoElem )
    {
      if( obj is GeometryInstance )
      {
        GeometryInstance geoInst
          = obj as GeometryInstance;

        GeometryElement geoElem2
          = geoInst.GetSymbolGeometry();

        foreach( GeometryObject geoObj2 in geoElem2 )
        {
          if( geoObj2 is Solid )
          {
            Solid solid = geoObj2 as Solid;

            ElementId id = solid.GraphicsStyleId;

            GraphicsStyle gStyle = doc.GetElement(
              id ) as GraphicsStyle;

            if( gStyle != null )
            {
              ++n;
              s += gStyle.GraphicsStyleCategory.Name
                + "\r\n";
            }
          }
        }
      }
    }

    TaskDialog.Show( n.ToString() + " Graphics Styles",
      s );

    return Result.Succeeded;
  }
}
```

Here is an example of running this and selecting one of the doors in the basic sample project:

![Sample project door instance](img/geom_obj_categories_1.png)

The following materials are listed for this instance:

![Solid graphics style categories](img/geom_obj_categories_2.png)

Many thanks to Joe for this answer!

#### Listing all Schedules Placed on a Sheet

Meanwhile, Saikat discusses yet another use of the
[temporary transaction and deletion trick](http://thebuildingcoder.typepad.com/blog/2012/11/temporary-transaction-trick-touchup.html) to
[determine all schedules placed on a given sheet](http://adndevblog.typepad.com/aec/2012/11/listing-schedules-placed-on-sheets-in-revit.html).

In this case, the trick is to temporarily delete the sheet and watch which schedules are wiped out by that operation.

Note that in a comment on that post, Phillip Miller points out a way to achieve the same result using a filtered element collector taking a view id argument.

Who is going to create a benchmark comparing the relative speed of these two approaches?