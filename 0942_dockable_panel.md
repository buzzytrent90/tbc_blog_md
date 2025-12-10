---
post_number: "0942"
title: "A Simpler Dockable Panel Sample"
slug: "dockable_panel"
author: "Jeremy Tammik"
tags: ['csharp', 'elements', 'parameters', 'revit-api', 'transactions', 'views', 'windows']
source_file: "0942_dockable_panel.htm"
original_url: "https://thebuildingcoder.typepad.com/blog/0942_dockable_panel.html"
---

### A Simpler Dockable Panel Sample

Here is a very nice little sample on using the new Revit 2014 dockable panels by Håkan Wikemar of
[AEC](http://www.aec.se), Sweden.

In Håkan's words, it is close to the DockableDialogs SDK sample but easier to follow.

The DockableDialogs SDK sample demonstrates modeless dialog design, external events, and the new dockable dialogs UI API framework.

Håkan's DockableDialog sample described here focuses purely on the dockable panel functionality and nothing else.

One thing to note is that you need to register the panel before it can be displayed, and this needs to happen in a zero document state, i.e. with no project open.

In order for the external application ribbon panel to be accessible at all in zero document state, it needs to implement and register an availability class, and the associated command has to use manual transaction mode, as explained in
[enabling ribbon items in zero document state](http://thebuildingcoder.typepad.com/blog/2011/02/enable-ribbon-items-in-zero-document-state.html).

Once this is done, the panel appears like this:

![Register dockable panel in zero document state](img/dockable_panel_1.png)

If you try to register it twice, an exception is thrown, naturally, saying "Cannot register the same dockable pane ID more than once. Parameter name: id".

Once the panel has been registered and a project activated, the commands to show and hide the dockable panel are presented:

![Show dockable panel in project document](img/dockable_panel_2.png)

Those two commands can happily use read-only transaction mode, since they do not modify the database in any way.

The dockable panel is initially displayed sharing the same floating window as the project browser:

![Dockable panel in project browser floating window](img/dockable_panel_3.png)

It can be dragged off that pane to its own individual floating window:

![Dockable panel in separate floating window](img/dockable_panel_4.png)

Clicking 'Hide' removes it again.

Here is the full source code, which unfortunately sports rather long lines.
You can copy to a text editor or view the HTML source to see them in full, or simply download the zip file below:

```csharp
public class Ribbon : IExternalApplication
{
  public Result OnStartup(
    UIControlledApplication a )
  {
    a.CreateRibbonTab( "AEC LABS" );

    RibbonPanel AECPanelDebug
      = a.CreateRibbonPanel( "AEC LABS", "AEC LABS" );

    string path = Assembly.GetExecutingAssembly().Location;

    #region DockableWindow
    PushButtonData pushButtonRegisterDockableWindow = new PushButtonData( "RegisterDockableWindow", "RegisterDockableWindow", path, "DockableDialog.RegisterDockableWindow" );
    pushButtonRegisterDockableWindow.LargeImage = GetImage( Resources.green.GetHbitmap() );
    pushButtonRegisterDockableWindow.AvailabilityClassName = "DockableDialog.AvailabilityNoOpenDocument";
    PushButtonData pushButtonShowDockableWindow = new PushButtonData( "Show DockableWindow", "Show DockableWindow", path, "DockableDialog.ShowDockableWindow" );
    pushButtonShowDockableWindow.LargeImage = GetImage( Resources.red.GetHbitmap() );
    PushButtonData pushButtonHideDockableWindow = new PushButtonData( "Hide DockableWindow", "Hide DockableWindow", path, "DockableDialog.HideDockableWindow" );
    pushButtonHideDockableWindow.LargeImage = GetImage( Resources.orange.GetHbitmap() );
    //IList<RibbonItem> ribbonpushButtonDockableWindow = AECPanelDebug.AddStackedItems(pushButtonRegisterDockableWindow, pushButtonShowDockableWindow, pushButtonHideDockableWindow);

    RibbonItem ri1 = AECPanelDebug.AddItem( pushButtonRegisterDockableWindow );
    RibbonItem ri2 = AECPanelDebug.AddItem( pushButtonShowDockableWindow );
    RibbonItem ri3 = AECPanelDebug.AddItem( pushButtonHideDockableWindow );
    #endregion

    return Result.Succeeded;
  }

  public Result OnShutdown(
    UIControlledApplication a )
  {
    return Result.Succeeded;
  }

  private System.Windows.Media.Imaging.BitmapSource GetImage(
    IntPtr bm )
  {
    System.Windows.Media.Imaging.BitmapSource bmSource
      = System.Windows.Interop.Imaging.CreateBitmapSourceFromHBitmap(
        bm,
        IntPtr.Zero,
        System.Windows.Int32Rect.Empty,
        System.Windows.Media.Imaging.BitmapSizeOptions.FromEmptyOptions() );

    return bmSource;
  }
}

/// <summary>
/// You can only register a dockable dialog in "Zero doc state"
/// </summary>
public class AvailabilityNoOpenDocument
  : IExternalCommandAvailability
{
  public bool IsCommandAvailable(
    UIApplication a,
    CategorySet b )
  {
    if( a.ActiveUIDocument == null )
    {
      return true;
    }
    return false;
  }
}

/// <summary>
/// Register your dockable dialog
/// </summary>
[Transaction( TransactionMode.Manual )]
public class RegisterDockableWindow
  : IExternalCommand
{
  MainPage m\_MyDockableWindow = null;

  public Result Execute(
    ExternalCommandData commandData,
    ref string message,
    ElementSet elements )
  {
    DockablePaneProviderData data
      = new DockablePaneProviderData();

    MainPage MainDockableWindow = new MainPage();

    m\_MyDockableWindow = MainDockableWindow;

    //MainDockableWindow.SetupDockablePane(me);

    data.FrameworkElement = MainDockableWindow
      as System.Windows.FrameworkElement;

    data.InitialState = new DockablePaneState();

    data.InitialState.DockPosition
      = DockPosition.Tabbed;

    //DockablePaneId targetPane;
    //if (m\_targetGuid == Guid.Empty)
    //    targetPane = null;
    //else targetPane = new DockablePaneId(m\_targetGuid);
    //if (m\_position == DockPosition.Tabbed)

    data.InitialState.TabBehind = DockablePanes
      .BuiltInDockablePanes.ProjectBrowser;

    DockablePaneId dpid = new DockablePaneId(
      new Guid( "{D7C963CE-B7CA-426A-8D51-6E8254D21157}" ) );

    commandData.Application.RegisterDockablePane(
      dpid, "AEC Dockable Window", MainDockableWindow
      as IDockablePaneProvider );

    commandData.Application.ViewActivated
      += new EventHandler<ViewActivatedEventArgs>(
        Application\_ViewActivated );

    return Result.Succeeded;
  }

  void Application\_ViewActivated(
    object sender,
    ViewActivatedEventArgs e )
  {
    m\_MyDockableWindow.lblProjectName.Content
      = e.Document.ProjectInformation.Name;
  }
}

/// <summary>
/// Show dockable dialog
/// </summary>
[Transaction( TransactionMode.ReadOnly )]
public class ShowDockableWindow : IExternalCommand
{
  public Result Execute(
    ExternalCommandData commandData,
    ref string message,
    ElementSet elements )
  {
    DockablePaneId dpid = new DockablePaneId(
      new Guid( "{D7C963CE-B7CA-426A-8D51-6E8254D21157}" ) );

    DockablePane dp = commandData.Application
      .GetDockablePane( dpid );

    dp.Show();

    return Result.Succeeded;
  }
}

/// <summary>
/// Hide dockable dialog
/// </summary>
[Transaction( TransactionMode.ReadOnly )]
public class HideDockableWindow : IExternalCommand
{
  public Result Execute(
    ExternalCommandData commandData,
    ref string message,
    ElementSet elements )
  {
    DockablePaneId dpid = new DockablePaneId(
      new Guid( "{D7C963CE-B7CA-426A-8D51-6E8254D21157}" ) );

    DockablePane dp = commandData.Application
      .GetDockablePane( dpid );

    dp.Hide();
    return Result.Succeeded;
  }
}
```

In case you overlooked any of the following aspects, here is a list of the functionality demonstrated by the relatively few lines of code above:

- Implementation of an external application and several external commands in the same assembly
- Creation and population of a custom ribbon tab
- Creation and population of a custom ribbon panel with command buttons
- Loading of button images from .NET assembly resources
- Implementation of a command availability class
- Static enabling a command in zero document state
- Dynamic toggling of command availability based on document state

Here is [DockableDialog.zip](zip/DockableDialog.zip) containing
the complete source code, Visual Studio solution and add-in manifest of this external application.

Many thanks to Håkan for this nice sample!

**Addendum:** Guy Robinson of [redbolts.com](http://www.redbolts.com) adds to this and says:

I noticed the post on DockablePanes and Håkan's comment about finding the SDK sample confusing.
I did too.
I've been testing some UI ideas for a new app today and posted a basic version of something similar if interested.

I didn't want a register command on the ribbon and wanted more abstraction.
For instance, I didn't want the Page inheriting IDockablePaneProvider.

Therefore, registration is in the external application and uses some extension methods.

It is not perfect; it would require use of the Win32 API to capture the dock hiding/showing state if hidden from the dock itself.

Anyway, anyone interested can have a look at my
[DockableUITest](https://github.com/Redbolts/R14Tests/tree/master/DockableUITest) project on GitHub.

**Addendum 2:** Make sure you
[do not reuse an existing GUID](http://thebuildingcoder.typepad.com/blog/2017/10/do-not-reuse-an-existing-guid.html) if
you decide to copy and paste the code above.

Several people simply copied the code above into their own project without replacing the existing GUID.

Now they suffer the consequences.

Avoid similar pain yourself.