---
post_number: "0817"
title: "Video Animated Ribbon Item Tooltip"
slug: "tooltip_video"
author: "Jeremy Tammik"
tags: ['csharp', 'family', 'references', 'revit-api', 'windows']
source_file: "0817_tooltip_video.htm"
original_url: "https://thebuildingcoder.typepad.com/blog/0817_tooltip_video.html"
---

﻿

### Video Animated Ribbon Item Tooltip

We already discussed a number of interesting things you can do with the Revit ribbon and user interface, many inspired by Rudolf Honke's exploration of uses of the .NET
[UI Automation](http://thebuildingcoder.typepad.com/blog/automation) functionality.

Here is a related contribution by Victor Chekalin, or Виктор Чекалин, fleshed out from his answer to Rudolf's
[question on animating a ribbon item tooltip](http://thebuildingcoder.typepad.com/blog/2012/06/retrieve-embedded-resource-image.html?cid=6a00e553e168978833016767281031970b#comment-6a00e553e168978833016767281031970b), discussing the retrieval of
[embedded resource images](http://thebuildingcoder.typepad.com/blog/2012/06/retrieve-embedded-resource-image.html):

It is possible to add your own video to a Revit ribbon button tool tip, and I've found a way to achieve this.

![Video running in a Revit ribbon item tool tip](img/VCButtonsWithVideoToolTip.png)

This can be achieved using the undocumented AdWindows.dll and UIFramework.dll assemblies located in the Revit program folder.

They provide many interesting classes, as Rudolf Honke already pointed out in his examples using
[UI Automation](http://thebuildingcoder.typepad.com/blog/automation).

There is a RibbonToolTip class in the AdWindows.dll assembly.
The class provides a property named ExpandedVideo.
The next step is to find a class which can contain ToolTip property.
This class is Autodesk.Windows.RibbonItem.

So, the idea is to find the correct RibbonItem in the ribbon control and set RibbonToolTip to it.

But here there is the trouble.
We can create a PushButton on a Panel using Application.CreatePanel().AddItem(RevitItem) to obtain a Autodesk.Revit.UI.RibbonItem.
But we need Autodesk.Windows.RibbonItem.

There are several ways.

The first one – use RevitControl class and iterate all tabs, panels and buttons and find button by name.

We can get RevitControl instance using static property RibbonControl UIFramework.RevitRibbonControl class.

RibbonControl contains exactly Autodesk.Windows.RibbonItem items not a Autodesk.Revit.UI.RibbonItem.
It's look like we need.

Here is the [code](http://pastebin.com/aXZ6u2nL) implementing this:
```csharp
  public Autodesk.Windows.RibbonItem GetRibbonItem(
    RibbonItem item )
  {
    RibbonControl ribbonControl
      = RevitRibbonControl.RibbonControl;

    foreach( var tab in ribbonControl.Tabs )
    {
      foreach( var panel in tab.Panels )
      {
        foreach( var ribbonItem
          in panel.Source.Items )
        {
          if( ribbonItem.AutomationName
            == item.Name )
          {
            return ribbonItem as
              Autodesk.Windows.RibbonItem;
          }
        }
      }
    }
    return null;
  }
```

But there is a second way.
The Autodesk.Revit.UI.RibbonItem class has an internal method getRibbonItem which returns Autodesk.Windows.RibbonItem. Interesting.
Try to use this method.
The reflection helps us.

Here is the [code](http://pastebin.com/52hVR61d):
```csharp
  public Autodesk.Windows.RibbonItem GetRibbonItem(
    RibbonItem item )
  {
    Type itemType = item.GetType();

    var mi = itemType.GetMethod( "getRibbonItem",
      BindingFlags.NonPublic | BindingFlags.Instance );

    var windowRibbonItem = mi.Invoke( item, null );

    return windowRibbonItem
      as Autodesk.Windows.RibbonItem;
  }
```

So, after we get a Autodesk.Windows.RibbonItem we can set RibbonToolTip to it.

For example [like this](http://pastebin.com/XtpKjLYq):
```csharp
  RibbonToolTip toolTip1 = new RibbonToolTip()
  {
    Title = "This is important button1",
    ExpandedContent =
@"Here you can see swf video from local file.
The file path is C:/Program Files
/Autodesk/Revit Structure 2013/Program/videos
/GUID-053F3A19-7EFF-48D5-A0FA-AF0C2CC4BD9D-low.swf",
    ExpandedVideo = new Uri( "C:/Program Files"
      + "/Autodesk/Revit Structure 2013/Program/videos"
      + "/GUID-053F3A19-7EFF-48D5-A0FA"
      + "-AF0C2CC4BD9D-low.swf" ),
    ExpandedImage = BitmapSourceConverter
      .ConvertFromImage( Resources.logoITC\_16 )
  };
```

PROFIT!!!

Also there is a third way to get a Autodesk.Windows.RibbonItem.
Create a tab, panel and button directly via RevitControl class.
But I didn't find how to set IExtenalCommand to a button.

RevitToolTip.ExpandedVideo supports SWF video.
Also I tried a GIF image.
The image was shown but didn't animate.
Also I tried to set Internet URL.
On my computer it doesn't work.
Maybe it doesn't work via proxy, or maybe access is denied on video in the internet.
Please try and tell me.

To wrap this up, here is the
[full source code](zip/VCButtonsWithVideoToolTip3.zip) of
my solution with test projects displaying video tool tip buttons for both Revit 2012 and 2013.

You may need to re-reference the Revit assemblies to compile them.

The first button loads a video from Revit Structure 201X\Program\videos.
If you have not installed RST you won't see the video on the first button.
The second button loads a video from the internet, and the third button loads one from the embedded resources.
So everyone should be able to see the video on the Button3.

Hope this information is useful for you.

Many thanks to Victor for exploring and explaining this exciting possibility!

#### Adjusting Beam Cutback

By the way, here is a yet another topic that I wanted to address for a long time, and now Mikako beat me to it: the FamilyInstance ExtensionUtility property allows you to control the joining, cutback and extension behaviour of beams, columns, girders etc.
Mikako shows how in her discussion and real-world sample on
[adjusting cutback programmatically](http://adndevblog.typepad.com/aec/2012/09/adjusting-cutback-programmatically-.html).

![Girders with cutback and miter joins](img/extensionutility.png)