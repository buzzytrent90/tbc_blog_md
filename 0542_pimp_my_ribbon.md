---
post_number: "0542"
title: "Pimp my AutoCAD or Revit Ribbon"
slug: "pimp_my_ribbon"
author: "Jeremy Tammik"
tags: ['csharp', 'elements', 'family', 'python', 'references', 'revit-api', 'windows']
source_file: "0542_pimp_my_ribbon.htm"
original_url: "https://thebuildingcoder.typepad.com/blog/0542_pimp_my_ribbon.html"
---

### Pimp my AutoCAD or Revit Ribbon

Here is another fun possibility to access and manipulate the Revit ribbon explored by Rudolf Honke of
[acadGraph CADstudio GmbH](http://www.acadgraph.de).
He says:

I've played around and found a possibility to modify the appearance of the AutoCAD and Revit ribbon bars.

This is the Revit ribbon bar in its usual colour scheme:

![Revit ribbon colour scheme](img/rh_pimp_ribbon_1.png)

Now let's change the panel background image and panel headers:

![Modified background image and panel headers](img/rh_pimp_ribbon_2.png)

What about using a gradient fill and changing the tab header font:

![Gradient fill and different tab header font](img/rh_pimp_ribbon_3.png)

The style is persistent, even if you tear off the panels:

![Free-floating panels](img/rh_pimp_ribbon_4a.png)
![Free-floating panels](img/rh_pimp_ribbon_4b.png)

You can use different styles in different panels:

![Different styles in different panels](img/rh_pimp_ribbon_5.png)

So, how to get there?

First, add new references to your VS project:

- AdWindows.dll- UIFramework.dll

You can find them in the same folder as RevitAPI.dll.

Possibly some other references need to be included as well.
Here is a complete list of the references in my project, although some of them may not be needed for UI customizing purposes:

![UI customisation references](img/rh_pimp_ribbon_refs.png)

Now add this to your ExternalApplication class:
```csharp
using System.Reflection; // for getting the assembly path
using System.Windows.Media; // for the graphics
using System.Windows.Media.Imaging;

// use an alias because Autodesk.Revit.UI
// uses classes which have same names:

using adWin = Autodesk.Windows;
```

Then insert this into your OnStartup method:
```python
public Result OnStartup( UIControlledApplication a )
{
  try
  {
    adWin.RibbonControl ribbon
      = adWin.ComponentManager.Ribbon;

    ImageSource imgbg = new BitmapImage(
      new Uri( Path.Combine(
        Path.GetDirectoryName(
          Assembly.GetExecutingAssembly().Location ),
        "yourBackGroundPicture.jpg" ),
        UriKind.Relative ) );

    // define an image brush

    ImageBrush picBrush = new ImageBrush();
    picBrush.ImageSource = imgbg;
    picBrush.AlignmentX = AlignmentX.Left;
    picBrush.AlignmentY = AlignmentY.Top;
    picBrush.Stretch = Stretch.None;
    picBrush.TileMode = TileMode.FlipXY;

    // define a linear brush from top to bottom

    LinearGradientBrush gradientBrush
      = new LinearGradientBrush();

    gradientBrush.StartPoint
      = new System.Windows.Point( 0, 0 );

    gradientBrush.EndPoint
      = new System.Windows.Point( 0, 1 );

    gradientBrush.GradientStops.Add(
      new GradientStop( Colors.White, 0.0 ) );

    gradientBrush.GradientStops.Add(
      new GradientStop( Colors.Blue, 1 ) );

    // change the tab header font

    ribbon.FontFamily = new System.Windows.Media
      .FontFamily( "Bauhaus 93" );

    ribbon.FontSize = 10;

    // iterate through the tabs and their panels

    foreach( adWin.RibbonTab tab in ribbon.Tabs )
    {
      foreach( adWin.RibbonPanel panel in tab.Panels )
      {
        panel.CustomPanelTitleBarBackground
          = gradientBrush;

        panel.CustomPanelBackground
          = picBrush; // use your picture

        //panel.CustomPanelBackground
        //  = gradientBrush; // use your gradient fill
      }
    }
    adWin.ComponentManager.UIElementActivated += new
      EventHandler<adWin.UIElementActivatedEventArgs>(
        ComponentManager\_UIElementActivated );
  }
  catch( Exception ex )
  {
    winform.MessageBox.Show(
      ex.StackTrace + "\r\n" + ex.InnerException,
      "Error", winform.MessageBoxButtons.OK );

    return Result.Failed;
  }
  return Result.Succeeded;
}
```

You can access the items inside the panels like this:
```csharp
  foreach( adWin.RibbonItem item
    in panel.Source.Items )
  {
    . . .
  }
```

Note that each tab, panel, or item provides event handlers, so you can subscribe to their events like this:
```csharp
  tab.Activated
    += new EventHandler(tab\_Activated);

  panel.PropertyChanged
    += new PropertyChangedEventHandler(
      panel\_PropertyChanged);

  item.PropertyChanged
    +=new PropertyChangedEventHandler(
      item\_PropertyChanged);
```

There are many undiscovered properties in the Revit Ribbon classes, so feel free to explore them on your own (and tell us your discoveries).

By the way, AutoCAD 2011 also includes the AdWindows.dll assembly, but there is no UIFramework.dll in the AutoCAD program folder (using AutoCAD MEP 2011).

Using the AdWindows.dll assembly, we can access the ribbon control like this:
```csharp
using Autodesk.Windows;
RibbonControl ribbon = ComponentManager.Ribbon;
```

Since both AutoCAD and Revit include the AdWindows.dll assembly, there are actually two ways to access to the ribbon control in Revit;
either using AdWindows.dll or UIFramework.dll, respectively:
```csharp
  // like in AutoCAD:

  RibbonControl ribbon = ComponentManager.Ribbon;

  // also possible, but requires
  // UIFramework.dll reference,
  // so we could better use the first method:

  UIFramework.RevitRibbonControl ribbon
    = UIFramework.RevitRibbonControl.RibbonControl;
```

Additionally, one can also use Reflection to retrieve the Ribbon bar from a RibbonPanel p created in the 'normal' way via its p.RibbonControl method:
```csharp
private Autodesk.Windows.RibbonPanel
  GetPanelFromPanel(
    Autodesk.Revit.UI.RibbonPanel panel )
{
  FieldInfo fi
    = panel.GetType().GetField(
      "m\_RibbonPanel",
      BindingFlags.Instance
      | BindingFlags.NonPublic );

  if( null != fi )
  {
    Autodesk.Windows.RibbonPanel p
      = fi.GetValue( panel )
      as Autodesk.Windows.RibbonPanel;

    return p;
  }
  return null;
}
```

So, one can customize the AutoCAD ribbon in a similar way:

![Pimp the AutoCAD ribbon](img/rh_pimp_ribbon_acad.png)

This can be performed by the following little piece of code:
```csharp
public void Initialize()
{
  RibbonControl ribbon = ComponentManager.Ribbon;

  ImageSource imgbg = new BitmapImage(
    new Uri( Path.Combine( Path.GetDirectoryName(
      Assembly.GetExecutingAssembly().Location ),
      "yourBackGroundPicture.jpg" ),
      UriKind.Relative ) );

  // define an image brush

  ImageBrush picBrush = new ImageBrush();
  picBrush.ImageSource = imgbg;
  picBrush.AlignmentX = AlignmentX.Left;
  picBrush.AlignmentY = AlignmentY.Top;
  picBrush.Stretch = Stretch.None;
  picBrush.TileMode = TileMode.FlipXY;

  // define a linear brush from top to bottom

  LinearGradientBrush myLinearGradientBrush
    = new LinearGradientBrush();

  myLinearGradientBrush.StartPoint
    = new System.Windows.Point( 0, 0 );

  myLinearGradientBrush.EndPoint
    = new System.Windows.Point( 0, 1 );

  myLinearGradientBrush.GradientStops.Add(
    new GradientStop( Colors.White, 0.0 ) );

  myLinearGradientBrush.GradientStops.Add(
    new GradientStop( Colors.Blue, 1 ) );

  // change the tab header font

  ribbon.FontFamily = new FontFamily( "Bauhaus 93" );
  ribbon.FontSize = 10;

  // now iterate through the tabs and their panels

  foreach( RibbonTab tab in ribbon.Tabs )
  {
    foreach( RibbonPanel panel in tab.Panels )
    {
      panel.CustomPanelTitleBarBackground
        = myLinearGradientBrush;

      panel.CustomPanelBackground = picBrush;

      //panel.CustomPanelBackground
      //  = myLinearGradientBrush;
    }
  }
}
```

As you see, only the access to the ribbon control differs, but most of the code is identical to the Revit version.

Since we can access the ribbon bar identically in both AutoCAD and Revit using AdWindows.dll, one could easily build a common module for ribbon operations in general.

Also, an event listener can be achieved for both CAD systems:
```csharp
ComponentManager.UIElementActivated
  += new EventHandler<Autodesk.Windows
    .UIElementActivatedEventArgs>(
      ComponentManager\_UIElementActivated );

void ComponentManager\_UIElementActivated(
  object sender,
  Autodesk.Windows.UIElementActivatedEventArgs e )
{
  // e.UiElement.PersistId says which item has been pressed
}
```

By the way, this is my first AutoCAD / Revit hybrid contribution.
This might be interesting for AutoCAD developers too, I think.
But after all, it's just playing around...

To wrap this up, here is a fully functional example of a Revit external application to play around with which exercises some of this functionality,
[PimpMyRevit2.zip](zip/PimpMyRevit2.zip).
It includes the Visual Studio solution, an add-in manifest, and the external application implementation source code, which looks like this in its entirety:
```csharp
using System;
using System.IO;
using System.Reflection;
using winform = System.Windows.Forms;
using System.Windows.Media;
using System.Windows.Media.Imaging;
using Autodesk.Revit.Attributes;
using Autodesk.Revit.UI;
using adWin = Autodesk.Windows;

namespace PimpMyRevit
{
  [Regeneration( RegenerationOption.Manual )]
  public class UIApp : IExternalApplication
  {
    public Result OnStartup( UIControlledApplication a )
    {
      try
      {
        adWin.RibbonControl ribbon
          = adWin.ComponentManager.Ribbon;

        ImageSource imgbg = new BitmapImage(
          new Uri( Path.Combine(
            Path.GetDirectoryName(
              Assembly.GetExecutingAssembly().Location ),
            "yourBackGroundPicture.jpg" ),
            UriKind.Relative ) );

        // define an image brush

        ImageBrush picBrush = new ImageBrush();
        picBrush.ImageSource = imgbg;
        picBrush.AlignmentX = AlignmentX.Left;
        picBrush.AlignmentY = AlignmentY.Top;
        picBrush.Stretch = Stretch.None;
        picBrush.TileMode = TileMode.FlipXY;

        // define a linear brush from top to bottom

        LinearGradientBrush gradientBrush
          = new LinearGradientBrush();

        gradientBrush.StartPoint
          = new System.Windows.Point( 0, 0 );

        gradientBrush.EndPoint
          = new System.Windows.Point( 0, 1 );

        gradientBrush.GradientStops.Add(
          new GradientStop( Colors.White, 0.0 ) );

        gradientBrush.GradientStops.Add(
          new GradientStop( Colors.Blue, 1 ) );

        // change the tab header font

        ribbon.FontFamily = new System.Windows.Media
          .FontFamily( "Bauhaus 93" );

        ribbon.FontSize = 10;

        // iterate through the tabs and their panels

        foreach( adWin.RibbonTab tab in ribbon.Tabs )
        {
          foreach( adWin.RibbonPanel panel in tab.Panels )
          {
            panel.CustomPanelTitleBarBackground
              = gradientBrush;

            panel.CustomPanelBackground
              = picBrush; // use your picture

            //panel.CustomPanelBackground
            //  = gradientBrush; // use your gradient fill
          }
        }
        adWin.ComponentManager.UIElementActivated += new
          EventHandler<adWin.UIElementActivatedEventArgs>(
            ComponentManager\_UIElementActivated );
      }
      catch( Exception ex )
      {
        winform.MessageBox.Show(
          ex.StackTrace + "\r\n" + ex.InnerException,
          "Error", winform.MessageBoxButtons.OK );

        return Result.Failed;
      }
      return Result.Succeeded;
    }

    public Result OnShutdown( UIControlledApplication a )
    {
      adWin.ComponentManager.UIElementActivated
        -= ComponentManager\_UIElementActivated;

      return Result.Succeeded;
    }

    void ComponentManager\_UIElementActivated(
      object sender,
      Autodesk.Windows.UIElementActivatedEventArgs e )
    {
      // e.UiElement.PersistId says which item has been pressed
    }
  }
}
```

Have fun with trying different pictures and colours.
By the way, one could write an add-in which stores different 'styles' in a xml file...
Revit for boys, Revit for girls, blue theme or pink theme ;-)

One notice: you have to re-invoke the function after all add-ins have been loaded completely, because it obviously does not affect the panels that are created after the call.
One could put the ribbon control into a global variable that can be used by the OnDocumentOpened event or something like that after all panels have been populated.

Here is my Revit started up after loading this external application, with some of the add-ins affected and others left in their original pristine state:

![Partially pimp Revit add-ins ribbon tab](img/rh_pimp_ribbon_jt.png)

#### Ribbon Bar Transformations

Some additions to the pimping theme.

You can apply all kinds of transformations to the ribbon bar, including translation, scaling, and rotation. You can turn it upside down, or throw it into space somewhere...

After the transformation, the ribbon buttons remain just as functional as before, e.g. tool tips are displayed and the buttons can be selected as usual.

Because the Ribbon bar is a WPF element, it can be transformed this way:
```csharp
using adWin = Autodesk.Windows;
  adWin.RibbonControl ribbon
    = adWin.ComponentManager.Ribbon;

  TransformGroup group = new TransformGroup();
  //group.Children.Add(new RotateTransform(-2));
  group.Children.Add( new TranslateTransform(
    ribbon.ActualWidth \* 0.25, 160 ) );

  group.Children.Add( new ScaleTransform(
    scale, scale ) );

  group.Children.Add( new SkewTransform(
    15, -3 ) );

  ribbon.RenderTransform = group;
```

Feel free to test different combinations.
Below are some examples.
To begin with, here is The Ribbon bar in its default state:
![Revit ribbon bar in default state](img/rh_pimp_ribbon_xform_1.png)
```csharp
  adWin.ComponentManager.Ribbon.RenderTransform
    = new System.Windows.Media.ScaleTransform(
      0.5, 0.75 );
```
![Revit ribbon bar with scaling transform](img/rh_pimp_ribbon_xform_2.png)
```csharp
  adWin.ComponentManager.Ribbon.RenderTransform
    = new System.Windows.Media.RotateTransform(
      5 );
```
![Revit ribbon bar with rotation transform](img/rh_pimp_ribbon_xform_3.png)
```csharp
  adWin.ComponentManager.Ribbon.RenderTransform
    = new System.Windows.Media.TranslateTransform(
      100, 25 );
```
![Revit ribbon bar with translation transform](img/rh_pimp_ribbon_xform_4.png)
```csharp
  adWin.ComponentManager.Ribbon.RenderTransform
    = new System.Windows.Media.SkewTransform(
      -15, 0 );
```
![Revit ribbon bar with skew transform](img/rh_pimp_ribbon_xform_5.png)
```csharp
  System.Windows.Media.TransformGroup g
    = new System.Windows.Media.TransformGroup();

  g.Children.Add( new ScaleTransform( 0.75, 0.75 ) );
  g.Children.Add( new SkewTransform( -25, 0 ) );
  g.Children.Add( new TranslateTransform( 50, 10 ) );

  adWin.ComponentManager.Ribbon.RenderTransform = g;
```
![Revit ribbon bar with several concatenated transformations](img/rh_pimp_ribbon_xform_6.png)

The black area is the container of the Ribbon; it may be possible to place your own WPF forms into it.
But this topic is more WPF than Revit API related.

#### Ribbon Transformations for AutoCAD

As said, the same thing works in AutoCAD as well.
This is the AutoCAD version:
```csharp
using Autodesk.Windows;
using Autodesk.AutoCAD.Runtime;
using System.Windows.Media;

namespace PimpMyAutoCAD
{
  public class UIApp : IExtensionApplication
  {
    public void Initialize()
    {
      try
      {
        TransformGroup group = new TransformGroup();

        group.Children.Add(
          new ScaleTransform( 0.96, 1 ) );

        group.Children.Add(
          new SkewTransform( -25, 0 ) );

        group.Children.Add(
          new TranslateTransform( 50, 0 ) );

        ComponentManager.Ribbon.RenderTransform
          = group;
      }
      catch
      {
      }
    }

    public void Terminate()
    {
    }
  }
}
```

Here is the result of running that:
![Pimp my AutoCAD transformation](img/rh_pimp_ribbon_xform_acad.png)