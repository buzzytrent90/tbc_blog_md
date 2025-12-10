---
post_number: "1019"
title: "Rotating a Plan View"
slug: "rotate_plan_view"
author: "Jeremy Tammik"
tags: ['csharp', 'doors', 'elements', 'filtering', 'references', 'revit-api', 'rooms', 'sheets', 'transactions', 'views']
source_file: "1019_rotate_plan_view.htm"
original_url: "https://thebuildingcoder.typepad.com/blog/1019_rotate_plan_view.html"
---

### Rotating a Plan View

Here is a query from the Autodesk Revit API discussion forum dealing with the
[rotation of a plan view](http://forums.autodesk.com/t5/Autodesk-Revit-API/Rotate-a-viewplan-after-creation/td-p/3842914):

**Question:** I am trying to create rotated duplicates of existing views using the Revit 2013 API.

I managed to create the view, both as dependent and not.
After this I activate the view crop box and make it visible.

I then create a rotation transformation.
When I debug it, I can see that the transformation and rotation of the bounding box is ok.

I try to set the rotated bounding box as the crop box.
However, this does not work, and all I get is the standard identity matrix.

I have a right handed rotation, and use basis Z as the rotation axis.
I have a reference to a transaction inside the method, but have also tried to encapsulate the specific part into a sub transaction.

This way of doing things is according to the Revit 2013 API documentation.
It states that the bounding box can be rotated, then used as section box/crop box, and provides to following example code for this:

```csharp
private void RotateBoundingBox( View3D view3d )
{
  BoundingBoxXYZ box = view3d.SectionBox;

  if( !box.Enabled )
  {
    TaskDialog.Show( "Revit",
      "The View3D section box is not enabled." );
    return;
  }

  // Create a rotation transform

  Transform rotate = Transform.get\_Rotation(
    XYZ.Zero, XYZ.BasisZ, 2 );

  // Transform the SectionBox
  // with the rotation transfrom

  box.Transform = box.Transform.Multiply( rotate );
  view3d.SectionBox = box;
}
```

I tried to use it like this:

```csharp
  ViewPlan rotatedView = doc.GetElement(
    viewToDuplicate.Duplicate(
      ViewDuplicateOption.AsDependent ) )
        as ViewPlan;

  rotatedView.CropBoxActive = true;
  rotatedView.CropBoxVisible = true;

  BoundingBoxXYZ box = rotatedView.CropBox;

  XYZ originOfRotation = 0.5 \* (box.Max - box.Min);
  XYZ axizOfRotation = XYZ.BasisZ;

  Transform rotate = Transform.get\_Rotation(
    originOfRotation, axizOfRotation, Math.PI );

  box.Transform = box.Transform.Multiply( rotate );

  rotatedView.CropBox = box;
```

I also tried using the ElementTransformUtils.RotateElement method.

Both approaches yield no result.

Manually I make sure the plan is situated to 'Project north', show the crop box, select it, rotate it and the view is rotated accordingly.

The desired result is a dependent view rotated at the point of creation.
I need to create several views at once, e.g. 4-8 differently rotated ones of the same plan, to create evacuation plans.

Let me reiterate my requirement and manual solution:

#### Issue – Rotating view plans

For creating evacuation plans for a floor, I need to create rotated dependent views so that the person looking at the plan on the inside of a door sees the situation the same way as the corridor outside.

I can achieve it manually like this:

- Create the floor, i.e. when modelling the building.
- Apply all the required fire symbols.
- Create three duplicates as dependent views.
- Apply the crop region to each.
- Rotate each plan individually by rotating the crop box.
- Create filled regions to show which room the current plan applies to.
- Hide the 'you are here' symbols in all other rooms.
- Optionally rotate the symbols as well.
- Place these views on corresponding sheets.

This is obviously a very time consuming operation.

I have already implemented methods to create the fire and evacuation plans.

The issue I am now dealing with is the rotation of the crop region.

I do get the transformation, but I am not able to apply it to the crop region.
Something fails and the transformation remains unchanged.

How can I fix this, please?

**Answer:** Yes,
[rotating a view](http://cad-notes.com/2010/04/rotating-revit-views) in
the user interface is perfectly straightforward, just as you say:

In the API, the issue is also clear: The element to rotate is not the view, but rather the crop box element associated to it.

If it is visible, it can be found using a filtered element collector taking document and view element id arguments.

Here is a macro that works based on your sample code:

```csharp
  public void CreateDuplicatedRotatedCroppedView(
    Document doc )
  {
    View activeView = doc.ActiveView;
    View duplicated = null;

    using( Transaction t = new Transaction( doc ) )
    {
      t.Start( "Duplicate View" );

      duplicated = doc.GetElement(
        activeView.Duplicate(
          ViewDuplicateOption.WithDetailing ) )
            as View;

      t.Commit();
    }

    Element cropBoxElement = null;

    using( TransactionGroup tGroup
      = new TransactionGroup( doc ) )
    {
      tGroup.Start( "Temp to find crop box element" );

      using( Transaction t2 = new Transaction(
        doc, "Temp to find crop box element" ) )
      {
        // Deactivate crop box

        t2.Start();
        duplicated.CropBoxVisible = false;
        t2.Commit();

        // Get all visible elements;
        // this excludes hidden crop box

        FilteredElementCollector collector
          = new FilteredElementCollector(
            doc, duplicated.Id );

        ICollection<ElementId> shownElems
          = collector.ToElementIds();

        // Activate crop box

        t2.Start();
        duplicated.CropBoxVisible = true;
        t2.Commit();

        // Get all visible elements excluding
        // everything except the crop box

        collector = new FilteredElementCollector(
          doc, duplicated.Id );
        collector.Excluding( shownElems );
        cropBoxElement = collector.FirstElement();
      }
      tGroup.RollBack();
    }

    using( Transaction t3 = new Transaction( doc ) )
    {
      BoundingBoxXYZ bbox = duplicated.CropBox;

      XYZ center = 0.5 \* ( bbox.Max + bbox.Min );

      Line axis = Line.CreateBound(
        center, center + XYZ.BasisZ );

      t3.Start( "Rotate crop box element" );

      ElementTransformUtils.RotateElement( doc,
        cropBoxElement.Id, axis, Math.PI / 6.0 );

      t3.Commit();
    }
  }
```

There are several noteworthy points in here.

All the transactions could be assimilated into one group, if desired.

The crop box element is retrieved using a clever trick: first hide it, retrieve the set V of all visible elements, unhide it and again retrieve all visible elements, this time excluding the set V.
Clever, huh?