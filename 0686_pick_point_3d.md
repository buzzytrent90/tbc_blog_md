---
ai_optimized: true
author: Jeremy Tammik
complexity_score: 3.2
content_type: qa
optimization_date: '2025-12-11T11:44:14.394204'
original_url: https://thebuildingcoder.typepad.com/blog/0686_pick_point_3d.html
post_number: 0686
reading_time_minutes: 4
series: geometry
slug: pick_point_3d
source_file: 0686_pick_point_3d.htm
tags:
- csharp
- elements
- geometry
- references
- revit-api
- selection
- transactions
- views
title: Pick a Point in 3D
word_count: 718
---

### Pick a Point in 3D

Today is the second day of AU for me, the first 'real' day of the conference.
I woke up at two in the morning, got back to sleep again eventually, but gave up at five and thus had plenty of time for a final private dress rehearsal for my lecture on the
[extensible](http://thebuildingcoder.typepad.com/blog/2011/04/extensible-storage.html)
[storage](http://thebuildingcoder.typepad.com/blog/2011/05/extensible-storage-of-a-map.html) API
later today.
I needed that, since it is a couple of weeks since I submitted the material and all but forgot what it is about at all.
Now I will be busy all the rest of the day running back and forth between the DevLab and my lecture.

On a completely different topic, here is a question on picking points in space that has already come up a few times:

**Question:** The Selection.PickPoint method is limited to selecting points on the active work plane.
Is there any way to pick an arbitrary 3D point, regardless of work plane?

**Answer:** Just as you say, the PickPoint method only allows point selection in the current work plane.
As stated by the Revit API help file RevitAPI.chm, the various overloads of PickPoint prompt the user to pick a point on the active sketch plane using optionally specified
snap settings.
The Revit API does not offer any other method to pick a point arbitrarily in 3D space.

So what if you need to pick a point somewhere in space that is not on the active work plane?

Well, if you know a plane on which the point needs to be selected, you can temporarily set the active work plane to that plane.
This approach was actually already mentioned briefly in the discussion on
[snapping to individual points in a point cloud](http://thebuildingcoder.typepad.com/blog/2011/07/point-cloud-snap-and-freeze.html).
You need a sketch plane in your document in order to set the active work plane, so you may want to encapsulate the entire operation in a dummy transaction in which to create a temporary sketch plane for that purpose.

I implemented a new external command CmdPickPoint3d in The Building Coder samples to demonstrate this by prompting the user to first select a face on an element and then pick a point on that face.
The element face picked first is used to temporarily redefine the active work plane, on which the second point can be picked.

This is implemented by the method PickFaceSetWorkPlaneAndPickPoint:
```csharp
bool PickFaceSetWorkPlaneAndPickPoint(
  UIDocument uidoc,
  out XYZ point\_in\_3d )
{
  point\_in\_3d = null;

  Document doc = uidoc.Document;

  Reference r = uidoc.Selection.PickObject(
    ObjectType.Face,
    "Please select a planar face to define work plane" );

  Element e = doc.get\_Element( r.ElementId );

  if( null != e )
  {
    PlanarFace face
      = e.GetGeometryObjectFromReference( r )
        as PlanarFace;

    if( face != null )
    {
      Plane plane = new Plane(
        face.Normal, face.Origin );

      Transaction t = new Transaction( doc );

      t.Start( "Temporarily set work plane"
        + " to pick point in 3D" );

      SketchPlane sp = doc.Create.NewSketchPlane(
        plane );

      uidoc.ActiveView.SketchPlane = sp;
      uidoc.ActiveView.ShowActiveWorkPlane();

      try
      {
        point\_in\_3d = uidoc.Selection.PickPoint(
          "Please pick a point on the plane"
          + " defined by the selected face" );
      }
      catch( OperationCanceledException )
      {
      }

      t.RollBack();
    }
  }
  return null != point\_in\_3d;
}
```

Note that as always, we need to encapsulate the call to PickPoint in an exception handler in order to gracefully take care of the use cancelling the selection operation.

PickFaceSetWorkPlaneAndPickPoint is driven by the following trivial Execute mainline:
```csharp
public Result Execute(
  ExternalCommandData commandData,
  ref string message,
  ElementSet elements )
{
  UIApplication uiapp = commandData.Application;
  UIDocument uidoc = uiapp.ActiveUIDocument;

  XYZ point\_in\_3d;

  if( PickFaceSetWorkPlaneAndPickPoint(
    uidoc, out point\_in\_3d ) )
  {
    TaskDialog.Show( "3D Point Selected",
      "3D point picked on the plane"
      + " defined by the selected face: "
      + Util.PointString( point\_in\_3d ) );

    return Result.Succeeded;
  }
  else
  {
    message = "3D point selection failed";
    return Result.Failed;
  }
}
```

Here are a couple of screen snapshots showing this command in action:

- Prompt the user to select a face on an element:![](img/pick_point_3d_select_face.png)- Set the active work plane and prompt the user to pick a point on it:![](img/pick_point_3d_select.png)- Report the result:

![](img/pick_point_3d_result.png)

Here is
[version 2012.0.95.0](zip/bc_12_95.zip) of
The Building Coder samples including the new CmdPickPoint3d command.