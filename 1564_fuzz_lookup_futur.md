---
post_number: "1564"
title: "Fuzz Lookup Futur"
slug: "fuzz_lookup_futur"
author: "Jeremy Tammik"
tags: ['elements', 'geometry', 'parameters', 'revit-api', 'sheets', 'transactions', 'views', 'walls']
source_file: "1564_fuzz_lookup_futur.md"
original_url: "https://thebuildingcoder.typepad.com/blog/1564_fuzz_lookup_futur.html"
---

### Connected BIM, Intuitive AI, RevitLookup and Fuzzy Comparison
We continue patrolling the well-trodden paths of BIM, AI, Revit precision and RevitLookup:
- [Connected BIM](#2)
- [Intuitive AI](#3)
- [Fuzzy Comparison](#4)
- [RevitLookup 2018.0.0.1](#5)
#### Connected BIM
Last week, I mentioned the
momentous [AI news](http://thebuildingcoder.typepad.com/blog/2017/06/ai-news-and-sub-transaction-regen.html#2) that
AlphaGo has now conclusively beaten mankind at the game of Go and is retiring from competitive play.
AI is not the only exciting thing happening with computers right now.
Much cheaper and more accessible technology with immediate and potentially revolutionary impact is provided by cheap sensors and connectedness, as explained in the Redshift article by Nicolas Mangon
on [the end of BIM as you know it – are you ready for connected BIM?](https://redshift.autodesk.com/connected-bim)
It discusses the combination of cheap sensors with GPS and connectivity, ending up with an outlook on AR and VR.
#### Intuitive AI
This in turn plays well with Maurice Conti's TED talk
on [The incredible inventions of intuitive AI](https://www.youtube.com/watch?v=aR5N2Jl8k14),
which also turns to the topic of connectedness towards the end:
> What do you get when you give a design tool a digital nervous system? Computers that improve our ability to think and imagine, and robotic systems that come up with (and build) radical new designs for bridges, cars, drones and much more – all by themselves. Take a tour of the Augmented Age with futurist Maurice Conti and preview a time when robots and humans will work side-by-side to accomplish things neither could do alone.
Back to the Revit API:
#### Fuzzy Comparison
A regularly asked question came up again in
the [Revit API discussion forum](http://forums.autodesk.com/t5/revit-api-forum/bd-p/160) thread
on [face curve loop last and next point](https://forums.autodesk.com/t5/revit-api-forum/face-curve-loop-last-and-next-point/m-p/7130212):
\*\*Question:\*\* I don't understand why the end point of one curve and first point of next curve are not in exactly the same place when I examine the edges from the top face of a curved wall.
The curve endpoint coordinates differ in the last digits. As you can see, the last three digits are 655 and 612, respectively:
![Curve endpoint digits differ](img/fuzz_curve_end_point_coords.png)
Due to this, I cannot get a closed curve loop.
Any solution or it is bug?
\*\*Answer:\*\* You need to implement a **fuzzy** comparison operator, since real-valued coordinates will almost never be exactly equal.
Here are a number of previous discussions of this topic:
- [Nested Instance Geometry](http://thebuildingcoder.typepad.com/blog/2009/05/nested-instance-geometry.html) and the first implementation of the `XyzEqualityComparer` class – I implement this container by using a generic dictionary class and attaching an equality comparison class which ensures that almost equal points are regarded as being exactly equal, allowing for some fuzziness due to numerical imprecision... The sorting process requires another fuzzy comparison routine, a `Compare` function which determines whether a given point is larger than, equal to, or smaller than another one, and correspondingly returns 1, 0 or -1.
- [Top Faces of Sloped Wall](http://thebuildingcoder.typepad.com/blog/2011/07/top-faces-of-wall.html) – Obviously, like all comparisons of real numbers, that requires us to add a fuzz factor, since real-valued coordinates will almost never be exactly equal.
-[Retrieving Unique Geometry Vertices](http://thebuildingcoder.typepad.com/blog/2012/03/melbourne-day-two.html#2) – For this, we need to implement a fuzzy method to distinguish between points that really are different, but detect that points that are nearly the same should in fact be treated as identical.
- [XYZ Vertex Lookup](http://thebuildingcoder.typepad.com/blog/2012/06/obj-model-export-considerations.html#6) – As discussed so many times in the past, the point comparison has to include some fuzz.
- [Sort and Orient Curves to Form a Contiguous Loop](http://thebuildingcoder.typepad.com/blog/2013/03/sort-and-orient-curves-to-form-a-contiguous-loop.html) – Its end point matching comparison relies on the standard Revit precision, which is around one sixteenth of an inch. Since the built-in Revit database length unit is feet, I define the following fuzz factor for that...
- [Never use Direct Comparison for Floating Point Numbers](http://thebuildingcoder.typepad.com/blog/2014/09/planes-projections-and-picking-points.html#05) – I would recommend never doing that, because they could always be off by an infinitesimal amount, in which case the comparison would return false, even if they are almost equal, within the possible precision. You need to use fuzzy comparison for floating point numbers. Look at this discussion on real number equality testing.
- [Extracting Unique Building Element Geometry Vertices](http://thebuildingcoder.typepad.com/blog/2015/04/back-from-easter-holidays-and-various-revit-api-issues.html#4) – implement a fuzzy point equality comparer and then add the points as keys to a dictionary.
OK?
#### RevitLookup 2018.0.0.1
Alexander Ignatovich, [@CADBIMDeveloper](https://github.com/CADBIMDeveloper),
aka Александр Игнатович, once again added several enhancements
to [RevitLookup](https://github.com/jeremytammik/RevitLookup),
our beloved and invaluable interactive Revit BIM database exploration tool to view and navigate element properties and relationships,
in his [pull request #34 – bugfixes and improvements](https://github.com/jeremytammik/RevitLookup/pull/34).
Here are his commits:
- [Add ability to explore geometry of view specific elements such annotative families that have no model geometry, but only view specific](https://github.com/jeremytammik/RevitLookup/pull/34/commits/c73237075d2c0bdca40b5caa57cd8675cb3c411c)
- [Display properties and methods returning `IEnumerable` as lists of elements, not of element ids](https://github.com/jeremytammik/RevitLookup/pull/34/commits/840b31ce2fa44cb72cc03197f7739566d5bd9615)
- [Improve usability – show name property (if available) not only for `Element` class objects; show parameter definition name not only in specific case for `Element.Parameters` property](https://github.com/jeremytammik/RevitLookup/pull/34/commits/8bc9662ffde4b703b4203b041c0876c63bc31685)
- [Add ability to view byte values (for example, for `FilledRegionType.Color`)](https://github.com/jeremytammik/RevitLookup/pull/34/commits/e872accb493bb7e2a1458475cefc0701375341e9)
Many thanks to Alexander for his continuous and efficient enhancement work!
Also note what a lot he achieves
by [modifying a very few lines of code](https://github.com/jeremytammik/RevitLookup/compare/2018.0.0.0...2018.0.0.1).
I integrated his improvements
in [RevitLookup](https://github.com/jeremytammik/RevitLookup)
[release 2018.0.0.1](https://github.com/jeremytammik/RevitLookup/releases/tag/2018.0.0.1).