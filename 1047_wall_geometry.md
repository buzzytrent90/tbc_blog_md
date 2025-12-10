---
post_number: "1047"
title: "Wall Compound Layer, Other Geometry and Licenses"
slug: "wall_geometry"
author: "Jeremy Tammik"
tags: ['csharp', 'doors', 'elements', 'geometry', 'revit-api', 'rooms', 'selection', 'walls', 'windows']
source_file: "1047_wall_geometry.htm"
original_url: "https://thebuildingcoder.typepad.com/blog/1047_wall_geometry.html"
---

### Wall Compound Layer, Other Geometry and Licenses

Below, we take another quick look at the issue of [licenses](#2).

First, here is a pretty basic question that keeps reappearing, so I summarise some existing results for everybody's convenience:

**Question:** How can I programmatically retrieve the coordinates of a wall boundary, especially the geometry of the finished interior and exterior layers?

I wish to export the building blueprint to GML.

I would also like to know how to achieve the same for the geometry of doors and windows.

Some C# sample code would be very helpful.

**Answer:** We discussed the topic of retrieving geometry in all conceivable variations here in the past, so numerous C# samples for achieving that are returned immediately by simply searching the web for
[building coder get\_geometry](http://lmgtfy.com/?q=building+coder+get_geometry).

You will also find a number of examples of retrieving element geometry in the Revit SDK samples.
Again, just search globally for 'get\_Geometry'.

For specific details on the location of wall layers and their boundaries, here are some of different aspects we already looked at:

- [Wall compound layers](http://thebuildingcoder.typepad.com/blog/2008/11/wall-compound-layers.html)
- [Compound wall layer volumes](http://thebuildingcoder.typepad.com/blog/2009/02/compound-wall-layer-volumes.html)
- [Retrieving detailed wall layer geometry](http://thebuildingcoder.typepad.com/blog/2011/10/retrieving-detailed-wall-layer-geometry.html)
- [Interior side of a wall](http://thebuildingcoder.typepad.com/blog/2011/12/interior-side-of-a-wall.html)
- [Identifying wall compound layers and parts](http://thebuildingcoder.typepad.com/blog/2012/01/identifying-wall-compound-layers-and-parts.html)
- [Updating the wall compound layer structure](http://thebuildingcoder.typepad.com/blog/2012/03/updating-wall-compound-layer-structure.html)
- [Setting the compound structure core and shell layers](http://thebuildingcoder.typepad.com/blog/2013/08/setting-the-compound-structure-core-and-shell-layers.html)
- [Creating an offset wall](http://thebuildingcoder.typepad.com/blog/2013/09/creating-an-offset-wall.html)

Talking about wall boundaries, we had an extensive related discussion on
[determining the room boundary segment generating element](http://thebuildingcoder.typepad.com/blog/2013/10/determining-a-room-boundary-segment-generating-element.html) –
a wall, in fact, nine time out of ten – just yesterday, pointing to numerous previous examples of retrieving room boundaries, ray shooting to discover neighbouring elements, and also looking at the
[spatial element boundary location](http://thebuildingcoder.typepad.com/blog/2013/10/determining-a-room-boundary-segment-generating-element.html#4) options.

That should provide more than enough information for most wall layer related issues.

Good luck, and have fun!

#### More on Licenses

Now that I pointed out
[the importance of a license](http://thebuildingcoder.typepad.com/blog/2013/10/the-building-coder-samples-on-github.html#2),
several people asked me to say more about this complex topic, both from inside and outside of Autodesk.

Actually, maybe, it is pretty simple, and the legalese and numerous variations just make it **appear** complex.

I looked at the
[Creative Commons](http://creativecommons.org) license
and like it a lot at first glance, but there seems to be some discussion on
[applying a Creative Commons license to software](https://github.com/github/choosealicense.com),
especially GitHub projects.

That discussion led me to the exhaustive
[GNU list of licenses and comments about them](https://www.gnu.org/licenses/license-list.html#Unlicense),
including a guide on
[how to choose a license for your own work](https://www.gnu.org/licenses/license-recommendations.html).

Being GNU, the recommendations veer heavily towards the
[Copyleft](http://en.wikipedia.org/wiki/Copyleft),
which is not suitable for commercial software, including many Revit and other Autodesk software add-ins.

However, it also states: "In these special situations where copyleft is not appropriate, we recommend the
[Apache License 2.0](http://www.apache.org/licenses/LICENSE-2.0)."

After reading through the Apache license, as far as I can fathom it, that makes a lot of sense, and I am considering switching to it from the MIT license that I chose for my initial GitHub projects.

Actually, searching the GNU list more thoroughly for the MIT license, I found this entry:

"[Expat License](http://directory.fsf.org/wiki/License:Expat):
This is a lax, permissive non-copyleft free software license, compatible with the GNU GPL.
It is sometimes ambiguously referred to as the ***MIT License***.
For substantial programs it is better to use the Apache 2.0 license since it blocks patent treachery."

Since my samples are all not very substantial, I am not worrying about the warning in the last statement, but I will indeed switch to the Apache 2 license for future submissions.

It has the added advantage of apperaing right at the top of the new GitHub repository license selection list :-)