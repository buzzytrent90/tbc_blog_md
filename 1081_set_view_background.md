---
post_number: "1081"
title: "Setting the View Display Background"
slug: "set_view_background"
author: "Jeremy Tammik"
tags: ['csharp', 'family', 'geometry', 'revit-api', 'views']
source_file: "1081_set_view_background.htm"
original_url: "https://thebuildingcoder.typepad.com/blog/1081_set_view_background.html"
---

### Setting the View Display Background

Here is an exploration of various attempts to ensure that the view background is always white, regardless of all user settings like inverted background etc.

Before getting to that, I have to share an important discovery for me and my Tammik namesakes by my sister Helene, who recently consulted an Estonian translator.

My last name is Estonian and comes from 'tamm', meaning 'oak'. Tammik is a small group of oaks, or oak grove, a common North European name, cf. Eklund in Swedish and Eichenhain in German.

The translator pointed out that there is a rhyming Estonian idiomatic expression that every Tammik should know: "Tere hommikust, tulen tammikust!" – literally "Good morning, I'm coming from oak-forest!" – sounds great in Estonian and is used for a very cheerful "hello".

A very tere hommikust to you too, and back to the view background setting issue.

#### Ensuring a White View Background for Consistent Image Export

I'm using ExportImage to export a given view and would like the background colour to always be white.
If the user running the plugin has an inverted background, the exported image has a black background.
How can I force a given view to use a normal non-inverted background?

We currently do not have access to the setting for inverting the background via Options > Graphics > Invert background.

We thought about simply changing all black pixels to white in the exported image, but that would also change non-background objects, like shadowed and black coloured parts, and the result would not be good.

Apparently there is no option to set the Inverted Background through the API.
An alternative solution might be to create a large piece of geometry in the background of the view, coloured white, export the view image and remove the geometry again.

We also tried using the rendering settings like this:

```csharp
  RenderingSettings rs = view.GetRenderingSettings();
  rs.BackgroundStyle = BackgroundStyle.Color;

  ColorBackgroundSettings cbs
    = (ColorBackgroundSettings) rs
      .GetBackgroundSettings();

  cbs.Color = new Color(255,0,0);
  rs.SetBackgroundSettings(cbs);
  view.SetRenderingSettings(rs);
```

That does not work.
In fact, it seems that none of the RenderingSettings have any effect on the exported image.

Why do the render settings not apply to the view?
Apparently, exporting a normal view to a bitmap file does not classify as rendering, and the rendering settings only affect the rendering command as described in the section on 'Rendering Options' in
[What's New in the Revit 2013 API](http://thebuildingcoder.typepad.com/blog/2013/03/whats-new-in-the-revit-2013-api.html),
which says "The settings shown in the Rendering Options Dialog are exposed..."

I finally found a way to change the background to white that only works from Revit 2014 onwards:

```csharp
  var view3D = ( viewFamilyType != null )
  ? View3D.CreateIsometric( doc, viewFamilyType.Id )
  : null;

  // Ensure white background.

  Color white = new Color( 255, 255, 255 );

  view3D.SetBackground(
    ViewDisplayBackground.CreateGradient(
      white, white, white ) );
```

I thought I might as well capture this discovery and save it somewhere, e.g. in The Building Coder samples.

The
[CmdExportImage](https://github.com/jeremytammik/the_building_coder_samples/blob/master/BuildingCoder/BuildingCoder/CmdExportImage.cs) external
command looks like a suitable place, used to
[set a default 3D view orientation and export an image](http://thebuildingcoder.typepad.com/blog/2013/08/setting-a-default-3d-view-orientation.html).

That command creates a new view for exporting.
I assume the newly created view is also affected by the inverted background setting.

The updated code is available from
[The Building Coder samples](https://github.com/jeremytammik/ExportCncFab) GitHub
repository, and the version discussed here is
[release 2014.0.106.1](https://github.com/jeremytammik/the_building_coder_samples/releases/tag/2014.0.106.1).

Once again, tere hommikust!