---
post_number: "1490"
title: "The Building Coder"
slug: "create_line_style"
author: "Jeremy Tammik"
tags: ['elements', 'family', 'filtering', 'geometry', 'revit-api', 'sheets', 'transactions', 'walls']
source_file: "1490_create_line_style.md"
original_url: "https://thebuildingcoder.typepad.com/blog/1490_create_line_style.html"
---

### How to Create a New Line Style
I arrived safe and sound in Munich to provide support for the
one-week [Forge accelerator](http://autodeskcloudaccelerator.com) workshop
at the Autodesk offices here.
- [Arrival in Munich and Hotel Schlicker](#2)
- [Official pictures from RTC in Porto](#3)
- [Art of README](#4)
- [What is Join?](#5)
- [How to Create a Line Style](#6)
#### Arrival in Munich and Hotel Schlicker
I am visiting a friend here in Munich starting today and needed a hotel for the first night.
I had no time to research and book anything in advance, so I just got off the subway at Marienplatz and started walking.
As luck would have it, I discovered the nicest hotel I have ever stayed at.
With just three stars and a moderate price,
[Hotel Schlicker](http://www.hotel-schlicker.de) beats
every single one of the five-star global hotel chain establishments that I normally have the bad luck to end up in.
Highly recommended!
#### Official Pictures from RTC in Porto
Here are links to the official photo collections from
the [RTC Revit Technology Conference Europe](http://www.rtcevents.com/rtc2016eur) in
Porto during the last few days:
- [Highlights Day 1](https://www.dropbox.com/sh/4ni6jamslxbc2yn/AACcdvHV_E4YogTswTF4nBPza?dl=0)
- [Highlights Day 2](https://www.dropbox.com/sh/7lqifbca203ttg1/AAB3IksT6rRdY3Ofs-xGe9sea?dl=0)
- [Highlights Day 3](https://www.dropbox.com/sh/m4rhkakiseh7pyz/AADZxZYhJcLWF41Utw7wb2AOa?dl=0)
Thanks to Lejla Secerbegovic for sharing these!
Here are my own pictures from the RTC Saturday evening gala dinner in the [Palácio da Bolsa](https://en.wikipedia.org/wiki/Pal%C3%A1cio_da_Bolsa), including the one and only Jay Zallan in his beautiful stripy fur coat in the arabic hall, a perfect fit tor the highlight of the palace, decorated in the exotic Moorish Revival style, used as reception hall for visiting personalities and heads of state:
[![RTC Gala Dinner](https://c3.staticflickr.com/6/5645/29924672034_fb23f87594_n.jpg)](https://www.flickr.com/photos/jeremytammik/albums/72157672114668283 "RTC Gala Dinner")
#### Art of README
A very nice and fundamentally important article for anyone sharing code, whether on GitHub or elsewhere, pointed out by [Philippe Leefsma](http://twitter.com/F3lipek):

[Art of README](https://github.com/noffle/art-of-readme)
Thank you, Philippe, for this instructive read.
#### What is Join?
I made a shocking discovery in a discussion with a participant here at the Forge accelerator.
I thought the Revit 'Join' operation was a sort of Boolean operation.
Apparently, that is not the case at all, at least not always.
In some cases, it is just a means to suppress the display of certain edges between adjacent collinear and coplanar elements of the same type and material.
Maybe this is also the functionality driven by the [JoinGeometryUtils class](http://www.revitapidocs.com/2017/c45b6484-3efd-1d81-0b47-ba678857fff1.htm)?
Not a Boolean operation at all?
Under other circumstances, it can apparently mean other things.
For instance, in the family editor, joining two solids may or may not cause a Boolean operation, which may or may not correspond to the [CombineElements method](http://www.revitapidocs.com/2017/5c33a711-2891-f353-5f39-24ba175be452.htm).
I would love to have more clarity on this, and so might others as well...
#### How to Create a Line Style
Scott Conover, Senior Engineering Manager of the Revit development team, pointed out the following important Revit 2017 API enhancement that now fully enables the programmatic creation of new line styles:
\*\*Question:\*\* How can I create a new line style?
\*\*Answer:\*\* Creating a line style is really easy.
One little stumbling block relates to terminology, as there is no `LineStyle` object in the Revit API.
The `LineStyle` is in fact represented by a `GraphicsStyle` element in the API.
Actually new `LineStyle` instances are defined as subcategories of `Line`, so are you can set one up by making a new subcategory and setting its weight, colour, and pattern as you please.
Up until 2017, there was a gap around assigning the line pattern to the category.
Happily, this is now exposed in Revit 2017.
Here is a macro that creates and sets up a new Line Style:
```csharp
///
/// Create a new line style using NewSubcategory
/// summary>
void CreateLineStyle(Document doc)
{
// Use this to access the current document in a macro.
//
//Document doc = this.ActiveUIDocument.Document;
// Find existing linestyle. Can also opt to
// create one with LinePatternElement.Create()
FilteredElementCollector fec
= new FilteredElementCollector( doc )
.OfClass( typeof( LinePatternElement ) );
LinePatternElement linePatternElem = fec
.Cast()
.First( linePattern
=> linePattern.Name == "Long dash" );
// The new linestyle will be a subcategory
// of the Lines category
Categories categories = doc.Settings.Categories;
Category lineCat = categories.get_Item(
BuiltInCategory.OST_Lines );
using( Transaction t = new Transaction( doc ) )
{
t.Start( "Create LineStyle" );
// Add the new linestyle
Category newLineStyleCat = categories
.NewSubcategory( lineCat, "New LineStyle" );
doc.Regenerate();
// Set the linestyle properties
// (weight, color, pattern).
newLineStyleCat.SetLineWeight( 8,
GraphicsStyleType.Projection );
newLineStyleCat.LineColor = new Color(
0xFF, 0x00, 0x00 );
newLineStyleCat.SetLinePatternId(
linePatternElem.Id,
GraphicsStyleType.Projection );
t.Commit();
}
}
```
I added Scott's code
to [The Building Coder samples](https://github.com/jeremytammik/the_building_coder_samples)
[release 2017.0.131.0](https://github.com/jeremytammik/the_building_coder_samples/releases/tag/2017.0.130.4)
in the new module [CmdCreateLineStyle.cs](https://github.com/jeremytammik/the_building_coder_samples/blob/master/BuildingCoder/BuildingCoder/CmdCreateLineStyle.cs).
Thank you very much, Scott, for pointing out and demonstrating this important enhancement!