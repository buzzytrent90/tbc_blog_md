---
post_number: "0291"
title: "Extra Transaction Required"
slug: "extra_transaction"
author: "Jeremy Tammik"
tags: ['csharp', 'elements', 'family', 'geometry', 'parameters', 'revit-api', 'transactions', 'views']
source_file: "0291_extra_transaction.htm"
original_url: "https://thebuildingcoder.typepad.com/blog/0291_extra_transaction.html"
---

### Extra Transaction Required

Here is another interesting issue to discuss in between the series of instalments of background information on the Revit API geometry library from Scott's Autodesk University presentation on
[analysing building geometry](http://thebuildingcoder.typepad.com/blog/2010/01/analyse-building-geometry.html).

As we mentioned repeatedly in the past, Revit automatically opens a transaction when an external command is invoked, and closes or aborts it when the command terminates, depending on the command result returned by the Execute method.
This affects the
[document IsModified property](http://thebuildingcoder.typepad.com/blog/2008/12/document-ismodified-property.html) and
clarifies why you need to open you own transactions for certain
[event handlers](http://thebuildingcoder.typepad.com/blog/2009/01/transactions.html) and
[modeless dialogues](http://thebuildingcoder.typepad.com/blog/2009/12/modeless-dialogues-in-revit.html),
forcing you to take on some
[transaction responsibility](http://thebuildingcoder.typepad.com/blog/2009/01/transaction-responsibility.html) yourself.
It is therefore always easier and more robust to
[use a modal dialogue](http://thebuildingcoder.typepad.com/blog/2009/06/export-family-instance-to-gbxml.html#1) instead of a modeless one, if that is possible.

Here is an example from a case handled by Joe Ye of a situation where we are forced to open our own transaction in spite of executing code within a normal modal external command context.
This is because in this specific situation, certain actions of our command depend on previous actions having completed, and this does apparently not always happen automatically.
By encapsulating the first step in its own transaction and closing that, the second step depending on it can execute correctly as well.

**Question:** I have created a new I beam in my Revit model.
I then use the Revit API to apply the following modifications to it:

1. Change the beam's "Vertical Projection" property to "Center of Beam".- Change the beam's "Z Direction Justification" property to "Center".- Create two rectangular openings to hold the beam using the beam's start and end points as center points and defining rectangles around them.

Of the two openings, the first is created in the wrong position, while the second one, at the end of the beam, is created correctly.
It seems as if the first opening is created before the "Z Direction Justification" is changed, and the second one afterwards.

**Answer:** This issue has to do with the model update.
The changes to the Z direction justification and vertical projection properties modify the model.
Apparently, these changes have not yet been completely applied to the model before the creation of the openings is launched.
The complete changes are only guaranteed to be applied to the model after the external command has completely terminated and its automatic transaction has been closed.

In this case, it appears that the property modifications have not been applied when the first opening is created.
The creation of the first opening affects the model in a way as to apply the property modifications as well, and therefore the second opening can make use of the updated properties, whereas the first one could not.
This explains why the first opening ended up in the wrong position and the second was correct.

The solution for this is to add a transaction to encapsulate the process of changing the two parameter values.
This makes sure the beam is updated.
After modifying the properties and closing the transaction, the two openings can be added successfully.

Here is a sample code snippet demonstrating this:
```csharp
using RvtElement = Autodesk.Revit.Element;
using CreationDocument = Autodesk.Revit.Creation.Document;
public IExternalCommand.Result Execute(
    ExternalCommandData commandData,
    ref string message,
    ElementSet elements )
{
  Application app = commandData.Application;
  Document doc = app.ActiveDocument;
  View view = commandData.View;

  ElementSet elemSet = view.Elements;

  IEnumerator elementEnumerator
    = elemSet.ForwardIterator();

  while( elementEnumerator.MoveNext() )
  {
    FamilyInstance fi = elementEnumerator.Current
      as FamilyInstance;

    if( null != fi )
    {
      CurveArray curves = new CurveArray();

      app.ActiveDocument.BeginTransaction();

      // set the Vertical Projection

      Parameter p = fi.get\_Parameter(
        BuiltInParameter.BEAM\_V\_JUSTIFICATION );

      p.Set( 1 );

      p = fi.get\_Parameter( BuiltInParameter
        .STRUCTURAL\_ANALYTICAL\_PROJECT\_MEMBER\_PLANE );

      if( p != null )
      {
        ElementId elemId = p.AsElementId();

        elemId.Value = -3;

        p.Set( ref elemId );
      }

      // Vertical Projection modification ends here

      app.ActiveDocument.EndTransaction();

      LocationCurve lc = fi.Location as LocationCurve;

      XYZ pointOnPlane = lc.Curve.get\_EndPoint( 0 );

      XYZ hostY = fi.FacingOrientation;
      XYZ hostX = fi.HandOrientation;
      XYZ hostZ = hostX.Cross( hostY );

      curves = CreateRectangle( app, pointOnPlane,
        hostX, hostZ, 0.5, 0.5 );

      Opening opening = doc.Create.NewOpening( fi,
        curves, CreationDocument.eRefFace.CenterY );

      pointOnPlane = lc.Curve.get\_EndPoint( 1 );

      curves = CreateRectangle( app, pointOnPlane,
        hostX, hostZ, 0.5, 0.5 );

      opening = doc.Create.NewOpening( fi,
        curves, CreationDocument.eRefFace.CenterY );
    }
  }
  return IExternalCommand.Result.Succeeded;
}
```

So the motto of this is that if you need to perform some model change immediately after some other model changing action, and the second change is closely related to the first, then it is safer to encapsulate the first change in a transaction to ensure that the second one correctly honours it.

Many thanks to Joe for handling this case!