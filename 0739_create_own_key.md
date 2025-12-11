---
ai_optimized: true
author: Jeremy Tammik
complexity_score: 2.3
content_type: qa
optimization_date: '2025-12-11T11:44:14.493498'
original_url: https://thebuildingcoder.typepad.com/blog/0739_create_own_key.html
post_number: 0739
reading_time_minutes: 6
series: general
slug: create_own_key
source_file: 0739_create_own_key.htm
tags:
- elements
- family
- geometry
- revit-api
- sheets
- views
title: Great Ocean Road and Creating Your Own Key
word_count: 1269
---

### Great Ocean Road and Creating Your Own Key

The
[post-DevLab evening](http://thebuildingcoder.typepad.com/blog/2012/03/melbourne-devlab.html) I
spent with Paul and Nick on Friday night turned into something of a pub crawl and one of the most pleasant post-training evenings for a long time.
Delightful company of highly talented programmers with a wide range of non-programming interests.

One of the many interesting topics that we discussed was on identifying a set of beams originating in different systems, and I suggested [creating your own key](
#2).
If you want to skip my weekend blather, please [jump straight down](
#2) to that technical topic...

After the Loop and Siglo, we had a bite to eat and a cup of cappuccino at the
[Pellegrini's](http://www.broadsheet.com.au/melbourne/food-and-drink/directory/restaurant/cafe/pellegrinis),
returned to the
[Supper Club](http://www.theage.com.au/news/bar-reviews/the-melbourne-supper-club/2006/04/03/1143916451849.html) for
a glass of red wine but left again in protest against their overpriced offerings, and finally had a little tiff with a bouncer a another rooftop bar.
As said, very entertaining and memorable evening in wonderful company.

The weekend was memorable too.
Kim and Robby & Co. were extremely nice and took me camping along the
[Great Ocean Road](http://www.visitgreatoceanroad.org.au),
which really is quite spectacular.
We spotted koalas along the road:

![Koala](file:////j/photo/jeremy/2012/2012-03-25_great_ocean_road/p1040309_koala.jpg)

I practiced at being one too:

![Jeremy in a Eucalyputus tree](file:////j/photo/jeremy/2012/2012-03-25_great_ocean_road/p1040316_jeremy_eucalyptus.jpg)

We went camping at Johanna Beach and had a swim, or rather a struggle with pretty huge waves, attempting to have one:

![Johanna Beach](file:////j/photo/jeremy/2012/2012-03-25_great_ocean_road/p1040473_johanna_beach.jpg)

It is autumn here now, of course, so the weather is turning cooler.

On Sunday we visited the
pretty amazing
[Twelve Apostles](http://en.wikipedia.org/wiki/The_Twelve_Apostles_%28Victoria%29),
of which only seven are actually visible, and
[Loch Ard Gorge](http://en.wikipedia.org/wiki/Loch_Ard_Gorge):

![Loch Ard Gorge](file:////j/photo/jeremy/2012/2012-03-25_great_ocean_road/p1040577_loch_ard.jpg)

We even caught a couple of glimpses of a seal cavorting in the waves, prepared a funeral for a poor little stranded puffer fish, were astounded by the crazy surfers doing huge waves under the bridge in Campbell:

![Puffer Fish grave](file:////j/photo/jeremy/2012/2012-03-25_great_ocean_road/p1040487_puffer_fish.jpg)

A really wonderful weekend, and I am so grateful!

Back in Melbourne, Monday morning, I met up with Paul again for a coffee in
[Pellegrini's](http://www.broadsheet.com.au/melbourne/food-and-drink/directory/restaurant/cafe/pellegrinis) and
a breakfast in
[the quarter](http://www.thequarter.com.au).
They provide no wifi access, so I am now sitting in the
[Melbourne Library](http://www.melbourne.vic.gov.au/MelbourneLibraryService/Pages/MelbourneLibraryService.aspx) instead, due to head back for the airport soon, and my long flight back home.

Meanwhile, here is another topic we discussed during the inspiring days last week.

#### Create Your Own Key!

One of the many interesting topics that came up during the Australian DevLab was raised by Nick, on identifying a set of beams originating in Rhino with their Revit model equivalents.
It made me realise how thoughtless we all tend to be, relying on identifiers created by various software packages instead of creating our own.

The issue and train of thought that led me to ponder this is the following:

**Question:** I am creating a large stadium model in Rhino with hundreds or even thousands of beams, which I then import into Revit.
That works fine.
The issue I have is the following: if I make modifications in Rhino, I wish to re-import the beams.
I could of course just delete them all in Revit and do so.
However, I might have edited them in Revit as well, in which case I would prefer to retain the edited elements and only update the ones which have been modified in Rhino.
For this, I need some way to correlate the Rhino beams with the corresponding Revit ones.

**Answer:** This is an interesting, common and generic problem.

You have two sets of elements which are related with each other and wish you correlate the corresponding members.

The non-brainer approach that we are all much too used is accepting some kind of God-given identifier such as the Revit element id.

In this specific case, if the model would have originated in Revit, we would probably automatically have opted to use that id to correlate corresponding members.

Since they originate in Rhino instead, that is not possible.
Maybe Rhino defines some kind of identifier as well, and that can be used?

Naturally, you can also define your own identifier, possibly simply based on a consecutive numbering system.
Possibly you can attach that kind of information as a sort of label to each element and use that label to select elements in both systems.

However, there is a much more natural and foolproof way to go that we all too seldom realise!

We can define an identifier for each element which is completely independent of all software systems and all numbering schemes.

In the simplest case, for instance, we might just be dealing with a single type of element based on one single point.

In that case, we can just use the insertion point itself to identify each element, providing there are no overlays, i.e. multiple elements sharing the same insertion point.

I showed how to define a comparison algorithm for points when looking at
[nested instance geometry](http://thebuildingcoder.typepad.com/blog/2009/05/nested-instance-geometry.html) and
revisited it in depth during the
[Melbourne Revit API training](http://thebuildingcoder.typepad.com/blog/2012/03/melbourne-day-two.html#2).

I implemented a related sort order and sorting algorithm for points based on this for the
[toposurface point classification](http://thebuildingcoder.typepad.com/blog/2011/03/toposurface-interior-and-boundary-points.html).

This allows me to use a standard generic .NET Dictionary class and the Revit API XYZ insertion point as a key into that dictionary.

I can easily generalise this for the slightly more complex case of straight beams.
In this case, I can consider each beam as a simple line segment.
If a line from A to B is considered identical to the one from B to A, which makes sense in many cases, we are looking at undirected line segments as opposed to directed ones.

My toposurface point analysis shows how to implement and use a generic .NET dictionary key class
[JtEdge](http://thebuildingcoder.typepad.com/blog/2011/03/toposurface-interior-and-boundary-points.html) for
undirected line segments.

**Response:** Hmm.
Interesting.

I actually have more data to store.

I would like to support more varied beam types, such as arcs, and other family instances as well.

**Answer:** No problem.

You can simply go on expanding your key class to include all the data you need.
You just need to implement it so that it really uniquely identifies each member you need to address in a normalised, comparable and sortable fashion.
For a point-based family instance, the data might include the instance type and the point.
For an arced beam, you might use three points, etc.

Lots of areas of mathemathics require
[normalisation](http://en.wikipedia.org/wiki/Database_normalization).
In this case, I just mean that every element that you want to compare equal to another really generates the same key.