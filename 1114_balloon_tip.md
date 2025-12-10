---
post_number: "1114"
title: "Using Balloon Tips in Revit"
slug: "balloon_tip"
author: "Jeremy Tammik"
tags: ['csharp', 'elements', 'references', 'revit-api', 'selection', 'windows']
source_file: "1114_balloon_tip.htm"
original_url: "https://thebuildingcoder.typepad.com/blog/1114_balloon_tip.html"
---

### Using Balloon Tips in Revit

Here is another tip from
Rudolf 'Revitalizer' Honke of [Mensch und Maschine acadGraph](http://www.acadgraph.de),
who provided most of the tips in the past making use of undocumented .NET and Autodesk API functionality provided by the
[UI Automation and AdWindows](http://thebuildingcoder.typepad.com/blog/automation) libraries, on using
[.NET Balloon Tips](#3) in a Revit add-in.

First, however, let me mention some dates for the international AU events.

#### International Autodesk University Event Locations and Dates

Some
[Autodesk University international](http://au.autodesk.com/las-vegas/international) and
AU Extension locations and dates for 2014 have been settled, so you can already mark your calendar:

- AU International

- August 29 – Tokyo, Japan
- October 2-3 – Moscow, Russia
- October 7-8 – Sao Paulo, Brazil
- October 23-24 – Frankfurt/Darmstadt, Germany
- October 27-28 – Beijing, China
- December 2-4 – Las Vegas, USA

- AU Extension (dates open)

- September – Johannesburg, South Africa
- October – Turkey
- December – Dubai, UAE

Locations and dates for Mexico City, Mumbai and Australia are also still open.

![AU 2014 locations](img/au_2014_locations.jpeg)

Back to the Revit API related stuff:

#### Using .NET Balloon Tips in a Revit Add-in

This tip comes straight from the Revit API discussion forum thread on
[using the status prompt in Revit](http://forums.autodesk.com/t5/Revit-API/Using-the-status-prompt-in-Revit/m-p/4829579#U4829579) initiated by
Remy van den Bor of [ICN Solutions](http://www.icnsolutions.net):

**Question:** Does anybody know if (and how) it is possible to use the status prompt for your own custom messages?

When you let your user select some elements, you can use the status prompt easily like so:

```csharp
  IList<Element> PickSelection
    = sel.PickElementsByRectangle(
      "Select by rectangle my highly appreciated user" );
```

But what if I want to use the status prompt to tell the user a certain action has ended.
What I am trying to avoid is an annoying message box to inform the user an action has ended.

Is there something like `StatusPrompt.Show("Action completed")`?

**Answer:** On one hand, we already pointed out how to
[set the status bar text](http://thebuildingcoder.typepad.com/blog/2011/02/status-bar-text.html) using
P/Invoke to call the standard Windows API methods provided in user32.dll.

On the other hand, if you just want to display a message in a non-blocking manner, you also could use a BalloonTip.

To do so, add a reference to AdWindows.dll to your VS project.

You can find it in the Revit program folder.

Then you could add code like this, for example as a static method in a "Utils" class:

```csharp
  public static void ShowBalloonTip(
    string category,
    string title,
    string text )
  {
    Autodesk.Internal.InfoCenter.ResultItem ri
      = new Autodesk.Internal.InfoCenter.ResultItem();

    ri.Category = category;
    ri.Title = title;
    ri.TooltipText = text;

    // Optional: provide a URL, e.g. a
    // website containing further information.

    ri.Uri = new System.Uri(
      "http://www.yourContextualHelp.de" );

    ri.IsFavorite = true;
    ri.IsNew = true;

    // You also could add a click event.

    ri.ResultClicked += new EventHandler<
      Autodesk.Internal.InfoCenter.ResultClickEventArgs>(
        ri\_ResultClicked );

    Autodesk.Windows.ComponentManager
      .InfoCenterPaletteManager.ShowBalloon( ri );
  }

  private static void ri\_ResultClicked(
    object sender,
    Autodesk.Internal.InfoCenter.ResultClickEventArgs e )
  {
    // do some stuff...
  }
```

Note that using functions provided by Revit's AdWindows.dll library is not officially supported by Autodesk
([disclaimer](http://thebuildingcoder.typepad.com/blog/about-the-author.html#4)).

But since AutoCAD also provides its own version of that library, which has been used officially by AutoCAD add-in developers for many years, e.g. to access the
[RibbonBar API in AutoCAD 2009](http://through-the-interface.typepad.com/through_the_interface/2008/04/the-new-ribbonb.html),
it seem to be a valid way.