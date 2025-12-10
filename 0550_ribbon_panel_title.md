---
post_number: "0550"
title: "Ribbon Panel Title Conflict"
slug: "ribbon_panel_title"
author: "Jeremy Tammik"
tags: ['csharp', 'elements', 'revit-api']
source_file: "0550_ribbon_panel_title.htm"
original_url: "https://thebuildingcoder.typepad.com/blog/0550_ribbon_panel_title.html"
---

### Ribbon Panel Title Conflict

Here is another exploration into the inner workings of the Revit ribbon by Rudolf Honke of
[acadGraph CADstudio GmbH](http://www.acadgraph.de).
He says:

If you try to add a new Ribbon panel whose title is already used by another Add-In panel, it will throw an exception.
You therefore have to ensure that no other third party panel uses the same text as you do.

The Revit API method GetRibbonPanels will not find panels that are not yet loaded, and there is no fixed order in which plug-in will be loaded.
Therefore, even if a plug-in is installed after yours, it might still be loaded before and could have 'occupied' your 'own' panel title, which would cause your plug-in to fail if the panel title conflict is not handled appropriately, for instance by renaming it on the fly.

Here is an example in which I loaded RevitLookup twice and renamed the displayed text of the second instance:

![Ribbon panel naming conflict resolution](img/rh_panel_title_conflict_1.png)

This may be a real concern, especially if your panel title is something generic such as 'Tools'.

To make sure, you need to iterate through the already loaded panels and adapt your title if necessary before creating it.

This affects the 'Title' property; I'm not sure whether the 'Name' needs to be unique, too.

Here is some sample code to illustrate this;
in your external application, you can define a global variable:
```csharp
  private UIControlledApplication m\_revit = null;
```

It can be initialised when starting the application:
```csharp
  public Result OnStartup( UIControlledApplication a )
  {
    m\_revit = a;

    /\*
    (do something useful here)
    \*/

    return Result.Succeeded;
  }
```

Then we can implement the following method to check whether a panel title has already been taken:
```csharp
  private bool IsPanelTitleUsed(
    string panelTitle )
  {
    List<RibbonPanel> loadedPluginPanels
      = m\_revit.GetRibbonPanels();

    foreach( RibbonPanel p in loadedPluginPanels )
    {
      if( p.Title.Equals( panelTitle ) )
      {
        return true;
      }
    }
    return false;
  }
```

If we decide to change our panel title, for example by appending an integer counter suffix, we obviously have to continue checking whether it is available until we find a title that has not been taken yet.

The cause of this exception when trying to create RibbonPanels of the same name is visible in the StackTrace:

- Autodesk.Revit.UI.UIApplication.verifyPanelNameExclusive(Tab tab, String newPanelName)- Autodesk.Revit.UI.UIApplication.CreateRibbonPanel(Tab tab, String panelName)- Autodesk.Revit.UI.UIApplication.CreateRibbonPanel(String panelName)- Autodesk.Revit.UI.UIControlledApplication.CreateRibbonPanel(String panelName)

After I called the CreateRibbonPanel method, the Title property is not initialized:
![Ribbon panel name and null title](img/rh_panel_title_conflict_2.png)

The title value stays null, even if you think that you see it in the UI.

So **the name is your title** except you assign a different one:
```csharp
rvtRibbonPanel.Title = "Dummy";
```
![Ribbon panel name and title differ](img/rh_panel_title_conflict_3.png)

![Ribbon panel name and title differ](img/rh_panel_title_conflict_4.png)

Having different names, titles **can** consist of the same string:

![Ribbon panels with identical names](img/rh_panel_title_conflict_5.png)

Using UISpy, by the way, shows that the Revit AutomationElements can be retrieved by name (see AutomationId), not by visible text.
Here are the properties of the left panel in the image above:

![Ribbon panel properties in UISpy](img/rh_panel_title_conflict_6.png)

Here are the right-hand panel properties:

![Ribbon panel properties in UISpy](img/rh_panel_title_conflict_7.png)

For both of these, the Name property is an empty string, while the AutomationId property still distinguishes them as "CustomCtrl\_%ADD\_INS\_TAB%RevitLookup" versus "CustomCtrl\_%ADD\_INS\_TAB%RevitLookup 2".

Many thanks to Rudolf for this important warning and interesting inside-the-ribbon information.