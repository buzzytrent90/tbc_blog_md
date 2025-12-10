---
post_number: "1049"
title: "RoomEditorApp Architecture and External Application"
slug: "roomeditorapp"
author: "Jeremy Tammik"
tags: ['csharp', 'family', 'python', 'references', 'revit-api', 'rooms', 'views', 'windows']
source_file: "1049_roomeditorapp.htm"
original_url: "https://thebuildingcoder.typepad.com/blog/1049_roomeditorapp.html"
---

### RoomEditorApp Architecture and External Application

A couple of days ago, I wrote about
[creating the RoomEditorApp GitHub repository](http://thebuildingcoder.typepad.com/blog/2013/10/roomeditorapp-for-revit-2014-on-github.html) for
the Revit add-in part of my cloud-based real-time round-trip 2D Revit model editing application on any mobile device and promised to discuss its implementation details anon.

Well, the obvious place to start that is by providing an
[architectural overview](#2),
followed by a look at its
[external application implementation](#3).

I'll also mention an issue with
[unresponsive Idling](#4) that
I am currently experiencing and hope to resolve, and where to
[download](#5) the
current state of things.

Before getting to the nitty-gritty, by the way, have you already heard that
[three out of the top ten mobile apps for architects are developed by Autodesk](http://www.archdaily.com/441499/top-10-apps-for-architects)?

#### RoomEditorApp Architectural Overview

The room editor add-in consists of the following modules:

- App.cs
- CmdAbout.cs
- CmdSubscribe.cs
- CmdUpdate.cs
- CmdUpload.cs
- CmdUploadAll.cs
- ContiguousCurveSorter.cs
- DbModel.cs
- DbUpdater.cs
- DbUpload.cs
- GeoSnoop.cs
- JtBoundingBox2dInt.cs
- JtBoundingBoxXyz.cs
- JtPlacement2dInt.cs
- JtWindowHandle.cs
- Point2dInt.cs
- Point2dIntLoop.cs
- RoomEditorDb.cs
- Util.cs

Starting at the end of the list, Util.cs contains a bunch of utilities to handle little details such as:

- Unit conversion
- Formatting
- Messages
- Browsing for a directory
- Flipping SVG Y coordinates

Moving back to the beginning of the list, the five modules with a Cmd prefix are external command implementations driven by the custom panel created by the external application defined in App.cs.

They fulfil the following tasks:

- CmdAbout – display an about message.
- CmdUpload – upload selected rooms and their furniture and equipment to the cloud database.
- CmdUploadAll – upload all rooms in the model and their furniture and equipment to the cloud database.
- CmdUpdate – refresh all furniture and equipment family instance placements from the cloud database.
- CmdSubscribe – toggle back and forth between real-time subscription to updates from the cloud database.

A noteworthy aspect of subscription command is that it switches the button text dynamically to reflect its state.

They are represented by corresponding icons displayed by the ribbon panel user interface:

![Room editor add-in user interface](img/roomedit_2014_ext_app_icons.png)

The remaining modules can be grouped into the following main areas:

- Boundary loops – determine the room and family instance boundary loop polygons, driven by CmdUpload:

- ContiguousCurveSorter.cs
- JtBoundingBox2dInt.cs
- JtBoundingBoxXyz.cs
- JtPlacement2dInt.cs
- Point2dInt.cs
- Point2dIntLoop.cs

- GeoSnoop – temporary graphical display of the boundary loops, triggered by CmdUpload when it has done its job:

- GeoSnoop.cs
- JtWindowHandle.cs

- Database model – manage the information uploaded to and retrieved from the cloud database:

- DbModel.cs
- DbUpdater.cs
- DbUpload.cs
- RoomEditorDb.cs

I already discussed all the aspects of the
[boundary loop determination, GeoSnoop graphical debugging display and database representation](http://thebuildingcoder.typepad.com/blog/2013/04/room-and-furniture-loops-using-symbols.html) in
pretty good detail back in April:

- [Database structure](http://thebuildingcoder.typepad.com/blog/2013/04/room-and-furniture-loops-using-symbols.html#2)
- [Database upload](http://thebuildingcoder.typepad.com/blog/2013/04/room-and-furniture-loops-using-symbols.html#3)
- [Integer based 2D placement](http://thebuildingcoder.typepad.com/blog/2013/04/room-and-furniture-loops-using-symbols.html#4)
- [Populating symbols and instances](http://thebuildingcoder.typepad.com/blog/2013/04/room-and-furniture-loops-using-symbols.html#5)
- [Retrieving the boundary loops](http://thebuildingcoder.typepad.com/blog/2013/04/room-and-furniture-loops-using-symbols.html#6)
- [GeoSnoop loop display](http://thebuildingcoder.typepad.com/blog/2013/04/room-and-furniture-loops-using-symbols.html#7)

Actually, that was the last time I discussed anything at all related to this add-in until migrating it to Revit 2014 last week, so all the items I listed as
[next steps](http://thebuildingcoder.typepad.com/blog/2013/04/room-and-furniture-loops-using-symbols.html#10) back
then and that have now been implemented remain to be discussed.

Let's begin with the external application implementation:

#### RoomEditorApp External Application Implementation

The external application fulfils the following main tasks:

1. [Handle retrieval of the embedded icon resources](3.1).
2. [Create and populate the custom ribbon panel](3.2).
3. [Toggle subscription command text and manage the Idling event handler](3.3).
4. [Main entry points](3.4).

Let's look at each of these in more detail.

#### Handle Retrieval of Embedded Icon Resources

All the icons are saved into the Revit add-in assembly as embedded resources, living in an own subfolder named Icon:

![Embedded icon resources](img/roomedit_2014_embedded_icon_resource.png)

This is obviously very handy, as there is no need to copy the icon files around separately.

Here are the methods used to extract the bitmap image information at runtime:

```python
  /// <summary>
  /// Executing assembly namespace
  /// </summary>
  static string \_namespace = typeof( App ).Namespace;

  /// <summary>
  /// Return path to embedded resource icon
  /// </summary>
  static string IconResourcePath(
    string name,
    string size )
  {
    return \_namespace
      + "." + "Icon" // folder name
      + "." + name + size // icon name
      + ".png"; // filename extension
  }

  /// <summary>
  /// Load a new icon bitmap from embedded resources.
  /// For the BitmapImage, make sure you reference
  /// WindowsBase and PresentationCore, and import
  /// the System.Windows.Media.Imaging namespace.
  /// </summary>
  static BitmapImage GetBitmapImage(
    Assembly a,
    string path )
  {
    string[] names = a.GetManifestResourceNames();

    Stream s = a.GetManifestResourceStream( path );

    Debug.Assert( null != s,
      "expected valid icon resource" );

    BitmapImage img = new BitmapImage();

    img.BeginInit();
    img.StreamSource = s;
    img.EndInit();

    return img;
  }
```

#### Create and Populate Custom Ribbon Panel

I define the various command button data such as its text, implementation class, icon and tooltip in arrays of strings to enable defining the ribbon items in a simple loop.

With the bitmap handling functionality in place, the entire custom ribbon panel creation is handled in one fell swoop by the following AddRibbonPanel method:

```csharp
  /// <summary>
  /// Caption
  /// </summary>
  public const string Caption = "Room Editor";

  /// <summary>
  /// Command name prefix
  /// </summary>
  const string \_cmd\_prefix = "Cmd";

  /// <summary>
  /// Currently executing assembly path
  /// </summary>
  static string \_path = typeof( App )
    .Assembly.Location;

  /// <summary>
  /// Keep track of our ribbon buttons to toggle
  /// them on and off later and change their text.
  /// </summary>
  static RibbonItem[] \_buttons;

  /// <summary>
  /// Create a custom ribbon panel and populate
  /// it with our commands, saving the resulting
  /// ribbon items for later access.
  /// </summary>
  static void AddRibbonPanel(
    UIControlledApplication a )
  {
    string[] tooltip = new string[] {
      "Upload selected rooms to cloud.",
      "Upload all rooms to cloud.",
      "Update furniture from the last cloud edit.",
      "Subscribe to or unsubscribe from updates.",
      "About " + Caption + ": ..."
    };

    string[] text = new string[] {
      "Upload Selected",
      "Upload All",
      "Update Furniture",
      "Subscribe",
      "About..."
    };

    string[] classNameStem = new string[] {
      "Upload",
      "UploadAll",
      "Update",
      "Subscribe",
      "About"
    };

    string[] iconName = new string[] {
      "1Up",
      "2Up",
      "1Down",
      "ZigZagRed",
      "Question"
    };

    int n = classNameStem.Length;

    Debug.Assert( text.Length == n,
      "expected equal number of text and class name entries" );

    \_buttons = new RibbonItem[n];

    RibbonPanel panel
      = a.CreateRibbonPanel( Caption );

    SplitButtonData splitBtnData
      = new SplitButtonData( Caption, Caption );

    SplitButton splitBtn = panel.AddItem(
      splitBtnData ) as SplitButton;

    Assembly asm = typeof( App ).Assembly;

    for( int i = 0; i < n; ++i )
    {
      PushButtonData d = new PushButtonData(
        classNameStem[i], text[i], \_path,
        \_namespace + "." + \_cmd\_prefix
        + classNameStem[i] );

      d.ToolTip = tooltip[i];

      d.Image = GetBitmapImage( asm,
        IconResourcePath( iconName[i], "16" ) );

      d.LargeImage = GetBitmapImage( asm,
        IconResourcePath( iconName[i], "32" ) );

      d.ToolTipImage = GetBitmapImage( asm,
        IconResourcePath( iconName[i], "" ) );

      \_buttons[i] = splitBtn.AddPushButton( d );
    }
  }
```

#### Toggle the Subscription Command Text and Idling Event Handler Management

With all of the commands in place, the subscription command text toggling and Idling event handler management becomes almost trivial.

I presented the principles to
[implement your own toggle button](http://thebuildingcoder.typepad.com/blog/2012/11/roll-your-own-toggle-button.html#3)
a year ago, and we simply make use of that here.

The button icon could be toggled as well, if we like.

The Idling event handler is defined in the subscription command implementation, where it belongs.

However, best practice as demonstrated by the ModelessDialog ModelessForm\_IdlingEvent Revit SDK sample retains the final control and the subscription to the event in the external application.

In order for the command to define the handler and toggle the subscription on and off, the external application provides a method named ToggleSubscription taking the event handler implementation as an argument.

It subscribes to or unsubscribes from the event as requested, and also toggles the text displayed by the corresponding command button:

I define a property name 'Subscribed' to determine the current subscription status, and toggle it on and off by calling the ToggleSubscription method:

```csharp
  /// <summary>
  /// Our one and only Revit-provided
  /// UIControlledApplication instance.
  /// </summary>
  static UIControlledApplication \_uiapp;

  /// <summary>
  /// Switch between subscribe
  /// and unsubscribe commands.
  /// </summary>
  const string \_subscribe = "Subscribe";
  const string \_unsubscribe = "Unsubscribe";
  /// <summary>
  /// Are we currently subscribed
  /// to automatic cloud updates?
  /// </summary>
  public static bool Subscribed
  {
    get
    {
      return \_buttons[3].ItemText.Equals(
        \_unsubscribe );
    }
  }

  /// <summary>
  /// Toggle on and off subscription to
  /// automatic cloud updates.
  /// </summary>
  public static void ToggleSubscription(
    EventHandler<IdlingEventArgs> handler )
  {
    if( Subscribed )
    {
      \_uiapp.Idling -= handler;
      \_buttons[3].ItemText = \_subscribe;
    }
    else
    {
      \_uiapp.Idling += handler;
      \_buttons[3].ItemText = \_unsubscribe;
    }
  }
```

#### Main Entry Points OnStartup and OnShutdown

All that remains to do for the external application is initialise the \_uiapp variable and add the custom ribbon panel on start-up, and remove the Idling event handler if it is still active on shutdown:

```csharp
  public Result OnStartup(
    UIControlledApplication a )
  {
    \_uiapp = a;

    AddRibbonPanel( a );

    return Result.Succeeded;
  }

  public Result OnShutdown(
    UIControlledApplication a )
  {
    if( Subscribed )
    {
      \_uiapp.Idling
        -= new EventHandler<IdlingEventArgs>(
          ( sender, ea ) => { } );
    }
    return Result.Succeeded;
  }
```

This is probably my most complex external application to date.

I hope you appreciate its simplicity in spite of all the requirements it fulfils, and that this presentation helps you keep your add-ins as simple as possible as well.

#### Unresponsive Idling

Before closing, let me mention that my tests of this application so far on Revit 2014 and Windows 7 show a decreased responsiveness of the Idling event compared to Revit 2013 and Windows XP.

In Revit 2013, I was even calling the SetRaiseWithoutDelay method to get as many Idling calls as possible with no problem.

Regardless of that setting, the system is currently much less responsive in Revit 2014.

The task manager shows Revit.exe hogging almost 100% percent of the CPU as soon as I subscribe to the Idling event.

Debugging this, I also note that my attempts to unsubscribe from the Idling event handler have no effect; surprisingly, the Idling event handler still gets called anyway.
Something seems to have changed in the interaction between Revit 2014 and the Idling event.

I added some debugging variables to count the number of Idling calls received, print a message now and then, and skip the database query for most of them.
I also removed the exception wrapping the database query.
The problem is somewhat alleviated but not yet solved.

I don't know yet whether I have an issue with my virtual machine in Parallels in Mac, or my cloud database is acting differently on Windows 7 than it did on Windows XP, or some other suboptimal setting is causing this.
Hopefully I can get it resolved soon, though.

Any advice on this is much appreciated!

#### Download

This application lives in the
[RoomEditorApp GitHub repository](https://github.com/jeremytammik/RoomEditorApp) and
the version discussed above is
[release 2014.0.0.15](https://github.com/jeremytammik/RoomEditorApp/releases/tag/2014.0.0.15).