---
post_number: "0139"
title: "Revit 2010 Ribbon API"
slug: "ribbon_api"
author: "Jeremy Tammik"
tags: ['csharp', 'elements', 'levels', 'revit-api', 'views', 'walls', 'windows']
source_file: "0139_ribbon_api.htm"
original_url: "https://thebuildingcoder.typepad.com/blog/0139_ribbon_api.html"
---

### Revit 2010 Ribbon API

I am currently sitting in Stockholm, preparing a visit to a Revit developer here and enjoying the wonderful Swedish nature in May.
On Saturday I cut a couple of cubic meters of beautiful birch wood for heating with my friend Alexander in Huddinge, and on Sunday we went together with another class mate Lasse for a walk in the woods and around the lakes and marshes in Roslagen north of Stockholm, specifically to the three lakes Stora Harsjn, Lilla Harsjn and
[Trnan](http://sv.wikipedia.org/wiki/T%C3%A4rnan),
where we used to go camping on the weekends as school boys thirty-five years ago.
We saw lots of rare birds and plants and enjoyed the wonderful clear blue Scandinavian air and sky and stillness, so unlike anything you can see and feel in central Europe.
Now I am back to setting up my development system again on my temporary replacement computer, and getting ready for the training I am giving in the next few days.

The topic of accessing and modifying the Revit ribbon user interface was addressed by a
[comment from Jake](http://thebuildingcoder.typepad.com/blog/2008/12/driving-revit-from-outside.html?cid=6a00e553e1689788330115705e1423970b#comment-6a00e553e1689788330115705e1423970b)
and the
[ensuing discussion](http://thebuildingcoder.typepad.com/blog/2008/12/driving-revit-from-outside.html?cid=6a00e553e168978833011570604384970b#comment-6a00e553e168978833011570604384970b).
Here, we will just discuss the basic Revit ribbon API functionality.

Revit 2010 has a completely new look and feel due to the ribbon based user interface, familiar to many from the Microsoft Office and AutoCAD 2009 and 2010 products.
The ribbon interface simplifies the usage for new users and reacts to the design context to provide immediate access to suitable functionality based on the current modelling requirements.
Unlike AutoCAD based products, this change is not optional.
There is no way to turn this off and switch back to the old UI.

The Revit API has been extended in order to enable Revit external applications to make use of the ribbon.
Here is an overview of the Revit 2010 ribbon API, which was also discussed in the recent
[Revit programming webcast](http://thebuildingcoder.typepad.com/blog/2009/05/revit-api-introduction-webcast.html),
and a few related questions that cropped up.

The ribbon contains a number of tabs, which are automatically activated depending on the current design context.
This is helpful both for new users to discover relevant commands and for experienced users to have convenient access to them.
Each tab can contain any number of panels, which in turn can host various user interface widgets.

All external application and external command related user interface items are located in the Add-Ins tab.
This tab is displayed as soon as an external application or command is added to the Revit.ini file.
External commands are listed under the 'External Tools' button.
External applications can create their own ribbon panels and populate them with push buttons and pull-down buttons in either a normal, single or stacked fashion with the option of two or three layers.

Existing external application will need to migrate their old toolbar or menu based user interface to the ribbon.
The ribbon API is the only GUI customization API in Revit 2010.
It is quite straight-forward to use.
Although the ribbon is based on the Windows Presentation Foundation WPF, no WPF knowledge is needed to make use of the Revit ribbon API.

The API enables you to add your custom ribbon panels and populate them with various types of buttons.
The buttons can be push or pull-down ones, in a single row or in a stacked layout with two or three rows.
Unfortunately, you cannot currently create your own separate ribbon tabs.

In addition to the programming aspect, the Revit SDK also includes documentation with guidelines for designing your custom ribbon and icons.
This is to ensure that your application integrates well with the standard Revit UI.
The documents are located in the SDK folder and also explain the terminology used in the ribbon API:

- Ribbon design guidelines.pdf- Autodesk Icon Guidelines.pdf

The new ribbon related classes in Revit are:

- RibbonPanel, a ribbon panel in the Add-Ins tab containing ribbon items.- RibbonItem, a button, push or pull-down.- PushButton and PushButtonData to manage push button information.- PulldownButton and PulldownButtonData to manage pull-down button information

The usage of the ribbon API is demonstrated by the Ribbon SDK sample application, which is an external application creating its own ribbon panel and showing how to create the various types of ribbon items programmatically.
It adds the following elements to the Add-Ins tab:

- Two add-in application ribbon panels.- A push button 'Create Wall'.- A three-layered stacked button.- A pull-down button 'Move Walls'.- Push buttons used to display add-in panel info.
        - Separators.- Tooltips.

This is what the result looks like in the user interface:

![Ribbon SDK sample](img/ribbon_sdk_sample.jpg)

The figure shows the two ribbon panels called 'Ribbon Sample' and 'Add-in Panel Info'.
The first panel, on the left, has one push button called 'Create Wall', and a three-layered stacked button next to it showing 'Delete Last Wall' at the top.
Notice that the third stacked button, 'Move Walls', is a pull-down button.
You can also add tooltips to and separators between buttons.

Here is the relevant part of the code, the method CreateRibbons, which is called from the external application OnStartup method:

```csharp
private void CreateRibbons(
  ControlledApplication application )
{
  // application path
  string pathRibbon
    = GetType().Assembly.Location;

  string pathAddIn
    = pathRibbon.Replace("Ribbon.dll",
      "AddInCommands.dll");

  string bitmapFolder
    = Path.GetDirectoryName(pathRibbon);

  # region create RibbonPanel with PushButtons, 'create wall' and 'delete walls'

  string firstPanelName = "PushButtons";

  RibbonPanel ribbonPanelPushButtons
    = application.CreateRibbonPanel(
      firstPanelName );

  PushButton pushButtonCreateWall
    = ribbonPanelPushButtons.AddPushButton(
    "createWall", "Create Wall", pathAddIn,
    "Revit.SDK.Samples.Ribbon.CS.CreateWall" );

  pushButtonCreateWall.ToolTip
    = "Create a wall on level 1.";

  pushButtonCreateWall.LargeImage
    = new BitmapImage(
      new Uri( Path.Combine( bitmapFolder,
        "CreateWall.bmp"), UriKind.Absolute ) );

  ribbonPanelPushButtons.AddSeparator();

  PushButton pushButtonDeleteWalls
    = ribbonPanelPushButtons.AddPushButton(
    "deleteWalls", "Delete Walls", pathAddIn,
    "Revit.SDK.Samples.Ribbon.CS.DeleteWalls" );

  pushButtonDeleteWalls.ToolTip
    = "Delete all the walls in active document.";

  pushButtonDeleteWalls.LargeImage
    = new BitmapImage(
      new Uri( Path.Combine( bitmapFolder,
        "DeleteWalls.bmp"), UriKind.Absolute ) );

  # endregion

  # region create RibbonPanel with a pulldownButton which contains two MenuItems, 'move walls'

  string secondPanelName = "PulldownButtons";

  RibbonPanel ribbonPanelPulldowns
    = application.CreateRibbonPanel(
      secondPanelName );

  PulldownButton pulldownMoveWall
    = ribbonPanelPulldowns.AddPulldownButton(
      "moveWall", "Move Walls");

  pulldownMoveWall.ToolTip
    = "Move all the walls in X or Y direction";

  pulldownMoveWall.LargeImage
    = new BitmapImage(
      new Uri( Path.Combine(
        bitmapFolder, "MoveWall.bmp" ),
        UriKind.Absolute ) );

  RibbonMenuItem moveX
    = pulldownMoveWall.AddItem(
    "X Direction", pathAddIn,
    "Revit.SDK.Samples.Ribbon.CS.XMoveWalls" );

  moveX.ToolTip
    = "move all walls 10 feet in X direction.";

  RibbonMenuItem moveY
    = pulldownMoveWall.AddItem(
    "Y Direction", pathAddIn,
    "Revit.SDK.Samples.Ribbon.CS.YMoveWalls" );

  moveY.ToolTip
    = "move all walls 10 feet in Y direction.";

  # endregion

  # region create RibbonPanel with "Add-in Panel Info" button

  string thirdPanelName
    = "Add-in Panel Info";

  RibbonPanel ribbonPanelOutputlInfo
    = application.CreateRibbonPanel(
      thirdPanelName );

  PushButton pushButtonOutputlInfo
    = ribbonPanelOutputlInfo.AddPushButton(
    "outputlInfo",
    "Output Add-in Panel Info",
    pathAddIn,
    "Revit.SDK.Samples.Ribbon.CS.OutputPanelInfo" );

  pushButtonOutputlInfo.ToolTip
    = "Output info of all the Add-In RibbonPanels.";

  pushButtonOutputlInfo.LargeImage
    = new BitmapImage(
      new Uri(Path.Combine(
        bitmapFolder,
        "OutputAddInPanelInfo.bmp" ),
        UriKind.Absolute ) );

  # endregion
}
```

Here is another question regarding the ribbon user interface and its customisation facilities:

#### User Interface Customisation

**Question:**
Is there a way to customize the ribbon toolbars in Revit 2010?
There is no right-click menu item for 'Customize' like in AutoCAD Architecture.

**Answer:**
The only customization possibility I see in the user interface is provided by the 'User Interface' button in the View tab.
This provides control of the visibility of the navigation bar, project browser, status bar, recent files list and project browser settings:

![Ribbon user interface button](img/ribbon_user_interface.jpg)

As stated above, the Revit API allows applications to add UI elements to the ribbon, but only in the Add-Ins tab, and only certain types of ribbon items, as illustrated by the Ribbon SDK sample.

Many thanks to my colleagues Adam Nagy and Saikat Bhattacharya for providing the basis for this summary.