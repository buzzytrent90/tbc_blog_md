---
post_number: "0422"
title: "Edit Wall Length"
slug: "edit_wall_length"
author: "Jeremy Tammik"
tags: ['csharp', 'family', 'parameters', 'revit-api', 'vbnet', 'walls', 'windows']
source_file: "0422_edit_wall_length.htm"
original_url: "https://thebuildingcoder.typepad.com/blog/0422_edit_wall_length.html"
---

### Edit Wall Length

I recently posted Augusto Gonçalves
[DevTV add-in templates](http://thebuildingcoder.typepad.com/blog/2010/07/devtv-addin-templates.html),
which by the way have been updated based on some recent feedback.
Here are the updated versions and the directories to plug them in to:

- [TemplateRevitArchAddinCS.zip](zip/TemplateRevitArchAddinCS.zip): [My Documents]\Visual Studio 2008\Templates\ProjectTemplates\Visual C#- [TemplateRevitArchAddinVB.zip](zip/TemplateRevitArchAddinVB.zip): [My Documents]\Visual Studio 2008\Templates\ProjectTemplates\Visual Basic

Augusto has already done quite a bit of other work on the Revit API as well, such as preparing the DevTV presentation for the
[Revit 2010 API](http://thebuildingcoder.typepad.com/blog/2009/12/updated-devtv-introduction-to-revit-programming.html).
He will be giving his first
[Revit API training](http://thebuildingcoder.typepad.com/blog/2010/07/api-training-and-aec-devcamp-material.html) in Brazil later this week.

The best of luck to you with that, Augusto!

He also recently stepped into the next phase of supporting the Revit API by answering several DevTech support cases in this area.
Here is one of his recent cases:

**Question:** I am working on an add-in that creates window families through the Revit API.
These families sometimes consist of a single window unit, but also may be combinations of windows, in which case the add-in creates nested window families and inserts instances of the nested families into the combination family.

I want to be able to increase the length and/or height of the wall in the family document, because the window combinations can be quite large in dimensions.
I am able to increase the height of the wall using the "Unconnected Height" parameter.
However, the "Length" parameter of the wall is read-only, and my code gets an InvalidOperationException if Set is called on the "Length" parameter.

Is there some other way to increase the length of this wall?

**Answer:** You are right, the Length is read-only.
You can use the LocationCurve.Curve instead, which is write enabled.
The following code snippet shows how to move the point and update the location:
```csharp
  // get the current wall location

  LocationCurve wallLocation = myWall.Location
    as LocationCurve;

  // get the points
  XYZ pt1 = wallLocation.Curve.get\_EndPoint( 0 );
  XYZ pt2 = wallLocation.Curve.get\_EndPoint( 1 );

  // change one point, e.g. move 1000 mm on X axis

  pt2 = pt2.Add( new XYZ( mmToFeet( 1000 ), 0, 0 ) );

  // create a new LineBound
  Line newWallLine = app.Create.NewLineBound(
    pt1, pt2 );

  // update the wall curve
  wallLocation.Curve = newWallLine;
```

#### More From Ko Tao

I am still on Ko Tao.
You may be surprised to hear that one of the reasons for me to come here was to practice Italian.
My friend Daniela invited me to join her here for a diving holiday.
I took a
[CMAS](http://www.cmas.org) diving exam
in swimming pools with a final test in the Zurich Lake several years ago, and never ever made any use of it by going diving in natural waters, and I also wanted to continue practicing Italian, so it sounded like a perfect idea.
Once here, we even met her friend Maurizio, another diver, more or less by coincidence, so now the three of us often hang out.
Here are Daniela and Maurizio in Hin Wong Bay, where we went for a very nice snorkelling tour:

![Maurizio and Daniela in Hin Wong Bay](img/daniela_506_maurizio_daniela_in_hin_wong_bay.jpg)

In that bay I had my first very impressive experience of swimming in the middle of a huge school of tiny fish, maybe a couple of hundred thousand of them, several cubic metres, swirling around me in all directions, like a single living mass.

And here is Daniela watching some chickens in front of the
[Alvaro Diving](http://www.divingcourseskohtao.com) office
in Alok Baan Kao Bay with the Buddha Rock in the background:

![Daniela with chickens and the Buddha Rock](img/daniela_516_chicken_daniela_buddha_alvaro_chalok_baan_kao.jpg)

Finally, here is a sweet little hut on a boat right next to the hut we were staying in until yesterday:

![Hut on a boat](img/daniela_509_hut_on_a_boat.jpg)

Now we moved to a more tranquil spot at Jul Julea Beach, away from noise and music and bars and people.