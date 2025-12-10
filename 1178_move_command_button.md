---
post_number: "1178"
title: "Moving an External Command Button"
slug: "move_command_button"
author: "Jeremy Tammik"
tags: ['csharp', 'elements', 'parameters', 'revit-api', 'rooms', 'transactions', 'windows']
source_file: "1178_move_command_button.htm"
original_url: "https://thebuildingcoder.typepad.com/blog/1178_move_command_button.html"
---

### Moving an External Command Button

I recently presented Frode Tørresdal's
[unofficial custom ribbon button](http://thebuildingcoder.typepad.com/blog/2014/06/room-editor-live-and-unofficial-custom-ribbon-button.html#4) implementation
showing how you can add a custom button that Revit knows nothing about to the ribbon and hook it up with the Idling event to perform some action requiring access to the Revit API.

It would be much simpler, safer and more effective if we could implement a normal official Revit external command instead and create a button anywhere we want in the Revit ribbon to drive that.

Well, happily, we can, almost, as Scott Wilson points out in his
[comment](http://thebuildingcoder.typepad.com/blog/2014/06/room-editor-live-and-unofficial-custom-ribbon-button.html#comment-6a00e553e16897883301a511d0c4b8970c) on that post.

#### The Living

Before getting to that, though, let me mention something of interest to architects, techies and all AEC visionaries alike, the acquisition of the very forward-thinking architectural design studio
[The Living](http://www.thelivingnewyork.com),
headed by David Benjamin.
It is the latest addition to the Autodesk research network, the first of its kind Autodesk Studio, in a move which Benjamin says 'will enable The Living to do more of what we are already doing and super-charge it.'

Here is the official
[announcement](http://inthefold.autodesk.com/in_the_fold/2014/06/the-intersection-of-design-culture-and-technology-exploring-new-frontiers-in-computing-and-the-built-environment.html) and
articles about it by
[Architect](http://www.architectmagazine.com/architects/autodesk-acquires-david-benjamins-design-studio-the-living_o.aspx),
[ArchDaily](http://www.archdaily.com/522532/what-autodesk-s-acquisition-of-the-living-means-for-architecture) and
[Architectural Record](http://archrecord.construction.com/news/2014/07/140701-David-Benjamin-The-Living-Autodesk.asp).

#### Scott

I thought I'd let you know of something I found a little while ago while participating in the Revit API forum thread on
[adding a button to an existing ribbon panel](http://forums.autodesk.com/t5/Revit-API/Add-button-to-existing-ribbon-panel/td-p/4981498).

Basically, there is a way of adding your own custom panels and buttons to the system tabs and have them behave as standard command buttons with full API access, no need for Idling callbacks and other stuff.

Here is my brief explanation from the thread: "If you add your command button using standard API methods to its own ribbon panel on either your own tab or the built in add-ins tab, you can then find the panel using the above methods and insert it into any of the system tabs using the Autodesk.Windows.RibbonTab.Panels.Add method. The button will then behave in the same way as it would on the original tab and the API doesn't seem to notice – to the extent of my brief testing, anyway. You can then remove the panel from the add-ins tab if desired and the panel will remain happily in its new home."

I dug up my test code, cleaned it up into a little demo project and posted it to Dropbox.com in
[RibbonMoveExample.zip](https://dl.dropboxusercontent.com/u/74965361/RevitAPICode/RibbonMoveExample.zip).

Please see the comments included in the source code below for the rest of Scott's detailed explanation.

#### Jeremy

Jeremy adds: wow, very cool!

What a beautiful workaround, working around the official Revit API limitations and yet retaining all the official functionality goodness – much better than the completely unofficial hack using the .NET UI Automation library.

I compiled and tested this.

It creates a new panel containing a new button and moves them to the Revit ribbon Manage tab.
Here is the result:

![Moving an external command ribbon button](img/move_external_command_ribbon_button.png)

It supports all the standard external command functionality, e.g. an external command availability class.

#### External Command for Testing

The external command used for testing is defined as follows:

```csharp
using System;
using Autodesk.Revit.Attributes;
using Autodesk.Revit.DB;
using Autodesk.Revit.UI;

namespace RibbonMoveExample
{
  public class ExampleCommand\_Availability
    : IExternalCommandAvailability
  {
    public bool IsCommandAvailable(
      UIApplication uiApp,
      CategorySet categorySet )
    {
      Autodesk.Revit.UI.UIDocument uiDoc
        = uiApp.ActiveUIDocument;

      if( uiDoc != null )
      {
        return true;
      }
      else
      {
        return false;
      }
    }
  }

  [Transaction( TransactionMode.Manual )]
  public class ExampleCommand : IExternalCommand
  {
    public Autodesk.Revit.UI.Result Execute(
      ExternalCommandData commandData,
      ref String message,
      ElementSet elements )
    {
      try
      {
        Document dbDoc = commandData.Application
          .ActiveUIDocument.Document;

        TaskDialog.Show( "ExampleCommand",
          "Current document title: "
          + dbDoc.ProjectInformation.Name
          + "\nLocation: " + dbDoc.PathName );
      }
      catch( Exception e )
      {
        message = e.Message;
        return Result.Failed;
      }
      return Result.Succeeded;
    }
  }
}
```

#### External Application Implementation

As said, the external application sports Scott's interesting explanations, including a detailed dissection of the ribbon panel and ribbon item Id property values required for identification:

```csharp
using System;
using System.Collections.Generic;
using System.Reflection;
using System.Windows.Forms;

using Autodesk.Revit.ApplicationServices;
using Autodesk.Revit.UI;

using adWin = Autodesk.Windows;

namespace RibbonMoveExample
{
  /// <summary>
  /// Demonstrates the use of Autodesk.Windows.RibbonControl
  /// to move API created ribbon buttons and / or panels from
  /// their original location to any other tab / panel including
  /// system created ones.
  /// I'm pretty sure that this technique would also work in
  /// reverse to allow grabbing system panels / buttons and adding
  /// them to API created Tabs and even other system tabs, meaning
  /// that some drastic customisation of Revit's ribbon menu is
  /// possible.
  /// Revit CUI addin anybody? :)
  ///
  /// Created by Scott Wilson, released into the public domain as
  /// demonstration of concept only. Use at own risk.
  /// </summary>
  public class RibbonMoveExampleApp : IExternalApplication
  {

    // System tab and panel ids to use as targets for the
    // demo to find other valid ids for system tabs and
    // panels, simply dump out the id of each item visited
    // in the foreach loops below

    String SystemTabId = "Manage";
    String SystemPanelId = "settings\_shr";

    // Example names of API created tabs, panels and buttons
    // To try this demo on ribbon items already created by
    // another loaded addin, enter the appropriate details
    // here and comment out everything in the OnStartup method
    // except for the ApplicationInitialized registration

    String ApiTabName = "New Tab";
    String ApiPanelName = "New Panel";
    String ApiButtonName = "NewButton";
    String ApiButtonText = "New\nButton";

    public Result OnStartup(
      UIControlledApplication uiConApp )
    {

      // Add our own tab, panel and command button
      // to the ribbon using the API

      uiConApp.CreateRibbonTab( ApiTabName );

      RibbonPanel apiRibbonPanel = uiConApp.CreateRibbonPanel(
        ApiTabName, ApiPanelName );

      PushButtonData buttonDataTest = new PushButtonData(
        ApiButtonName, ApiButtonText,
        Assembly.GetExecutingAssembly().Location,
        "RibbonMoveExample.ExampleCommand" );

      buttonDataTest.AvailabilityClassName
        = "RibbonMoveExample.ExampleCommand\_Availability";

      apiRibbonPanel.AddItem( buttonDataTest );

      // Subscribe to the "ApplicationInitialized" event
      // then continue from there once it is fired.
      // This is to ensure that the ribbon is fully
      // populated before we mess with it.

      uiConApp.ControlledApplication.ApplicationInitialized
        += OnApplicationInitialized;

      return Result.Succeeded;
    }

    void OnApplicationInitialized(
      object sender,
      Autodesk.Revit.DB.Events.ApplicationInitializedEventArgs e )
    {
      // Find a system ribbon tab and panel to house
      // our API items
      // Also find our API tab, panel and button within
      // the Autodesk.Windows.RibbonControl context

      adWin.RibbonControl adWinRibbon
        = adWin.ComponentManager.Ribbon;

      adWin.RibbonTab adWinSysTab = null;
      adWin.RibbonPanel adWinSysPanel = null;

      adWin.RibbonTab adWinApiTab = null;
      adWin.RibbonPanel adWinApiPanel = null;
      adWin.RibbonItem adWinApiItem = null;

      foreach( adWin.RibbonTab ribbonTab
        in adWinRibbon.Tabs )
      {
        // Look for the specified system tab

        if( ribbonTab.Id == SystemTabId )
        {
          adWinSysTab = ribbonTab;

          foreach( adWin.RibbonPanel ribbonPanel
            in ribbonTab.Panels )
          {
            // Look for the specified panel
            // within the system tab

            if( ribbonPanel.Source.Id == SystemPanelId )
            {
              adWinSysPanel = ribbonPanel;
            }
          }
        }
        else
        {
          // Look for our API tab

          if( ribbonTab.Id == ApiTabName )
          {
            adWinApiTab = ribbonTab;

            foreach( adWin.RibbonPanel ribbonPanel
              in ribbonTab.Panels )
            {
              // Look for our API panel.

              // The Source.Id property of an API created
              // ribbon panel has the following format:
              // CustomCtrl\_%[TabName]%[PanelName]
              // Where PanelName correlates with the string
              // entered as the name of the panel at creation
              // The Source.AutomationName property can also
              // be used as it is also a direct correlation
              // of the panel name, but without all the cruft
              // Be sure to include any new line characters
              // (\n) used for the panel name at creation as
              // they still form part of the Id & AutomationName

              //if(ribbonPanel.Source.AutomationName
              //  == ApiPanelName) // Alternative method

              if( ribbonPanel.Source.Id ==
                "CustomCtrl\_%" + ApiTabName + "%" + ApiPanelName )
              {
                adWinApiPanel = ribbonPanel;

                foreach( adWin.RibbonItem ribbonItem
                  in ribbonPanel.Source.Items )
                {
                  // Look for our command button

                  // The Id property of an API created ribbon
                  // item has the following format:
                  // CustomCtrl\_%CustomCtrl\_%[TabName]%[PanelName]%[ItemName]
                  // Where ItemName correlates with the string
                  // entered as the first parameter (name)
                  // of the PushButtonData() constructor
                  // While AutomationName correlates with
                  // the string entered as the second
                  // parameter (text) of the PushButtonData()
                  // constructor
                  // Be sure to include any new line
                  // characters (\n) used for the button
                  // name and text at creation as they
                  // still form part of the ItemName
                  // & AutomationName

                  //if(ribbonItem.AutomationName
                  //  == ApiButtonText) // alternative method

                  if( ribbonItem.Id
                    == "CustomCtrl\_%CustomCtrl\_%"
                    + ApiTabName + "%" + ApiPanelName
                    + "%" + ApiButtonName )
                  {
                    adWinApiItem = ribbonItem;
                  }
                }
              }
            }
          }
        }
      }

      // Make sure we got everything we need

      if( adWinSysTab != null
        && adWinSysPanel != null
        && adWinApiTab != null
        && adWinApiPanel != null
        && adWinApiItem != null )
      {
        // First we'll add the whole panel including
        // the button to the system tab

        adWinSysTab.Panels.Add( adWinApiPanel );

        // now lets also add the button itself
        // to a system panel

        adWinSysPanel.Source.Items.Add( adWinApiItem );

        // Remove panel from original API tab
        // It can also be left there if needed,
        // there doesn't seem to be any problems with
        // duplicate panels / buttons on seperate tabs
        // / panels respectively

        adWinApiTab.Panels.Remove( adWinApiPanel );

        // Remove our original API tab from the ribbon

        adWinRibbon.Tabs.Remove( adWinApiTab );
      }

      // A new panel should now be added to the
      // specified system tab. Its command buttons
      // will behave as they normally would, including
      // API access and ExternalCommandAvailability tests.
      // There will also be a second copy of the command
      // button from the panel added to the specified
      // system panel.

    }

    public Result OnShutdown( UIControlledApplication a )
    {
      return Result.Succeeded;
    }
  }
}
```

Many thanks to Scott for this very creative workaround!

#### Use at Your Own Risk

By the way, please note our standard
[disclaimer](http://thebuildingcoder.typepad.com/blog/about-the-author.html#4).

The Revit API development team have good reasons for limiting the places that an add-in can place its command buttons.

You work around those limitations at your own risk.

This functionality may very well be disabled at any time with no warning.