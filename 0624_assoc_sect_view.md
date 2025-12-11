---
ai_optimized: true
author: Jeremy Tammik
complexity_score: 3.2
content_type: news
optimization_date: '2025-12-11T11:44:14.287107'
original_url: https://thebuildingcoder.typepad.com/blog/0624_assoc_sect_view.html
post_number: '0624'
reading_time_minutes: 3
series: views
slug: assoc_sect_view
source_file: 0624_assoc_sect_view.htm
tags:
- csharp
- revit-api
- views
title: Associative Section View Fix and Greece
word_count: 645
---

### Associative Section View Fix and Greece

Here I am back from a very pleasant break.

#### Break in Greece

As mentioned, I gave a Revit API training in Athens last week.
I was able to extend my stay to visit the
[Acropolis](http://en.wikipedia.org/wiki/Acropolis_of_Athens) and
spend a wonderful extended weekend on the beaches of
[Limni](http://en.wikipedia.org/wiki/Limni,_Euboea) or
[Λιμνι](http://en.wikipedia.org/wiki/Limni,_Euboea), on the island of
[Euboea](http://en.wikipedia.org/wiki/Euboea).
Very impressive, how the ancient Greeks built things that last for thousands of years, even though some renovation is required...

![Acropolis renovation](img/jeremy_acropolis.jpg)

#### Mastic Liqueur and Masticate

One interesting thing that I learned and was able to explore through first-hand experience during the training is the etymology of the English word
[masticate](http://en.wiktionary.org/wiki/masticate).
I went out for a meal with my mainly Lebanese training participants, so we were speaking English and they spoke Arabic in this Greek environment.
We had fish, salad,
[tzatsiki](http://en.wikipedia.org/wiki/Tzatziki),
water and
[ouzo](http://en.wikipedia.org/wiki/Ouzo) (τζα&τζικι, ουζο)
all very typical Greek, or rather Eastern Mediterranean.
For dessert, as a present from the Egyptian chef, came a
[mastica liqueur](http://en.wikipedia.org/wiki/Mastika),
μαστιχα.
This is produced from
[mastic](http://en.wiktionary.org/wiki/mastic),
the resin of the
[Pistacia lentiscus](http://en.wiktionary.org/w/index.php?title=Pistacia_lentiscus&action=edit&redlink=1) shrub native to the Mediterranean.
The really interesting point for me was that the same word 'mastic' is used in Arabic as well.
I was surprised.

#### Dynamic Model Update Associative Section View SDK Sample

The training was unconventional, since these guys were all unusually experienced programmers, so I never needed to explain anything, just point out briefly how things works and they would immediately say 'ok, fine; next, please'.
Mostly, in my trainings, I would like to go faster, and spend much too much time explaining .NET programming basics instead of the Revit API.
This time, as soon as I got into any .NET related details, they got impatient and urged me to get on with the next topic.
Great fun!

One thing we ended up looking at was the DynamicModelUpdate sample, which demonstrates an example of making use of the DMU or Dynamic Model Update framework.
The version in the SDK does not work out of the box right now due to a mismatch between the namespace prefix of the class implementing the external application loaded by Revit.
The add-in manifest file specifies the full class name 'Revit.SDK.Samples.DynamicModelUpdate.CS.AssociativeSectionUpdater'.

The two source files Application.cs and SectionUpdater.cs define the namespace 'DynamicModelUpdate' instead.

To fix the problem, simply replace the line
```csharp
namespace DynamicModelUpdate
```

The correct specification in both files should be
```csharp
namespace Revit.SDK.Samples.DynamicModelUpdate.CS
```

Once that is done, the sample works fine.

#### More Pictures of Beaches

To return to the unbelievably unpopulated beaches of Limni, obviously mainly due to the current economic crisis, here are some final pictures; here is Limni itself and its beach:

![Limni and its beach](file:////j/photo/jeremy/2011/2011-08-01_greece_athens_limni/617.jpg)

A pier and beach all to ourselves:

![A pier and beach all to ourselves](file:////j/photo/jeremy/2011/2011-08-01_greece_athens_limni/648.jpg)

Totally clear water:

![Totally clear water](file:////j/photo/jeremy/2011/2011-08-01_greece_athens_limni/668.jpg)

Another nice lonesome beach to sleep on:

![Another nice lonesome beach to sleep on](file:////j/photo/jeremy/2011/2011-08-01_greece_athens_limni/679.jpg)

Here are [yet more pictures](http://www.facebook.com/media/set/?set=a.2162481814488.122318.1019863650&l=7167da5cad&type=1).

A wonderful place to be, I must say.
I really am overwhelmed by the beauty and friendliness of Greece!