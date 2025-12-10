---
post_number: "1112"
title: "Detach Beam from Plane"
slug: "beam_detach_plane"
author: "Jeremy Tammik"
tags: ['elements', 'family', 'filtering', 'levels', 'parameters', 'python', 'references', 'revit-api', 'selection', 'transactions']
source_file: "1112_beam_detach_plane.htm"
original_url: "https://thebuildingcoder.typepad.com/blog/1112_beam_detach_plane.html"
---

### Detach Beam from Plane

Here is a long-standing question raised once again now by Miroslav Schonauer of Autodesk Consulting and solved with help from Sasha Crotty of the Revit development team:

**Question:** Is there a way to programmatically replicate the 'Detach from Plane' functionality accessible in the user interface through the beam context menu?

I can right click on a structural beam and select 'Detach from Plane'.
Revit removes the parameter 'Work plane', i.e. BuiltInParameter.SKETCH\_PLANE\_PARAM, and enables the 'Reference Level' parameter, which is read-only otherwise.
I would like to replicate this behaviour using the API.

I tried to delete the Work plane parameter but that did not help.
Even the FamilyInstance.Host for a beam is returned as read-only.

Is there a way to achieve this using the API?

**Answer:** Changing the end elevation of the beam using the end elevation parameters will force it to go out of plane.
That should also detach it from the reference plane.

**Response:** I need the beam to remain in the same physical position, so should I do it like this, then?

- First transaction: move the end elevations +dv in plane normal direction.
- Second transaction: move it back by –dv.

**Answer:** I think you may be able to get away without two transactions.
Move the beam and then move it back in the same transaction.

If that doesn't work, you could use a TransactionGroup and assimilate the two transactions into one.

**Response:** Thanks all a lot.
It works like a charm, all from a single transaction!
Here is some sample code:

```python
  class SelectionFilterBeam : ISelectionFilter
  {
    public bool AllowElement( Element e )
    {
      if( !(e is FamilyInstance) )
      {
        return false;
      }

      if( e.Category.Id.IntegerValue != (int)
        BuiltInCategory.OST\_StructuralFraming )
      {
        return false;
      }

      // In theory could still be a Brace, but
      // Structural Usage is sometimes NOT set and
      // cannot be relied upon!
      // So, good enough as "Beam" if here.

      return true;
    }

    public bool AllowReference( Reference r, XYZ p )
    {
      return true;
    }
  }

  [Transaction( TransactionMode.Automatic )]
  public class CmdDetachBeamFromPlane : IExternalCommand
  {
    public Result Execute(
      ExternalCommandData commandData,
      ref string message,
      ElementSet elements )
    {
      UIApplication uiapp = commandData.Application;
      UIDocument uidoc = uiapp.ActiveUIDocument;
      Document doc = uidoc.Document;

      // Pick a beam

      SelectionFilterBeam selFilterBeam
        = new SelectionFilterBeam();

      Reference r = uidoc.Selection.PickObject(
        ObjectType.Element, selFilterBeam,
        "Select a Beam to 'Detach From Plane'" );

      FamilyInstance beam = doc.GetElement( r )
        as FamilyInstance;

      // Check if it has 'Work Plane' to detach
      //
      // One would expect that it is simplest to
      // check .Host as commented below, BUT there
      // are some strange situations where Host IS
      // null and Revit STILL displays (as read-only):
      // "Work Plane = <not associated>" !?
      //
      //if (null == beam.Host)
      //{
      //  MessageBox.Show("Selected Family Instance of 'Structural Framing' Category has NO 'Work Plane'!");
      //  return Result.Cancelled;
      //}
      //
      // So, must check if that read-only SKETCH\_PLANE\_PARAM param exists:

      if( null == beam.get\_Parameter(
        BuiltInParameter.SKETCH\_PLANE\_PARAM ) )
      {
        MessageBox.Show( "Selected Family Instance "
          + "of 'Structural Framing' Category has NO "
          + "'Work Plane'!" );

        return Result.Cancelled;
      }

      // In theory, the plane could be non-horizontal
      // but in 99% should be and in 99.99% for beams
      // it would NOT be vertical which is the only
      // case that would not work using the following
      // (adjusting the elevation):
      // As .Host is RO property, the workaround is
      // to move the END ELEVATIONs outside the plane
      // which will internally "detach" it in Revit,
      // then simply move back!
      // Note that Moving the element would not work
      // as it is constrained to the plane.

      double elevOldSta = beam.get\_Parameter(
        BuiltInParameter.STRUCTURAL\_BEAM\_END0\_ELEVATION )
          .AsDouble();

      double elevOldEnd = beam.get\_Parameter(
        BuiltInParameter.STRUCTURAL\_BEAM\_END1\_ELEVATION )
          .AsDouble();

      double elevTmpSta = elevOldSta + 1.0;
      double elevTmpEnd = elevOldEnd + 1.0;

      // This will "detach from plane"...

      beam.get\_Parameter(
        BuiltInParameter.STRUCTURAL\_BEAM\_END0\_ELEVATION )
          .Set( elevTmpSta );

      beam.get\_Parameter(
        BuiltInParameter.STRUCTURAL\_BEAM\_END1\_ELEVATION )
          .Set( elevTmpEnd );

      // ...and this move back to the
      // same original position

      beam.get\_Parameter(
        BuiltInParameter.STRUCTURAL\_BEAM\_END0\_ELEVATION )
          .Set( elevOldSta );

      beam.get\_Parameter(
        BuiltInParameter.STRUCTURAL\_BEAM\_END1\_ELEVATION )
          .Set( elevOldEnd );

      MessageBox.Show( "Successfully removed the "
        + "'Work Plane' constraint" );

      return Result.Succeeded;
    }
  }
```

Please note the numerous valuable hints included in the code comments.

Also note that a slight performance improvement might be achievable by adjusting just one end of the beam instead of both.

Finally, note that the code could be made more readable by defining shorthand variables for the lengthy built-in paramenter enumeration values :-)

Many thanks to Miro and Sascha for this useful solution!