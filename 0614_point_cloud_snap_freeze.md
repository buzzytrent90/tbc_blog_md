---
post_number: "0614"
title: "Point Cloud Snap and Freeze"
slug: "point_cloud_snap_freeze"
author: "Jeremy Tammik"
tags: ['csharp', 'elements', 'filtering', 'revit-api', 'selection', 'transactions', 'views', 'windows']
source_file: "0614_point_cloud_snap_freeze.htm"
original_url: "https://thebuildingcoder.typepad.com/blog/0614_point_cloud_snap_freeze.html"
---

### Point Cloud Snap and Freeze

Here are some questions that came up a while ago regarding point clouds:

**Question:** As far as I can tell, the standard object snap settings also control snapping to point clouds:

![Object snap settings](img/point_cloud_snap.png)

Is there any way to snap to individual points in the point cloud?

**Answer:** Revit itself never asks for the pick of an individual point in a point cloud, so there is no mechanism to support this in the API either.
Even when Revit prompts the user to pick things via snapping using the UI option 'snap to point cloud', it creates a small window, asks for 100 points, and is satisfied if there is at least one.
Revit does not do anything at all to determine the forward most point or the point closest to the cursor from the cloud.

Here is some sample code showing how to prompt the user to select a box containing points and use a PointCloudFilter to retrieve them:
```csharp
public void PromptForPointCloudSelection(
  UIDocument uiDoc,
  PointCloudInstance pcInstance )
{
  Application app = uiDoc.Application.Application;
  Selection currentSel = uiDoc.Selection;

  PickedBox pickedBox = currentSel.PickBox(
    PickBoxStyle.Enclosing,
    "Select region of cloud for highlighting" );

  XYZ min = pickedBox.Min;
  XYZ max = pickedBox.Max;

  // Transform points into filter

  View view = uiDoc.ActiveView;
  XYZ right = view.RightDirection;
  XYZ up = view.UpDirection;

  List<Plane> planes = new List<Plane>();

  // X boundaries

  bool directionCorrect = IsPointAbovePlane(
    right, min, max );

  planes.Add( app.Create.NewPlane( right,
    directionCorrect ? min : max ) );

  planes.Add( app.Create.NewPlane( -right,
    directionCorrect ? max : min ) );

  // Y boundaries

  directionCorrect = IsPointAbovePlane(
    up, min, max );

  planes.Add( app.Create.NewPlane( up,
    directionCorrect ? min : max ) );

  planes.Add( app.Create.NewPlane( -up,
    directionCorrect ? max : min ) );

  // Create filter

  PointCloudFilter filter = PointCloudFilterFactory
    .CreateMultiPlaneFilter( planes );

  Transaction t = new Transaction( uiDoc.Document );
  t.Start( "Highlight" );

  pcInstance.SetSelectionFilter( filter );

  pcInstance.FilterAction
    = SelectionFilterAction.Highlight;

  t.Commit();
}

private static bool IsPointAbovePlane(
  XYZ normal,
  XYZ planePoint,
  XYZ point )
{
  XYZ diff = point - planePoint;
  diff = diff.Normalize();
  double dotProduct = diff.DotProduct( normal );
  return dotProduct > 0;
}
```

If you don't want the user to have to create the small rectangular window for obtaining points, you may be able to do one of the following:

1. Prompt for element selection using the PickObject method and the ObjectType.PointOnElement, with a filter that is limited to PointCloudInstances or even one specific instance – it may however be difficult to get the cursor to switch from the "don't pick" option for this.- Prompt for picking a point using the PickPoint method. You may have to set an active sketch plane for the view to allow this work in 3D.

In either case, you should be able to obtain a 3D point representing the user's selection.
The point will have no real relationship to the view and the front of the point cloud.
Then you could use the View.ViewDirection property to compute a small rectangle centred on the picked point, and build planes for the PointCloudFilter through the boundary lines of the rectangle and extending parallel to the view direction.
That will give you a filter capable of intersecting the point cloud and returning actual points.
If too many points are returned, you may need a back clipping plane added to the filter to focus on the points nearest the user's eye position.

Note that View.EyePosition should not be used to compute the distance directly, since the eye position for orthographic views is very close to the model bounding box.
Instead, a plane should be computed normal to view direction passing through eye position, and the distance of points measured to this plane.

**Question:** In Revit, the point cloud can be moved around in the model.
Is it possible to disable this?

I want my point cloud to appear as a static object.
It should not be moved or modified in CAD environment once it is brought in.

**Answer:** You can lock the point cloud by pinning it.
In the API, simply set the property Element.Pinned to true.
This will prevent accidental moves.

Of course, the user can unpin the cloud if they really want to, but at least that requires a conscious decision.

A more complicated way to prevent changes would be to use the Dynamic Update Framework and post an error on an illegal operation; the user will have to cancel the action and restore the original state.

Another completely untested idea might possibly be to assign the point cloud to a workset and check it out with a fake user.