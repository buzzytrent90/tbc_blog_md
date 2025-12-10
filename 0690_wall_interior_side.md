---
post_number: "0690"
title: "Interior Side of a Wall"
slug: "wall_interior_side"
author: "Jeremy Tammik"
tags: ['revit-api', 'views', 'walls']
source_file: "0690_wall_interior_side.htm"
original_url: "https://thebuildingcoder.typepad.com/blog/0690_wall_interior_side.html"
---

### Interior Side of a Wall

I really enjoyed my little outing in Tel Aviv on Tuesday evening to the
[5Rhythms](http://en.wikipedia.org/wiki/5Rhythms)
[City Wave](http://www.5rhythms-rivi.co.il/Preview.asp?Page=7594&WebsiteID=3751) offered
by Rivi Diamond.

#### Confusing Ramat Gan with Tel Aviv and First Hebrew Characters

Getting there turned out to be unexpectedly adventurous, because while I assumed that I was walking around in Tel Aviv, I was in fact in the neighbouring city Ramat Gan.
Two parallel streets named Jabotinsky and Arlozorov span both of them, and I was running around in the area between them asking people for directions.
I only had a map of the Tel Aviv part, and everybody pointed in completely different directions.
Extremely confusing, if you don't know the deal.

I have since heard that it can get even worse, and that there is someplace nearby where two different Jabotinsky streets start at the same point, both with number 1, obviously, and head off in opposite directions.
Anyway, I got there in the end, by bus.

I spent Wednesday back in conference space and meeting our local developer community here, followed by a very enjoyable evening with Hemi and Rani and my colleagues Partha, Adam, Philippe and Jim in the restaurant
[Boya](http://www.boya.co.il) in
the Tel Aviv port area.

Hemi showed me how to say and write אני לא מדבר עברית (ani lo mevin ivrit, I don't speak Hebrew) and לחיים (lechaim, for life, cheers).
To rescue me from having to pervert my pure ASCII HTML by saving it in Unicode format, I converted these characters to HTML escape codes using an
[online Unicode character map](http://www-atm.physics.ox.ac.uk/user/iwi/charmap.html) that
I discovered when visiting
[Cairo](http://thebuildingcoder.typepad.com/blog/2010/10/cairo-and-free-net-books.html).

Afterwards, I walked (and ran) back to the hotel in Ramat Gan so the remaining people could squeeze into one single car (and we could save the taxi).
It also enabled me to enjoy a bit of exercise and the almost full moon.

#### Revit Timeline

One interesting topic that came up at dinner with Jim Quanci and my other colleagues on Monday night in Moscow was the history and origins of Revit.
I am still searching for more information on this, and so far Anthony Hauck very kindly pointed out the an AUGI thread listing the
[Revit Timeline](http://forums.augi.com/showthread.php?t=20803).
A more verbose and descriptive but less precise and exhaustive listing is given in the
[history section](http://en.wikipedia.org/wiki/Revit#History_of_Revit) of the
[Wikipedia article on Revit](http://en.wikipedia.org/wiki/Revit).

#### Wall Orientation

Here is a simple question and succinct answer on the orientation of a wall, which we discussed in more depth when looking at
[wall compound layers](http://thebuildingcoder.typepad.com/blog/2008/11/wall-compound-layers.html),
highlighting a simple fact that was not explicitly mentioned there:

**Question:** How can I determine which side of a wall is on its interior?

**Answer:** Query the wall for its LocationCurve.
When you are standing at the wall start point and looking toward the end point of its location curve, and assuming the wall's Flipped property is false, its interior side is on the right-hand side.
If the Flipped property is true, then the interior side of the wall is on the left.

Many thanks to my colleagues Joe Yes and Katsuaki Takamizawa-san for pointing this out so clearly!