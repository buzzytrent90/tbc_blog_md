---
post_number: "1468"
title: "The Building Coder"
slug: "hololens_exitpath"
author: "Jeremy Tammik"
tags: ['doors', 'references', 'revit-api', 'selection', 'sheets', 'transactions', 'views', 'windows']
source_file: "1468_hololens_exitpath.md"
original_url: "https://thebuildingcoder.typepad.com/blog/1468_hololens_exitpath.html"
---

### HoloLens Escape Path Waypoint JSON Exporter
Last week, I worked with Kean Walmsley on
his [entry for the first global Autodesk Hackathon: a HoloLens-based tool for navigating low visibility environments](http://through-the-interface.typepad.com/through_the_interface/2016/08/my-entry-for-autodesks-first-global-hackathon-a-hololens-based-tool-for-navigating-low-visibility-environments.html), resulting in the
new [HoloGuide Autodesk Hackathon project](http://through-the-interface.typepad.com/through_the_interface/2016/09/hologuide-an-autodesk-hackathon-project.html).
Watch this [three-minute video](https://www.youtube.com/watch?v=8MjBgiQZUzw) for a quick first impression of what it is all about:

This project is part of Kean's [HoloLens project series](http://through-the-interface.typepad.com/through_the_interface/hololens), including and not limited to:
- [Using HoloLens to display diagnostic information for building components](http://through-the-interface.typepad.com/through_the_interface/2016/08/using-hololens-to-display-diagnostic-information-for-building-components.html)
- [Scaling our Unity model in HoloLens](http://through-the-interface.typepad.com/through_the_interface/2016/08/scaling-our-unity-model-in-hololens.html)
- [Adding spatial sound to our Unity model in HoloLens](http://through-the-interface.typepad.com/through_the_interface/2016/08/adding-spatial-sound-to-our-unity-model-in-hololens-part-3.html)
- [Displaying your Unity model in 3D using HoloLens](http://through-the-interface.typepad.com/through_the_interface/2016/07/displaying-your-unity-model-in-3d-using-hololens.html)
My modest contribution is the [ExportWaypointsJson Revit add-in](https://github.com/jeremytammik/ExportWaypointsJson), an external application implementing the main command, exporting the waypoints, and a subsidiary option settings command, displaying a form, validating input and storing the user preferences.
The main command and the main functionality of the add-in simply prompts the user to pick a model line in the Revit project, traverses it, determines waypoints at predefined intervals and exports them to JSON for consumption by the HoloGuide visualisation.
In the process of implementing it, I was able to explore an issue I never looked at before, validating user input to a Windows form using the `ErrorProvider` class, `Validating` and `Validated` events.
I also implemented two different approaches to store the user-defined add-in option settings, in XML and JSON.
Today, I'll just present the external application class creating the ribbon panel with split button providing access to two commands. It reads the ribbon button icons from embedded resources and provides a method to ensure that the main command always remains the default current button.
I used the [stacked ribbon button panel SplitButtonOptionConcept](http://thebuildingcoder.typepad.com/blog/2016/09/stacked-ribbon-button-panel-options.html) approach to retain the main command button as the current default split button option. Thus the main command is always displayed at the top and immediately accessible with a single click, while access to the other subsidiary commands requires opening the split button drop-down options first.
I'll just present the main external application implementation of this add-in today.
If you would like to explore the areas that I have not covered today right away on your own, feel free to clone, compile and debug the project from
the [ExportWaypointsJson GitHub repository](https://github.com/jeremytammik/ExportWaypointsJson).
The version discussed here
is [release 2017.0.0.11](https://github.com/jeremytammik/ExportWaypointsJson/releases/tag/2017.0.0.11).
#### External Application Implementation
The add-in displays the following ribbon panel:
![ExportWaypointsJson ribbon panel](img/hololens_exit_path_ribbon_panel.png)
You can either click the main button, which is always displayed at the top as the current option, to trigger the main command, or drop down the rest of the stacked button contents to display the option button:
![ExportWaypointsJson main command and settings buttons](img/hololens_exit_path_ribbon_buttons.png)
I ran the application in a model of the Autodesk office building in Neuchâtel:
![Autodesk office building in Neuchâtel](img/hololens_adsk_office_bldg_3d.png)
The escape path we use for demonstration leads from Kean's desk to the nearest exit door:
![HoloGuide exit path plan view](img/hololens_adsk_3rd_floor_plan.png)
It is represented by a model curve:
![HoloGuide exit path 3D view](img/hololens_adsk_3rd_floor_3d.png)
The external command prompts you to select the exit path waypoints model curve, unless you pre-selected one before launching it, traverses it, determines waypoints and exports the results to JSON.
To report success, it displays the number of points exported:
![HoloGuide exit path report message](img/hololens_exit_path_result_msg.png)
Each waypoint can optionally also be represented by a marker in the view you generated it from:
![HoloGuide exit path markers](img/hololens_exit_path_result_markers.png)
The marker generation option can also be switched off.
If you do so, the command can run in read-only mode.
Right now, the transaction mode is set to manual, and a transaction is only started if the waypoint markers are requested.
Here is the entire external application implementation, which:
- Creates the ribbon tab and buttons.
- Populates the button icons from embedded resources.
- Ensures that the main command button always remains the current and default split button option.
```csharp
class App : IExternalApplication
{
public const string Caption = "Waypoints";
SplitButton split_button;
///
/// This external application
/// singleton class instance.
/// summary>
internal static App _app = null;
///
/// Provide access to this class instance.
/// summary>
public static App Instance
{
get { return _app; }
}
///
/// Return the full add-in assembly folder path.
/// summary>
public static string Path
{
get
{
return System.IO.Path.GetDirectoryName(
Assembly.GetExecutingAssembly().Location );
}
}
#region Create Ribbon Tab
///
/// Load a new icon bitmap from embedded resources.
/// For the BitmapImage, make sure you reference
/// WindowsBase and PresentationCore, and import
/// the System.Windows.Media.Imaging namespace.
/// summary>
BitmapImage NewBitmapImage(
System.Reflection.Assembly a,
string imageName )
{
Stream s = a.GetManifestResourceStream( imageName );
BitmapImage img = new BitmapImage();
img.BeginInit();
img.StreamSource = s;
img.EndInit();
return img;
}
void CreateRibbonTab(
UIControlledApplication a )
{
Assembly assembly = Assembly.GetExecutingAssembly();
string ass_path = assembly.Location;
string ass_name = assembly.GetName().Name;
// Create ribbon tab
string tab_name = Caption;
try
{
a.CreateRibbonTab( tab_name );
}
catch( Autodesk.Revit.Exceptions.ArgumentException )
{
// Assume error is due to tab already existing
}
PushButtonData pbCommand = new PushButtonData(
"Export", "Export", ass_path,
ass_name + ".Command" );
PushButtonData pbCommandOpt = new PushButtonData(
"Settings", "Settings", ass_path,
ass_name + ".CmdSettings" );
pbCommand.LargeImage = NewBitmapImage( assembly,
"ExportWaypointsJson.iCommand.png" );
pbCommandOpt.LargeImage = NewBitmapImage( assembly,
"ExportWaypointsJson.iCmdSettings.png" );
// Add button tips (when data, must be
// defined prior to adding button.)
pbCommand.ToolTip = "Export waypoints to JSON.";
pbCommand.LongDescription = "Export exit path "
+ "guide waypoints to JSON for Hololens "
+ "visualisation.";
// Add new ribbon panel.
string panel_name = Caption;
RibbonPanel thisNewRibbonPanel = a.CreateRibbonPanel(
tab_name, panel_name );
// add button to ribbon panel
SplitButtonData split_buttonData
= new SplitButtonData(
"splitFarClip", "FarClip" );
split_button = thisNewRibbonPanel.AddItem(
split_buttonData ) as SplitButton;
split_button.AddPushButton( pbCommand );
split_button.AddPushButton( pbCommandOpt );
}
///
/// Reset the top button to be the current one.
/// Alternative solution:
/// set RibbonItem.IsSynchronizedWithCurrentItem
/// to false after creating the SplitButton.
/// summary>
public void SetTopButtonCurrent()
{
IList sbList = split_button.GetItems();
split_button.CurrentButton = sbList[0];
}
#endregion // Create Ribbon Tab
public Result OnStartup( UIControlledApplication a )
{
_app = this;
CreateRibbonTab( a );
return Result.Succeeded;
}
public Result OnShutdown( UIControlledApplication a )
{
return Result.Succeeded;
}
}
```
As said, the ExportWaypointsJson add-in implements a couple of other noteworthy features that we may highlight in future discussions, including:
- Main external command
- Model curve selection
- Parametric curve traversal
- Conversion of XYZ points in feet to metres
- Serialisation of two-digit truncated XYZ coordinates
- JSON export of data using `JavaScriptSerializer` class
- Subsidiary external settings command
- Display modal Windows form
- Implement form validation using `ErrorProvider` class, `Validating` and `Validated` events
- Store add-in option settings in XML using the .NET `System.Configuration.ApplicationSettingsBase` class
- Store add-in option settings in JSON using custom solution and `JavaScriptSerializer` class
The entire add-in project is available from
the [ExportWaypointsJson GitHub repository](https://github.com/jeremytammik/ExportWaypointsJson),
and the version discussed here
is [release 2017.0.0.11](https://github.com/jeremytammik/ExportWaypointsJson/releases/tag/2017.0.0.11).
Have fun!
Thank you very much, Kean, for the inspiring idea and nice video!