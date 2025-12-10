---
post_number: "0440"
title: "Adding a Rebar inside a Rebar Cover"
slug: "rebar_inside_rebar_cover"
author: "Jeremy Tammik"
tags: ['csharp', 'revit-api']
source_file: "0440_rebar_inside_rebar_cover.htm"
original_url: "https://thebuildingcoder.typepad.com/blog/0440_rebar_inside_rebar_cover.html"
---

### Adding a Rebar inside a Rebar Cover

The
[Pantagruel](http://www.classicsailing.de) arrived
safely in Göteborg last night after our week-long tour of Norway, Denmark and the Swedish west coast, and our last day has arrived.
Time to clean up, disembark, go to the airport and return home again.

Meanwhile, here is an interesting structural rebar issue raised by my colleagues Katsuaki Takamizawa and Tim Culver:

**Question:** Here are some rebars that were created using the Revit API NewRebar method:

![Rebar and rebar cover](img/rebar_cover.png)

In the user interface, rebars can only be added inside of a rebar cover represented by the green dotted line.
If they are generated using the following code in a column where xVec and yVec fit the width and depth of the column profile, they can extend beyond the rebar cover.
```csharp
Rebar createdRebar
= doc.Create.NewRebar( barShape, barType,
column, origin, xVec, yVec );
```

Is it possible to automatically ensure the rebar placement inside of a rebar cover when generating it programmatically?
Or does one have to adjust the location and scale whenever a rebar is added?
For instance, using ScaleToBox:

```
  createdRebar.ScaleToBox( origin, xVec, yVec );
```

**Answer:** Keep in mind that Revit does not enforce the cover when it is generated programmatically.
However you create the bars, you need to account for the cover yourself.
You can specify the location and scale when generating or adjust by ScaleToBox to place a rebar inside the cover.
ScaleToBox is the same algorithm used internally by the UI.

However, Revit does perform one adjustment automatically: if you place the rebar so that the centre line is close to the cover (within a bar radius, I think) and parallel, the next call to the Regenerate method will move it to attach to the cover.
You can see this by moving the rebar in the UI.

There are actually three ways to create and scale the rebar: using the origin and vectors, create from curves, and ScaleToBox, where the latter requires you to create the bar in one of the other ways first.

No matter how you create the bars, Revit never enforces cover, except in case the rebar is very close and parallel.
I.e., when creating rebars programmatically, the cover cannot be enforced like it is in the user interface, the developer has to take care of that explicitly herself.

Many thanks to Katsu-san and Tim for this explanation!