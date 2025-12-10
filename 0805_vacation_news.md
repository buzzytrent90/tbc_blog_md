---
post_number: "0805"
title: "Vacation Time and Various Notes"
slug: "vacation_news"
author: "Jeremy Tammik"
tags: ['elements', 'family', 'parameters', 'revit-api', 'views', 'walls']
source_file: "0805_vacation_news.htm"
original_url: "https://thebuildingcoder.typepad.com/blog/0805_vacation_news.html"
---

### Vacation Time and Various Notes

To tell the truth, today is not a vacation day, but the
[Swiss national holiday](http://en.wikipedia.org/wiki/Swiss_National_Day),
right in the middle of my vacation.

In my last post, I mentioned various urgent and important issues that I still wanted to take care of.

I am very happy to tell you that these items all felt a lot less urgent and important after immersing myself in a couple of days of vacation in Avignon to visit family and attend some offerings of one of the world's biggest theatre events, the annual
[Avignon theatre festival](http://www.festival-avignon.com),
and especially its less official
['OFF'](http://www.avignonleoff.com) sibling,
initiated by independent theatre companies.

![Avignon theatre festival](file:////j/photo/jeremy/2012/2012-07-28_avignon/collage.jpg)

The main reason for my visit were my relatives, though, and not the festival, although I did attend a couple of shows and loved the buzzing activity everywhere.

I also went climbing in
[Collias](http://fr.wikipedia.org/wiki/Collias),
paddling on the
[Gardon](http://en.wikipedia.org/wiki/Gardon),
enjoyed the spectacular
[Pont du Gard](http://en.wikipedia.org/wiki/Pont_du_Gard),
and revelled in the atmosphere and warm summer nights in the city.
![Avignon theatre festival](file:////j/photo/jeremy/2012/2012-07-29_avignon/08.jpg)

As I once
[mentioned](http://thebuildingcoder.typepad.com/blog/2009/06/adding-a-shared-parameter-to-an-rfa-file.html),
I also love the trip itself down to Avignon along back roads and desolate river gorges.
The main discovery this time around was the beautiful
[Lac du Bourget](http://en.wikipedia.org/wiki/Lac_du_Bourget),
where I was mysteriously and magically lent a gorgeous wooden canoe to go paddling around in between rounds of swimming.
![Lac du Bourget](file:////j/photo/jeremy/2012/2012-07-29_avignon/82.jpg)

Meanwhile, here are a couple of work and Revit API related thingies that were either pending publication or hanging around my inbox that I would like to share with you:

- [Revit API overview](#2)- [Presenting colour coded source code](#3)- [Preview control with linked document](#4)- [Calculating the mass of a building element](#5)

#### Revit API Overview

We regularly hear requests for a Revit API overview.

The first and foremost place to look is in the material provided in the
[Revit Developer Center](http://www.autodesk.com/developrevit) and
in the
[developer guide wiki](http://thebuildingcoder.typepad.com/blog/2012/04/developer-center-and-sdk-update.html#0).
The Revit Developer Center provides a host of self-learning material as well.

By the way, due to popular demand, a new lesson on debugging has been added to the
[My First Revit Plug-in](http://www.autodesk.com/myfirstrevitplugin) tutorial:
[Lesson 4: Debugging the code](http://usa.autodesk.com/adsk/servlet/index?siteID=123112&id=20132893).

Both the
[Revit 2012 API Labs](http://images.autodesk.com/adsk/files/revit_2012_api_training.zip)
and the
[Revit API Webcast Archive](http://www.adskconsulting.com/adn/cs/api_course_webcast_archive.php)
include various versions of API overviews as well.

The
[DevDays Online presentation on What's New in Revit 2013](http://thebuildingcoder.typepad.com/blog/2012/05/devdays-online-on-whats-new-in-revit-2013.html) and
the follow-up
[Revit 2013 API webcast](http://thebuildingcoder.typepad.com/blog/2012/05/revit-2013-api-webcast-recording.html)
complementing it both discuss new features of the Revit 2013 API, and the first half of the latter is completely dedicated to the basics of the Revit API in general and is independent of the current version.

Ritchie Jackson also created a complete
[Revit API Introduction](http://thebuildingcoder.typepad.com/blog/2011/04/ritchies-revit-api-introduction.html) from
a completely different point of view.

#### Presenting Colour Coded Source Code

Here are some tips on pasting colour coded source code into a PowerPoint slide deck.

##### a) Simple One-step Approach via HTML

- Use CopyToHTML in Visual Studio to copy the colour coded source code into the clipboard in HTML format.- In PowerPoint, use Home > Paste > Arrow down > Paste Special... > HTML format to paste.

##### b) Safer Two-step Approach via Word and RTF

You may have a still better result using Word in an intermediate step.
In that case, you don't even need to copy as HTML from VS; you can just use the standard copy.
In Word, first paste the code, then select and copy it again from Word.
Now, in PowerPoint, use Home > Paste > Arrow down > Paste Special... > RTF format to paste it in.

##### Colour Coded Source Code for Black Slides

If your slides use a black background, and you are using a white background in VS, your black source code will be invisible. In that case, in VS, you can go to Tools > Options... > Environment > Fonts and Colors > Item background and set that to black, set the item foreground to white, and hit OK.

Another option besides using CopySourceAsHtml is Notepad++, which supports many other languages, including JavaScript.

#### Preview Control with Linked Document

One if the new
[Revit 2013 API](http://thebuildingcoder.typepad.com/blog/2012/03/revit-2013-and-its-api.html) features is the
[preview control](http://thebuildingcoder.typepad.com/blog/2012/03/revit-2013-and-its-api.html#2),
supporting enhanced integration between Revit and an add-in.

I presented a minimal sample named
[PreviewControlSimple](http://thebuildingcoder.typepad.com/blog/2012/06/devcamp-day-two.html#23) making
use of it in my DevCamp session on the
[Revit 2013 UI API enhancements](http://thebuildingcoder.typepad.com/blog/2012/06/devcamp-day-two.html#2),
and a more complete and complex one is given by the
[UIAPI Revit SDK sample](http://thebuildingcoder.typepad.com/blog/2012/03/new-revit-2013-sdk-samples.html#4).

Here is a question that was not yet addressed, though:

**Question:** Is it possible to use the preview control with a linked document?
I tried to do so, and it throws an exception saying "Cannot preview a linked document".

**Answer:** This is by design: a linked document cannot be previewed using this control.

One workaround would be to close the project document, open the linked document and preview it then.

Another would be to create a new view in your project, display the linked instance in it, hide all other instances, and send that view to the preview control.

#### Calculating the Mass of a Building Element

And finally, another pretty fundamental little BIM question that has been hanging around waiting to be published for a while:

**Question:** How can I determine the mass of a building element?

I am working with data sets providing certain crucial calculation values based on material mass.

I can see how to calculate volume and area in the Revit API, but I see no API call to calculate mass.

Is there any way to calculate a Revit model element's material mass through the API?

**Answer:** You can query the material for its density, and ask the building element for its volume.

The volume obviously needs to be provided for several different materials per building element, e.g. for the different layers of the compound structure of a complex wall or floor, so you can use the Category.HasMaterialQuantities property to check which materials contribute volume and the Element.GetMaterialVolume method to determine the volume of a specific material for a given element.

The density can be obtained from the Density property on StructuralAsset, which can be retrieved from Material.StructuralAssetId and PropertySetElement.GetStructuralAsset.

A related issue and a different access to the density in the context of
[AMEE](http://www.amee.com) is
discussed by David Laing in his thread on
[calculating element material mass](https://github.com/AMEE/revit/issues/18).

That is all from me for now.
I still have to relax a bit more...