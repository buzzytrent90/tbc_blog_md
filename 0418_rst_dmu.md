---
post_number: "0418"
title: "Structural Dynamic Model Update Sample"
slug: "rst_dmu"
author: "Jeremy Tammik"
tags: ['csharp', 'elements', 'family', 'filtering', 'geometry', 'levels', 'parameters', 'revit-api', 'transactions']
source_file: "0418_rst_dmu.htm"
original_url: "https://thebuildingcoder.typepad.com/blog/0418_rst_dmu.html"
---

### Structural Dynamic Model Update Sample

I am away again on the next leg of my extensive vacations this summer, travelling a bit further afield this time.
After one night in a plane and another night in a bus I find myself on the island of
[Ko Tao](http://en.wikipedia.org/wiki/Ko_Tao) in
Thailand, where I completed my first open sea water dive in the
[Japanese Garden](http://en.wikipedia.org/wiki/Diving_in_Ko_Tao) with Boris of
[Alvaro Diving](http://www.divingcourseskohtao.com).

Since I will be away for the whole month of August, I decided to take the computer with me this time.
I have a couple of interesting questions lined up that I have been wanting to discuss for quite a while now, and hope to finally get around to doing that now.

Here is the first one, a neat sample of using the
[dynamic model update](http://thebuildingcoder.typepad.com/blog/2010/04/element-level-events.html#2)
features in the context of Revit Structure created by and presented at the recent
[AEC DevCamp](http://thebuildingcoder.typepad.com/blog/2010/07/api-training-and-aec-devcamp-material.html) in Boston by Saikat Bhattacharya.

It causes all concrete beam rebars in the model to automatically resize every time the beam geometry changes.

Here is Saikat's description:

#### Dynamic Model Update Sample in the Structural Context

This sample was written to illustrate one of the many possible uses of the new platform API feature called Dynamic Model Update (DMU) in a Structural context in Revit Structure. For more on the basics of on DMU and how to use this new mechanism, we can refer to the chapter 25 (page 283) of the Revit API Developers Guide PDF document located in the SDK. It causes concrete beam rebars (included in the attached project) to automatically resize every time the beam geometry changes.

This sample creates a new ribbon in the Analyze tab with two radio group button which can be used to enable the alignment functionality in the sample to be active or not. If the alignment option is on (or active), every resize of the beam in the model resizes the rebars in the beam to perfectly align it between the two opposite faces of the beam.

This sample uses the IExternalApplications OnStartup event to create a ribbon with two radio buttons on the Analyze tab. It also registers the updater class and adds triggers to it. The trigger is set to work with implicit set of elements which are of Structural Framing category (beams) which is specified using an ElementCategoryFilter and is set to work with change of scope, specifically change in Geometry of the elements (beams).

The sample implements two external commands called UIDynamicModelUpdateOn and UIDynamicModelUpdateOff which are executed when users click on the radio group buttons in the ribbon panel in the analyze tab. These commands internally just set a local variable which stores whether the alignment of the rebars functionality is activated or not.

The sample also contains an Updater class called RebarUpdater which implements the IUpdater interface. This interface requires implementing certain methods to return the updater Id, updater name, additional (auxiliary) information and the execute method which gets informed of the change of scope (geometry) of the elements (beams). This Execute method is called when there is a change of geometry of the beam in the given set up in RebarUpdater class. In this Execute method, the code gets access to all the modified beams using the data.GetModifiedElementIds and then iterates through each beam. It uses the new element iteration API to create a collection of all the rebars and calculates the beam line of each beam. This beam line helps provide the start and end point of the beam which is the extent to which the rebars would be stretched or compressed to. Following this, the sample moves the rebar to one end of the beam using the doc.Move method and passing in a translational vector which is based on the beam and rebar start points. After the rebar has been moved, the beam line needs to be recalculated again and using the length from the beam line, we can set the corresponding length parameter of the rebar to get it in sync with the new length of the beam. In the included RVT file, the rebar parameter \*B\* refers to the length of the rebar which is used to set the new length.

The intention of the sample is to show how the new Dynamic Model Update API can be used in a structural context inside Revit Structure. This sample assumes you are using only one beam which has single or multiple rebars contained in it and needs to be extensively tested and modified to work with broader types of beams and rebars.

Here is the entire code:
```csharp
[Transaction( TransactionMode.Automatic )]
[Regeneration( RegenerationOption.Manual )]
public class AlignRebar : IExternalApplication
{
  /// <summary>
  /// On shutdown, unregister the updater
  /// </summary>
  public Result OnShutdown( UIControlledApplication a )
  {
    RebarUpdater updater
      = new RebarUpdater( a.ActiveAddInId );

    UpdaterRegistry.UnregisterUpdater(
      updater.GetUpdaterId() );

    return Result.Succeeded;
  }

  /// <summary>
  /// On start up, add UI buttons, register
  /// the updater and add triggers
  /// </summary>
  public Result OnStartup( UIControlledApplication a )
  {
    // Add the UI buttons on start up
    AddButtons( a );

    RebarUpdater updater = new RebarUpdater(
      a.ActiveAddInId );

    // Register the updater in the singleton
    // UpdateRegistry class
    UpdaterRegistry.RegisterUpdater( updater );

    // Set the filter; in this case we
    // shall work with beams specifically
    ElementCategoryFilter filter
      = new ElementCategoryFilter(
        BuiltInCategory.OST\_StructuralFraming );

    // Add trigger
    UpdaterRegistry.AddTrigger(
      updater.GetUpdaterId(), filter,
      Element.GetChangeTypeGeometry() );

    return Result.Succeeded;
  }

  /// <summary>
  /// Add UI buttons
  /// </summary>
  public void AddButtons( UIControlledApplication a )
  {
    // create a ribbon panel on the Analyze tab
    RibbonPanel panel = a.CreateRibbonPanel(
      Tab.Analyze, "RST Labs" );

    AddDmuCommandButtons( panel );
  }

  /// <summary>
  /// Control buttons for the Dynamic Model Update
  /// </summary>
  public void AddDmuCommandButtons(
    RibbonPanel panel )
  {
    string path = GetType().Assembly.Location;

    string sDirectory
      = System.IO.Path.GetDirectoryName( path );

    // create toggle buttons for radio button group

    ToggleButtonData toggleButtonData3
      = new ToggleButtonData(
        "RSTLabsDMUOff", "Align Off", path,
        "RstAvfDmu.UIDynamicModelUpdateOff" );

    toggleButtonData3.LargeImage
      = new BitmapImage( new Uri( sDirectory
        + "\\Images\\" + "Families.ico" ) );

    ToggleButtonData toggleButtonData4
      = new ToggleButtonData(
        "RSTLabsDMUOn", "Align On", path,
        "RstAvfDmu.UIDynamicModelUpdateOn" );

    toggleButtonData4.LargeImage
      = new BitmapImage( new Uri( sDirectory
        + "\\Images\\" + "Families.ico" ) );

    // make dyn update on/off radio button group

    RadioButtonGroupData radioBtnGroupData2 =
      new RadioButtonGroupData( "RebarAlign" );

    RadioButtonGroup radioBtnGroup2
      = panel.AddItem( radioBtnGroupData2 )
        as RadioButtonGroup;

    radioBtnGroup2.AddItem( toggleButtonData3 );
    radioBtnGroup2.AddItem( toggleButtonData4 );
  }
}

public class RebarUpdater : IUpdater
{
  public static bool m\_updateActive = false;
  AddInId addinID = null;
  UpdaterId updaterID = null;

  public RebarUpdater( AddInId id )
  {
    addinID = id;
    // UpdaterId that is used to register and
    // unregister updaters and triggers
    updaterID = new UpdaterId( addinID, new Guid(
      "63CDBB88-5CC4-4ac3-AD24-52DD435AAB25" ) );
  }

  /// <summary>
  /// Align rebar to updated beam
  /// </summary>
  public void Execute( UpdaterData data )
  {
    if( m\_updateActive == false ) { return; }

    // Get access to document object
    Document doc = data.GetDocument();

    try
    {
      // Loop through all the modified elements
      foreach( ElementId id in
        data.GetModifiedElementIds() )
      {
        FamilyInstance beam = doc.get\_Element( id )
          as FamilyInstance;

        // Create a filter to retrieve all rebars
        FilteredElementCollector rebars
          = new FilteredElementCollector( doc );

        rebars.OfCategory( BuiltInCategory.OST\_Rebar );
        rebars.OfClass( typeof( Rebar ) );

        foreach( Rebar rebar in rebars )
        {
          // Calculate the beam line
          XYZ beamStartPoint = new XYZ();
          XYZ beamEndPoint = new XYZ();
          Line line = CalculateBeamLine( beam );

          // Get the start and end point
          if( null != line )
          {
            beamStartPoint = line.get\_EndPoint( 0 );
            beamEndPoint = line.get\_EndPoint( 1 );
          }

          // To align the rebar to the new beam's
          // length, we split the tasks in two stages
          // Step 1: Move the rebar to align with one
          // of the end of the beam

          // For this we first access the
          // rebar line geometry
          Line rebarLine = rebar.Curves.get\_Item( 0 )
            as Line;

          // Calculate the translation vector
          // (the extent of the move)
          XYZ transVec = new XYZ( beamStartPoint.X
            - rebarLine.get\_EndPoint( 0 ).X, 0.0, 0.0 );

          // Perform the move
          doc.Move( rebar, transVec );

          // This move causes the beam line to change
          // and so recalculating the beam line
          line = CalculateBeamLine( beam );

          // Step 2: Set the new length of the rebar
          // based on new beam length. For this, we
          // can set the relevant parameter after
          // checking at UI
          rebar.get\_Parameter( "B" ).Set(
            GetLength( line ) );
        }
      }
    }
    catch( Exception ex )
    {
      TaskDialog.Show( "Exception", ex.Message );
    }
  }

  // Calculate the beam line
  private Line CalculateBeamLine(
    FamilyInstance beam )
  {
    GeometryElement geoElement
      = beam.get\_Geometry( new Options() );

    if( null == geoElement
      || 0 == geoElement.Objects.Size )
    {
      throw new Exception(
        "Can't get the geometry of selected element." );
    }

    Line beamLine = null;

    foreach( GeometryObject geoObject
      in geoElement.Objects )
    {
      // get the driving path and vector of the beam
      beamLine = geoObject as Line;

      if( null != beamLine )
      {
        return beamLine;
      }
    }
    return null;
  }

  /// <summary>
  /// Return the length of the given line
  /// </summary>
  public static double GetLength( Line line )
  {
    XYZ v = line.get\_EndPoint( 1 ) - line.get\_EndPoint( 0 );
    return v.GetLength();
  }

  /// <summary>
  /// Return the auxiliary string
  /// </summary>
  public string GetAdditionalInformation()
  {
    return "Automatically align rebar to match beam";
  }

  /// <summary>
  /// Set the priority
  /// </summary>
  public ChangePriority GetChangePriority()
  {
    return ChangePriority.Rebar;
  }

  /// <summary>
  /// Return the updater Id
  /// </summary>
  public UpdaterId GetUpdaterId()
  {
    return updaterID;
  }

  /// <summary>
  /// Return the updater name
  /// </summary>
  public string GetUpdaterName()
  {
    return "Rebar alignment updater";
  }
}

[Transaction( TransactionMode.ReadOnly )]
[Regeneration( RegenerationOption.Manual )]
public class UIDynamicModelUpdateOff : IExternalCommand
{
  public Result Execute(
    ExternalCommandData commandData,
    ref string message,
    ElementSet elements )
  {
    RebarUpdater.m\_updateActive = false;
    return Result.Succeeded;
  }
}

[Transaction( TransactionMode.ReadOnly )]
[Regeneration( RegenerationOption.Manual )]
public class UIDynamicModelUpdateOn : IExternalCommand
{
  public Result Execute(
    ExternalCommandData commandData,
    ref string message,
    ElementSet elements )
  {
    RebarUpdater.m\_updateActive = true;
    return Result.Succeeded;
  }
}
```

The sample model included with the application includes two point loads, a rectangular concrete beam, and a rebar.
Here are the selected point loads:

![Point loads](img/rst_dmu_loads.png)

This is the beam itself:

![Concrete rectangular beam](img/rst_dmu_beam.png)

These are the rebar elements:

![Rebar](img/rst_dmu_rebar.png)

On selecting and dragging the beam around, the add-in automatically updates the rebar to follow it:

![Moving the beam automatically updates the rebar](img/rst_dmu_updated.png)

Here is
[RstAvfDmu.zip](zip/RstAvfDmu.zip)
including the complete source code and Visual Studio solution for this sample.

This solution actually includes another additional sample, which demonstrates a use of the analysis visualisation framework in a structural context, to simulate a visual display a load distribution.
We might get around to presenting that in detail as well one of these days, or you can just go ahead and explore it for yourself.