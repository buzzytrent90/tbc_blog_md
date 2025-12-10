---
post_number: "0894"
title: "Add a Button to Existing Ribbon Panel"
slug: "exist_ribbon_panel"
author: "Jeremy Tammik"
tags: ['csharp', 'elements', 'references', 'revit-api', 'transactions', 'windows']
source_file: "0894_exist_ribbon_panel.htm"
original_url: "https://thebuildingcoder.typepad.com/blog/0894_exist_ribbon_panel.html"
---

### Add a Button to Existing Ribbon Panel

Here is an interesting user interface contribution from Eduardo Teixeira of Autodesk:

**Question:** I am building a Revit add-in using the Revit API.

I would like to add a button for it to an existing out-of-the-box Revit ribbon panel using the Revit API in C#.

I could not find any documentation that shows how to do that.

Is it possible at all?

**Answer:** Yes, this can be done.
I am not certain whether it is officially supported, though.
You will have to explore on your own a bit.

The official API is limited.
You can however also use functionality from the .NET
[UI Automation](http://thebuildingcoder.typepad.com/blog/automation) library
and from the AdWindows.dll .NET assembly included with Revit, similar to the one provided with AutoCAD.

Here are some previous discussion exploring related issues that might help:

- [Pimp my ribbon](http://thebuildingcoder.typepad.com/blog/2011/02/pimp-my-autocad-or-revit-ribbon.html)
- [Roll a toggle button](http://thebuildingcoder.typepad.com/blog/2012/11/roll-your-own-toggle-button.html)
- [Enable button in zero document state](http://thebuildingcoder.typepad.com/blog/2011/02/enable-ribbon-items-in-zero-document-state.html)
- [Add non-command button to the ribbon](http://thebuildingcoder.typepad.com/blog/2010/03/adding-noncommands-to-the-revit-ribbon.html)

**Response:** Thanks for the tip.
The article that really helped me was the first one you list.
I was able to add a button to the existing out-of-the-box Energy Analysis panel under the Analyze tab.

There are just two simple steps required achieve the desired result:

To add a custom button to the Analyze tab in the Revit Energy Analysis ribbon panel:

1. [Add a reference](#2) to the AdWindows.dll .NET assembly provided with Revit.
2. Add the two snippets of source code listed below to the external application implementation:
   1. In OnStartup, [create and add the button](#3).
   2. [Handle the UIElementActivated event](#4).

#### Adding the References

Add a reference to the AdWindows.dll .NET assembly provided with Revit.

Don't forget to set the copy local flag to false on that.

This will also require references to the .NET WindowsBase and PresentationFramework assemblies, which in turn require PresentationCore and System.Xaml.

#### Creating and Adding the Button

In OnStartup, localise the existing tab and panel, instantiate a new ribbon button, add it to the tab, and subscribe to the UIElementActivated event.

Here is a simplified version of Eduardo's code set up to run on Jeremy's system and open the browser to display The Building Coder home page:

```csharp
public Result OnStartup( UIControlledApplication a )
{
  adWin.RibbonControl ribbon
    = adWin.ComponentManager.Ribbon;

  foreach( adWin.RibbonTab tab in ribbon.Tabs )
  {
    if( tab.Id == "Analyze" )
    {
      foreach( adWin.RibbonPanel panel
        in tab.Panels )
      {
        if( panel.Source.Id == "cea\_shr" )
        {
          adWin.RibbonButton button
            = new adWin.RibbonButton();

          button.Name = "TbcButtonName";
          //button.Image = image;
          //button.LargeImage = image;
          button.Id = "ID\_TBC\_BUTTON";
          button.AllowInStatusBar = true;
          button.AllowInToolBar = true;
          button.GroupLocation = Autodesk.Private
            .Windows.RibbonItemGroupLocation.Middle;
          button.IsEnabled = true;
          button.IsToolTipEnabled = true;
          button.IsVisible = true;
          button.ShowImage = false; //  true;
          button.ShowText = true;
          button.ShowToolTipOnDisabled = true;
          button.Text = "The Building Coder";
          button.ToolTip = "Open The Building "
            + "Coder blog on the Revit API";
          button.MinHeight = 0;
          button.MinWidth = 0;
          button.Size = adWin.RibbonItemSize.Large;
          button.ResizeStyle = adWin
            .RibbonItemResizeStyles.HideText;
          button.IsCheckable = true;
          button.Orientation = System.Windows
            .Controls.Orientation.Vertical;
          button.KeyTip = "TBC";

          adWin.ComponentManager.UIElementActivated
            += new EventHandler<
              adWin.UIElementActivatedEventArgs>(
              ComponentManager\_UIElementActivated );

          panel.Source.Items.Add( button );

          return Result.Succeeded;
        }
      }
    }
  }
  return Result.Succeeded;
}
```

As you can see, I am triggering the reaction to the button click by subscribing to the UIElementActivated event.
Not sure if this is the correct way of doing it, but it works.

#### Handling the Event

In the event handler, check that the custom button was actually clicked and act on that, e.g. opening the browser at a specified URL.

```csharp
void ComponentManager\_UIElementActivated(
  object sender,
  adWin.UIElementActivatedEventArgs e )
{
  if( e != null
    && e.Item != null
    && e.Item.Id != null
    && e.Item.Id == "ID\_TBC\_BUTTON" )
  {
    // Perform the button action

    // Local file

    string path = System.Reflection.Assembly
      .GetExecutingAssembly().Location;

    path = Path.Combine(
      Path.GetDirectoryName( path ),
      "test.html" );

    // Internet URL

    path = "http://thebuildingcoder.typepad.com";

    Process.Start( path );
  }
}
```

Note that this requires no interaction with the Revit database, so there is no need to worry about a valid API context or transactions or any of that stuff.

Many thanks to Eduardo for sharing this!

#### Sample Run

Jeremy adds: for your convenience, here is
[AddButton.zip](zip/AddButton.zip) containing
the source code, Visual Studio solution and add-in manifest for my version of this external application.

On my system, however, the button appears in the expected location but is disabled, so no action is triggered:

![Custom button in existing panel](img/AddButton.png "Custom button in existing panel")

I am not sure why that happens.
Maybe Eduardo or somebody else can help resolve this.