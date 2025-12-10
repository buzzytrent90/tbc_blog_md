---
post_number: "0502"
title: "Birthdays and Gaps in Shells"
slug: "gaps_in_shell"
author: "Jeremy Tammik"
tags: ['elements', 'family', 'geometry', 'revit-api']
source_file: "0502_gaps_in_shell.htm"
original_url: "https://thebuildingcoder.typepad.com/blog/0502_gaps_in_shell.html"
---

### Birthdays and Gaps in Shells

On Wednesday we held our developer conference in Munich.
It was very successful, the biggest of these events so far, and we had many interesting discussions, both during the event itself and at the dinner and pool games with partners in the evening.

Thursday we continued with a DevLab, to which any developer interested in any Autodesk API is welcome.
A couple of us DevTech guys are available, and the space is open for all discussions and questions, with no pre-defined agenda at all.

By the way, we also celebrated the birthday of a very special person, Karl Osti, the main organiser of this whole month-long series of European events:

![Carlo's birthday cake](file:////j/photo/jeremy/2010/2010-12-16_devlab_munich/carlo_birthday_cake.jpg)

Another very special person, Kristine Middlemiss, also celebrated her birthday at the DevLab in the Autodesk office in Tokyo yesterday.
Kristine works in the DevTech team with the multimedia and entertainment products:

![Kristine's birthday cake](file:////j/photo/jeremy/2010/2010-12-16_devlab_munich/kristine1.jpg)

![Kristine's card and brilliant smile](file:////j/photo/jeremy/2010/2010-12-16_devlab_munich/kristine2.jpg)

Happy birthday, Karl and Kristine, and
[many happy returns of the day](http://en.wikipedia.org/wiki/Many_Happy_Returns_%28greeting%29)!

Now we have arrived in Milano for yet another DevDay conference.
We will continue from here to Farnborough in England beginning of next week, and then finally wrap up and start preparing for Christmas.

Meanwhile, here is an interesting issue related to geometry retrieval from Revit that I encountered a couple of times in the past and that was now reported again by my colleague Katsuaki Takamizawa:

**Question:** I am creating triangulated surfaces from a Revit family representing a sink, creating STL files to export the geometry to an external application.

However, some curved surfaces end up having gaps or cracks in them:

![Gaps in shell of sink](img/gaps_in_shell_of_sink.png)

I am creating the triangulated surfaces by the following steps:

1. Retrieve the Revit element.- Retrieve its faces via Element.get\_Geometry().Objects.- Call Face.Triangulate().get\_Triangle() to obtain triangles.

There also seem to be interferences between some of the resulting faces.

I suspect this could be some issue with accuracy.
Is there any way to improve the accuracy in this case?

**Answer:** Yes, this is an issue I have run into previously:
I think you are completely correct in assuming that this has to do with the precision used internally by Revit.
There is no way to raise the accuracy used, but I made a suggestion for handling this in those previous cases which seems to help resolve the problem.

The problem is that each face is triangulated independently of the others, and the precision used by Revit is pretty low, so the neighbouring edges end up with gaps between them.

Revit makes use of a fixed approximation size when tessellating edges and comparing points for equality.
I believe the size used internally is about 1/16 of an inch, or about 0.0052 feet.
So you need to
[think big in Revit](http://thebuildingcoder.typepad.com/blog/2009/07/think-big-in-revit.html)!

My suggestion to handle this is to use the following simple algorithm:

1. While your add-in application iterates through the solid, its faces, and edges, it should keep track of all XYZ points encountered so far.- Whenever a new point is encountered, it should check whether the new point is "close" to any one of the previous points.
     If it is, ignore the new one and use the existing one instead.

Some code that may prove useful for this is my
[XyzEqualityComparer helper class](http://thebuildingcoder.typepad.com/blog/2009/05/nested-instance-geometry.html) and
the way I make use of it in the
[GetVertices method](http://thebuildingcoder.typepad.com/blog/2009/05/nested-instance-geometry.html)

If you are working with an own epsilon constant for equality comparisons, you should ensure that it is not too small, or you will be requesting a precision from Revit that Revit simply does not support nor care about.

There may be cases where this approach does not solve the issue, but for simple ones I think it should.
Hopefully, your sink geometry falls into the latter category.

You may want to add some assertions to check that the resulting geometry really does have a closed shell of bounding faces in the end with no gaps or cracks.

One developer that I discussed a similar case with reported that this suggestion helped resolve his issue with the following result:

"We modified our tool and used your hints about the precision and now apply rounding to 0.##. Many problems with the edges disappeared. Because we need to rely on the tessellation, we are going to find the edge loops by ourselves and continue the export with that data."

I hope this helps in your case as well.