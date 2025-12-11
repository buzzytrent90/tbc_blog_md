---
ai_optimized: true
author: Jeremy Tammik
complexity_score: 7.2
content_type: qa
optimization_date: '2025-12-11T11:44:13.612040'
original_url: https://thebuildingcoder.typepad.com/blog/0244_ribbon_embed_image.html
post_number: '0244'
reading_time_minutes: 4
series: general
slug: ribbon_embed_image
source_file: 0244_ribbon_embed_image.htm
tags:
- csharp
- elements
- references
- revit-api
- windows
title: Ribbon Embed Image
word_count: 741
---

### Ribbon Embed Image

Previous posts discussed the
[Revit 2010 Ribbon API](http://thebuildingcoder.typepad.com/blog/2009/05/revit-2010-ribbon-api.html),
the associated Ribbon SDK Ribbon sample, and my use of it for the
[MEP sample ribbon panel](http://thebuildingcoder.typepad.com/blog/2009/08/mep-sample-ribbon-panel.html).
Krispy5 very friendlily posted a series of
[comments](http://thebuildingcoder.typepad.com/blog/2009/08/mep-sample-ribbon-panel.html#comment-6a00e553e1689788330120a6a18e08970c) to
the latter describing a detailed solution to embed the images used in the ribbon buttons into the Revit external application executable assembly.
I implemented a sample application RibbonEmbedImage to make Krispy's solution more readable and readily available.
This sample is obviously less a demonstration of any Revit API functionality;
rather it shows how to make use of Visual Studio and the .NET framework to handle bitmaps as embedded resources.

[RibbonEmbedImage](zip/RibbonEmbedImage.zip) implements the following:

- An embedded bitmap resource with its build action set to 'Embedded Resource'.- Krispy's method GetEmbeddedImage to read the embedded bitmap resource and return a BitmapSource instance.- A method AddRibbonPanel to create a custom ribbon panel labelled "Ribbon Embed Image" containing a push button labelled "Hello" and displaying the embedded resource bitmap as its image.- An external application calling AddRibbonPanel in its OnStartup method.- An external command implementing a trivial Execute method which is invoked by the Hello button.

This is what the resulting custom ribbon panel looks like when I drag it off the ribbon to a free floating state:

![RibbonEmbedImage sample custom ribbon panel](img/RibbonEmbedImage.png)

Let's look at the source code of the items mentioned above one by one.
There is no code to display for the bitmap, but as said, it needs to have its build action set to 'Embedded Resource'.

Here are the namespaces we make use of:
```csharp
using System;
using System.Collections.Generic;
using System.Diagnostics;
using System.IO;
using System.Reflection;
using System.Windows.Media.Imaging;
using Autodesk.Revit;
```

Besides the standard references to System and RevitAPI, these will require references to the following assemblies:

- PresentationCore- WindowsBase

The application defines its own namespace RibbonEmbedImage, and within it two classes named App and Command, i.e. the following global source code structure:
```csharp
namespace RibbonEmbedImage
{
  public class App : IExternalApplication
  {
  }

  public class Command : IExternalCommand
  {
  }
}
```

Within the App class, we first of all define a constant message to be used as the ribbon panel button tooltip
and displayed by the external command invoked by the button:
```csharp
public const string Message =
  "Ribbon Embed Image says 'Hello world'"
  + " via an embedded bitmap resource.";
```

Then we add the definition of Krispy's method GetEmbeddedImage to read the embedded bitmap resource and return a BitmapSource instance:
```csharp
static BitmapSource GetEmbeddedImage( string name )
{
  try
  {
    Assembly a = Assembly.GetExecutingAssembly();
    Stream s = a.GetManifestResourceStream( name );
    return BitmapFrame.Create( s );
  }
  catch
  {
    return null;
  }
}
```

It extracts a bitmap resource embedded within the executing assembly and returns a corresponding BitmapSource object.
The 'name' argument is the fully qualified resource name, i.e. includes the application namespace etc.

Next comes the AddRibbonPanel method which creates a custom ribbon panel labelled "Ribbon Embed Image" containing a push button labelled "Hello" and displaying the embedded resource bitmap as its image:
```csharp
static void AddRibbonPanel(
  ControlledApplication a )
{
  string path = Assembly.GetExecutingAssembly().Location;

  PushButtonData data = new PushButtonData(
    "Hello", "Hello", path, "RibbonEmbedImage.Command" );

  BitmapSource bitmap = GetEmbeddedImage(
    "RibbonEmbedImage.Bitmap1.bmp" );

  data.Image = bitmap;
  data.LargeImage = bitmap;
  data.ToolTip = Message;

  RibbonPanel panel = a.CreateRibbonPanel(
    "Ribbon Embed Image" );

  RibbonItem item = panel.AddButton( data );
}
```

The only remaining part of the external application class is the implementation of the required interface methods and the call to AddRibbonPanel in the OnStartup method:
```csharp
public IExternalApplication.Result OnStartup(
  ControlledApplication a )
{
  AddRibbonPanel( a );
  return IExternalApplication.Result.Succeeded;
}

public IExternalApplication.Result OnShutdown(
  ControlledApplication a )
{
  return IExternalApplication.Result.Succeeded;
}
```

All that is needed for the external command is the implementation of the required interface, i.e. the Execute method.
It reuses the application class Message constant and simply passes it back as a warning message to Revit:
```csharp
    public IExternalCommand.Result Execute(
      ExternalCommandData commandData,
      ref string message,
      ElementSet elements )
    {
      message = App.Message;
      return IExternalCommand.Result.Failed;
    }
```

Here is the complete
[RibbonEmbedImage](zip/RibbonEmbedImage.zip) source code and Visual Studio solution.

Many thanks to Krispy for providing this solution!
I hope it will prove useful to everybody needing bitmaps or any other resources in a Revit plug-in, whether it be for ribbon panel buttons or other purposes.