---
post_number: "1367"
title: "Sheet To Model Coord"
slug: "sheet_to_model_coord"
author: "Jeremy Tammik"
tags: ['csharp', 'elements', 'geometry', 'levels', 'python', 'references', 'revit-api', 'sheets', 'transactions', 'views', 'walls']
source_file: "1367_sheet_to_model_coord.md"
original_url: "https://thebuildingcoder.typepad.com/blog/1367_sheet_to_model_coord.html"
---

### Sheet to Model Coordinate Conversion
James Carroll raised
a [question](http://thebuildingcoder.typepad.com/blog/2013/10/exact-viewport-positioning-conceptual-design-automation-and-graitec.html#comment-2296469077) on the discussion
of [exact viewport positioning](http://thebuildingcoder.typepad.com/blog/2013/10/exact-viewport-positioning-conceptual-design-automation-and-graitec.html):
\*\*Question:\*\*
You mentioned that "there is currently a gap in the API related to converting between model and sheet coordinates."
This is exactly what we are trying to do!
We want to get element boundary geometry in paper space.
Is this gap still present in the Revit 2016 API?
Are you aware of a workaround?
\*\*Answer:\*\*
There is still a gap, as you will see if you read through Miro's sample below.
It is however possible to quite easily convert from sheet coordinates to Revit model coordinates, e.g., to place model text notes and other geometric elements based on input read from something in sheet coordinates, for example a linked DWFX file.
Paolo Serra, Technical BIM Consultant at Autodesk in Milano, provides a nice example demonstrating this:
> I've been doing something related to this subject for a client in Italy, given that the level height is the same as the view in the viewport on the sheet I collected the DWF markups (polyLines and arcs) to write a textnote, for instance, or to place families... here is something I did on the fly... hope it helps.
\*\*Jeremy says:\*\*
I implemented a new external
command [CmdSheetToModel](https://github.com/jeremytammik/the_building_coder_samples/blob/master/BuildingCoder/BuildingCoder/CmdSheetToModel.cs)
in [The Building Coder](https://github.com/jeremytammik/the_building_coder_samples) samples
version [2016.0.121.0](https://github.com/jeremytammik/the_building_coder_samples/releases/tag/2016.0.121.0) to
host and test Paolo's code.
#### Converting Sheet to Model Coordinate Sample
I ran it in a trivial Revit sample model with a linked DWFX file attached to it:
![Sample model with linked DWFX](img/sheet_to_model_01.png)
Here is the viewport that we can use to grab coordinates from:
![Viewport](img/sheet_to_model_02.png)
The sample command reads the DWFX markup in sheet coordinates, converts them to model coordinates, and places a text element at the centre of each DWFX arc element:
![Text notes added based on DWFX markup location](img/sheet_to_model_03.png)
The generated test notes on each DWFX arc look like this:
![Sample model with linked DWFX](img/sheet_to_model_04.png)
As said, I implemented a new external
command [CmdSheetToModel](https://github.com/jeremytammik/the_building_coder_samples/blob/master/BuildingCoder/BuildingCoder/CmdSheetToModel.cs)
in [The Building Coder](https://github.com/jeremytammik/the_building_coder_samples) samples
version [2016.0.121.0](https://github.com/jeremytammik/the_building_coder_samples/releases/tag/2016.0.121.0) to implement and test this.
#### Setting Plan View Location within Sheet
Miroslav Schonauer, Solution Architect for Autodesk Consulting, has another interesting sample to share that shows a very powerful approach that is hopefully edifying and useful to many other add-in developers as well and also hits the limits:
I have an interesting requirement to set each Plan View within Sheet (always that single view on it!) so that the \*physical model\* X-Y position is always the same among different sheets, regardless of the View/Viewport shape, crops, etc.
This is so that when multiple printed Sheets are 'overlapped', the Plans coordinate systems (physical XY position) are also overlapped…
After a lot of research about the Sheet, Viewport and View classes, I thought I worked out how to do the task. For simplicity, I temporarily use single-sheet to test. In the code, I also made assumptions that the Sheet is active and that there are 'Level 0' Floor Plan (Master to which to align to) and several other Floor Plans (Slaves whose Viewports are to be moved), all of the same Scale.
It all works great apart when aligning the views which are cropped, BUT for some reason one of their Elevation symbols although outside the crop box STILL shows in the View. Strangely, it always seems to be East Elevation – if that one is left outside the crop box, it still shows in the Viewport, all others don't!
Two questions:
- Is this a bug in UI that the Elevation Symbol shows in plan Viewport even though it's outside of the crop box?
- Regardless of the answer to above, is there a way to adjust my code to cover that case as well? The problem is that I made hopefully reasonable assumption that the centre of View Outline is also the centre of Viewport (from my number-crunching analyses, Revit seems to add a 0.01ft buffer on all 4 sides to the former in order to create the latter), but that assumption is not valid when the elevation symbol outside the very cropped view is 'bundled' alongside that view in its sheet viewport L.
The code command is fully copied below and I attach a 2015 RVT model with walls used to write some sample text that can be used to repro the problem. The 'culprit' View is 'Level 1 View2' for which it can be seen that Elevation Symbols outside crop box is displayed. Other 2 views ('Level 0 Test1' and 'Level 1') align spot-on.
Before command:
![Before command](img/sheet_to_model_05.png)
After command:
![After command](img/sheet_to_model_06.png)
Here is the code I use to test this:

```
[Transaction( TransactionMode.Automatic )]
public class CmdMiroTest2 : IExternalCommand
{
  // KIS - public fields
  public UIApplication _appUI = null;
  public ApplicationRvt _app = null;
  public UIDocument _docUI = null;
  public Document _doc = null;

  Result IExternalCommand.Execute(
    ExternalCommandData commandData,
    ref string message,
    ElementSet elements )
  {
    // cache admin data
    _appUI = commandData.Application;
    _app = _appUI.Application;
    _docUI = commandData.Application.ActiveUIDocument;
    _doc = _docUI.Document;

    try // generic
    {
      // Current View must be Sheet
      ViewSheet sheet = _doc.ActiveView as ViewSheet;
      if( null == sheet )
      {
        Util.ErrorMsg( "Current View is NOT a Sheet!" );
        return Result.Cancelled;
      }

      // There must be a Floor Plan named "Level 0"
      // which is the "master" to align to
      Viewport vpMaster = null;
      // There must be at least one more Floor Plan
      // View to align (move)
      List<Viewport> vpsSlave = new List<Viewport>();
      // Find them:
      foreach( ElementId idVp in sheet.GetAllViewports() )
      {
        Viewport vp = _doc.GetElement( idVp ) as Viewport;
        ViewRvt v = _doc.GetElement( vp.ViewId ) as ViewRvt;
        if( v.ViewType == ViewType.FloorPlan )
        {
          if( v.Name.Equals( "Level 0", StringComparison
            .CurrentCultureIgnoreCase ) )
          {
            vpMaster = vp;
          }
          else
          {
            vpsSlave.Add( vp );
          }

        } //if FloorPlan

      } //foreeach idVp

      // Check if got them all
      if( null == vpMaster )
      {
        Util.ErrorMsg( "NO 'Level 0' Floor Plan on the Sheet!" );
        return Result.Cancelled;
      }
      else if( vpsSlave.Count == 0 )
      {
        Util.ErrorMsg( "NO other Floor Plans to adjust on the Sheet!" );
        return Result.Cancelled;
      }

      // Process Master
      // --------------

      XYZ ptMasterVpCenter = vpMaster.GetBoxCenter();
      ViewRvt viewMaster = _doc.GetElement(
        vpMaster.ViewId ) as ViewRvt;
      double scaleMaster = viewMaster.Scale;

      // Process Slaves
      // --------------

      foreach( Viewport vpSlave in vpsSlave )
      {
        XYZ ptSlaveVpCenter = vpSlave.GetBoxCenter();
        ViewRvt viewSlave = _doc.GetElement(
          vpSlave.ViewId ) as ViewRvt;
        double scaleSlave = viewSlave.Scale;
        // MUST be the same scale, otherwise can't really overlap
        if( scaleSlave != scaleMaster ) continue;

        // Work out how to move the center of Slave
        // Viewport to coincide model-wise with Master
        // (must use center as only Viewport.SetBoxCenter
        // is provided in API)
        // We can ignore View.Outline as Viewport.GetBoxOutline
        // is ALWAYS the same dimensions enlarged by
        // 0.01 ft in each direction.
        // This guarantees that the center of View is
        // also center of Viewport, BUT there is a
        // problem when any Elevation Symbols outside
        // the crop box are visible (can't work out why
        // - BUG?, or how to calculate it all if BY-DESIGN)

        BoundingBoxXYZ bbm = viewMaster.CropBox;
        BoundingBoxXYZ bbs = viewSlave.CropBox;

        // 0) Center points in WCS
        XYZ wcsCenterMaster = 0.5 * bbm.Min.Add( bbm.Max );
        XYZ wcsCenterSlave = 0.5 * bbs.Min.Add( bbs.Max );

        // 1) Delta (in model's feet) of the slave center w.r.t master center
        double deltaX = wcsCenterSlave.X - wcsCenterMaster.X;
        double deltaY = wcsCenterSlave.Y - wcsCenterMaster.Y;

        // 1a) Scale to Delta in Sheet's paper-space feet
        deltaX *= 1.0 / (double) scaleMaster;
        deltaY *= 1.0 / (double) scaleMaster;

        // 2) New center point for the slave viewport, so *models* "overlap":
        XYZ newCenter = new XYZ(
          ptMasterVpCenter.X + deltaX,
          ptMasterVpCenter.Y + deltaY,
          ptSlaveVpCenter.Z );
        vpSlave.SetBoxCenter( newCenter );
      }
    }
    catch( Exception ex )
    {
      Util.ErrorMsg( "Generic exception: " + ex.Message );
      return Result.Failed;
    }
    return Result.Succeeded;
  }
} // CmdMiroTest2
```

An update after some help from a local user expert – he explained that what I'm seeing in 'Level 1 View2' is due to the View's Annotation Crop settings.
So, it is now confirmed that Q1) is no bug, but it's legal and likely that there may be annotation elements outside the crop box within a Viewport.
Therefore, the whole feasibility of my task boils to the answer on now more-specific Q2:
As the Crop Box is NOT guaranteed to be centred within a Viewport, is there a way to determine where within the Viewport's Outline the Crop Box is positioned?
Or any other way to determine 2D Viewport to 3D Model mapping?
I am not too optimistic, but can someone pls confirm either way as I need to be sure about the feasibility?
\*\*Scott Conover responds:\*\*
This is a known gap tracked as an enhancement request in the change request REVIT-10373.
Thank you very much, Paolo and Miro, for sharing these important insights and samples!
I just added Miro's sample code to
the module [CmdSheetToModel](https://github.com/jeremytammik/the_building_coder_samples/blob/master/BuildingCoder/BuildingCoder/CmdSheetToModel.cs) as well,
in [The Building Coder](https://github.com/jeremytammik/the_building_coder_samples) samples
version [2016.0.121.1](https://github.com/jeremytammik/the_building_coder_samples/releases/tag/2016.0.121.1).