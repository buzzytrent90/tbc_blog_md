---
ai_optimized: true
author: Jeremy Tammik
complexity_score: 5.3
content_type: code_example
optimization_date: '2025-12-11T11:44:14.745186'
original_url: https://thebuildingcoder.typepad.com/blog/0857_toggle_button.html
post_number: 0857
reading_time_minutes: 5
series: general
slug: toggle_button
source_file: 0857_toggle_button.htm
tags:
- csharp
- elements
- python
- references
- revit-api
- transactions
- walls
- windows
title: Roll Your Own Toggle Button
word_count: 1073
---

### Roll Your Own Toggle Button

Today I took my first stab at creating, compiling and debugging a new Revit add-in on the Mac.

I cannot run Revit or Visual Studio natively on the Mac, of course, so those have to remain in Windows, hosted by the Parallels environment on the Mac.

I had to fiddle around quite a bit with the Parallels synchronisation, which seems to be breaking more things than it fixes, for me at least.
I managed in the end in spite of that, though.

The opportunity to implement a new sample add-in was provided by the following question:

**Question:** Is there a way to create a ribbon button that can be toggled, e.g. a sort of on/off switch like the Modify > Properties button?

**Answer:** Yes, sure.

You will have to roll your own, though.

The Revit API just provides radio buttons.

Here is a description of radio buttons from Saikat Bhattacharya's upcoming Autodesk University class
[CP3272](https://www.autodeskuniversity2012.com/connect/sessionDetail.ww?SESSION_ID=3272) Snapshot of the Autodesk Revit User Interface API:

#### Radio Buttons

![Radio buttons](img/radio_button.png)

A radio button group helps toggle between options by letting users select only one ribbon item at a time.
After adding a RadioButtonGroup to a panel, use the AddItem and AddItems methods to add radio buttons, which are basically normal PushButtons, to the group.
The RadioButtonGroup.Current property can be used to access the currently selected button.
Just like SplitButton, tooltips do not apply to radio button groups; instead the tooltip for each toggle button is displayed.
```csharp
  RadioButtonGroupData radioData =
    new RadioButtonGroupData( "radioGroup" );

  RadioButtonGroup radioButtonGroup =
    panel.AddItem( radioData ) as RadioButtonGroup;

  ToggleButtonData tb1 = new ToggleButtonData(
    "OverrideCommand1",
    "Override Cmd: Off",
    assemblyPath + "\\" + assemblyName,
    "SnapshotRevitUI\_CS.OverrideOff" );

  ToggleButtonData tb2 = new ToggleButtonData(
    "OverrideCommand2",
    "Override Cmd: On",
    assemblyPath + "\\" + assemblyName,
    "SnapshotRevitUI\_CS.OverrideOn" );

  tb2.ToolTip = "Override the Wall Creation command";

  tb2.LargeImage = new BitmapImage(
    new Uri( imageFolder + "globe\_32.png" ) );

  radioButtonGroup.AddItem( tb1 );
  radioButtonGroup.AddItem( tb2 );
```

However, this is still not quite what you want, is it?

You would really like one single button that changes its state each time it is clicked.

#### A Real Toggle Button

Well, as said, this can easily be achieved by implementing your own custom button.

Note that the RibbonButton Image, LargeImage and ItemText properties are all read-write.

Therefore, there is nothing to stop you from changing the button appearance any way you like at any time you like.
For instance, if you change these properties appropriately each time the button is clicked, it will appear like a toggle button such as you describe.

I implemented a minimal sample illustrating this using just the ItemText property:

It would look more impressive if I switched the image as well, of course.
I leave that as a simple exercise to the reader.

The basic framework to achieve this consists of a very small external application and a single-line external command.

The external application sets up the ribbon panel and button, saves a reference to the latter, and provides a public method Toggle that uses the button reference to toggle its text:

```csharp
class App : IExternalApplication
{
  static string \_path = typeof( App ).Assembly.Location;

  RibbonItem \_button;

  /// <summary>
  /// Singleton external application class instance.
  /// </summary>
  internal static App \_app = null;

  /// <summary>
  /// Provide access to singleton class instance.
  /// </summary>
  public static App Instance
  {
    get { return \_app; }
  }

  public Result OnStartup(
    UIControlledApplication a )
  {
    \_app = this;

    RibbonPanel panel = a.CreateRibbonPanel(
      "ToggleButton" );

    PushButtonData data = new PushButtonData(
      "Toggle", "On", \_path,
      "ToggleButton.Command" );

    data.AvailabilityClassName
      = "ToggleButton.Availability";

    \_button = panel.AddItem( data );

    return Result.Succeeded;
  }

  public Result OnShutdown( UIControlledApplication a )
  {
    return Result.Succeeded;
  }

  public void Toggle()
  {
    string s = \_button.ItemText;

    \_button.ItemText = s.Equals( "On" ) ? "Off" : "On";
  }
}
```

The button images could be toggled in a similar fashion.

The Toggle method is public, and used by the command to toggle the button state each time it is launched:

```csharp
  public Result Execute(
    ExternalCommandData commandData,
    ref string message,
    ElementSet elements )
  {
    App.Instance.Toggle();

    return Result.Succeeded;
  }
```

Probably one of the simplest command implementations you have ever seen, isn't it?

This command was initially read-only, and that makes sense, but I had to change it to manual transaction mode for reasons explained below.

#### Zero Document State Support

I made the whole sample a little bit more complicated in order to enable its use in a
[zero document context](http://thebuildingcoder.typepad.com/blog/2011/02/enable-ribbon-items-in-zero-document-state.html) as
well as when a project is opened.

I implemented a trivial
[availability](http://thebuildingcoder.typepad.com/blog/2010/04/revitaddinutility.html#availability) class
that I use to set the button availability.
In this case, I want it to be available always:

```python
  public class Availability
    : IExternalCommandAvailability
  {
    public bool IsCommandAvailable(
      UIApplication a,
      CategorySet b )
    {
      return true;
    }
  }
```

Enabling the command in zero document context also requires changing its transaction mode from read-only to manual, because Revit only allows manual transaction mode commands in zero document state.
Violation of this rule causes the following error:

![Zero document context requires manual transaction mode](img/zero_doc_requires_manual.png)

As you can see from the recording above, all is well now, and this is a viable solution.

Here is
[ToggleButton.zip](zip/ToggleButton.zip) containing
the complete source code, Visual Studio solution and add-in manifest for this samplpe add-in.

While I've been having fun creating this sample, let's look at some examples of cool stuff other people are doing as well.

#### IDAT Revit Structure Slab Add-In

The German company [IDAT](http://www.idat.de/home-en-GB) presents
a neat example of using the Revit API implement an add-in for the design, fabrication and construction logistics of precast hollow core slabs.
This
[video](http://www.youtube.com/1kslQ9P1160) shows
their precast module for hollowcore slabs inside Revit Structure:

#### Ab in den Hafen by Allerdings

Completely unrelated to Revit and its API, but closely related to me, my son Christopher aka
[Allerdings](http://soundcloud.com/allerdings) has
been composing and publishing music on SoundCloud for some time now.

Here is his most recent oevre,
[Ab in den Hafen](http://soundcloud.com/allerdings/ab-in-den-hafen):

Enjoy.

#### AUGI Wish List

Another cool and important thing, albeit more administrative, is the
[AUGI wish list](http://www.augi.com/wishlist) providing
a unified independent voice back to Autodesk on how to improve its products.

Please [vote now](http://www.augi.com/wishlist/vote-wishes) for your most important wish.