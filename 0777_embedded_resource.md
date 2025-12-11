---
ai_optimized: true
author: Jeremy Tammik
complexity_score: 2.8
content_type: code_example
optimization_date: '2025-12-11T11:44:14.571103'
original_url: https://thebuildingcoder.typepad.com/blog/0777_embedded_resource.html
post_number: '0777'
reading_time_minutes: 3
series: general
slug: embedded_resource
source_file: 0777_embedded_resource.htm
tags:
- csharp
- python
- references
- revit-api
- windows
title: Retrieve Embedded Resource Image
word_count: 622
---

### Retrieve Embedded Resource Image

I had my first haircut since January this weekend :-)

The reason I had such a long break from that task is that I shaved my head completely in January.
I had been thinking of doing that sometime in my life ever since I was a teenager, but I was always way too scared to actually put it into practice.
In the meantime, three of my four kids already showed me how easy it is, and they all looked good, which reduced the threshold somewhat.
I mentioned it to my son Christopher back in November, and he immediately responded with "well, I have a machine for that; let's get down to it right away."
I was not that brave, but when he brought up the topic again in January, I was finally ready for it:

![Shaving Jeremy's head](/p/2012/2012-01-12_haircut_shave_head/haircut-small.gif)

His friend Naomi took pictures of the process and Christopher made an animated GIF from them.

I had associated many things with it, such as letting go, clearing up, connecting to the spiritual, getting lighter, accessing my own embedded resources etc...
It was a good feeling :-)

Anyway, today is the day for
[leaving, on a jet plane](http://en.wikipedia.org/wiki/Leaving_on_a_Jet_Plane)
([lyrics](http://www.lyrics007.com/John%20Denver%20Lyrics/Leaving%20on%20a%20Jet%20Plane%20Lyrics.html)),
for the
[AEC DevCamp](http://www.cvent.com/events/devcamp-2012/event-summary-56817a3b57614f8eb59ea05fcd59bc32.aspx)
in Boston.

Here is one last little not-so-spiritual and rather down-to-earth Revit API item before I pack and go.

Ah, no, yet one more intersting thing to point out that you should not miss: one of the "rarest of predictable celestial phenomena" is taking place tomorrow, June 5: a
[transit of Venus in front of the sun](http://en.wikipedia.org/wiki/Transit_of_Venus,_2012).

**Question:** Can you please provide some insight and sample code on how to retrieve an image from the resources embedded in a DLL?

I am writing an add-in that creates its own Revit ribbon.

I can access image files stored in separate external files without any problems, but I would like to deploy a single DLL without having to include a folder full of images.

Thank you very much!

**Answer:** Please take a look at the
[String Search ADN Plugin of the Month](http://thebuildingcoder.typepad.com/blog/2011/10/string-search-adn-plugin-of-the-month.html).
It demonstrates exactly what you are asking for.

The method that is probably of greatest interest to you is the NewBitmapImage one in the external application implementation class App:
```python
  /// <summary>
  /// Load a new icon bitmap from embedded resources.
  /// For the BitmapImage, make sure you reference
  /// WindowsBase and PresentationCore, and import
  /// the System.Windows.Media.Imaging namespace.
  /// </summary>
  BitmapImage NewBitmapImage(
    Assembly a,
    string imageName )
  {
    // to read from an external file:
    //return new BitmapImage( new Uri(
    //  Path.Combine( \_imageFolder, imageName ) ) );

    Stream s = a.GetManifestResourceStream(
        \_namespace\_prefix + imageName );

    BitmapImage img = new BitmapImage();

    img.BeginInit();
    img.StreamSource = s;
    img.EndInit();

    return img;
  }
```

It is used like this in to populate the push button image data:
```csharp
  d.Image = NewBitmapImage( exe, "Img1.png" );
  d.LargeImage = NewBitmapImage( exe, "Img2.png" );
  d.ToolTipImage = NewBitmapImage( exe, "Img3.png" );
```

Obviously the images have to be included in the project files and marked as embedded resources in the Visual Studio IDE, and the path within the project has to match.

All of that is demonstrated by the StringSearch sample.

Actually, since I was working on it recently anyway, here is an
[updated version of the source code for Revit 2013](file:///C:/a/src/revit/plugin/string_search/zip/StringSearch2013.zip),
including the entire Visual Studio solution and add-in manifest.