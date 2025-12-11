---
ai_optimized: true
author: Jeremy Tammik
complexity_score: 3.3
content_type: qa
optimization_date: '2025-12-11T11:44:16.533136'
original_url: https://thebuildingcoder.typepad.com/blog/1692_family_tamper_protect.html
post_number: '1692'
reading_time_minutes: 4
series: family
slug: family_tamper_protect
source_file: 1692_family_tamper_protect.md
tags:
- family
- geometry
- references
- revit-api
- sheets
title: Family Tamper Protect
word_count: 818
---

### Protecting a Family from Tampering
The [Forge](https://autodesk-forge.github.io)
[accelerator in Rome](http://autodeskcloudaccelerator.com) is
winding down with the demonstrations of what was achieved being recorded as I write.
An interesting conversation I had at the celebratory dinner last night gave me an idea for a solution to a longstanding question on family tampering protection:
- [Roma accelerator group photo](#2)
- [Protecting a family from tampering](#3)
- [Implementing a canonical key for geometrical objects](#4)
#### Roma Accelerator Group Photo
I missed the Roma Accelerator group photo by a few seconds:
![Roma accelerator participants lacking Jeremy](img/accelerator_roma_participants_400x300.jpg)
In what certainly must be one of the worst photoshop jobs ever, Jaime very kindly added me to it afterwards:
![Roma accelerator participants plus Jeremy](img/accelerator_roma_participants_plus_jeremy_400.jpg)
Thank you for that, Jaime! Keep practicing...
#### Protecting a Family from Tampering
Here are two different aspects of protecting Revit families:
- Copy protection, to stop the competition from stealing them and the [IP or intellectual property](https://en.wikipedia.org/wiki/Intellectual_property) they contain.
- Tampering protection, to stop the customer from corrupting her own models, intentionally or not.
Often discussed, various solutions suggested.
For example, we discussed one example of a copy protection solution
by [simplifying nested family instances](http://thebuildingcoder.typepad.com/blog/2018/06/simplifying-nested-family-instances.html) to
protect the intellectual property built into a complex hierarchy of nested family instances by replacing them with a flatter and simpler hierarchy, yet retaining all the relevant non-confidential custom data.
Here is an AUGI discussion from 2009 on the pros and cons
of [protecting Revit families](http://forums.augi.com/showthread.php?73233-Protecting-Revit-Families).
Today, I make a suggestion to protect from tampering, nothing totally safe yet afaik.
You want a fool-proof method to protect your family definitions from tampering?
You can encode a secret hash code or something and hide it somewhere.
For brevity, let's call this secret hash code your key.
However, the tamperer might just retain it, either intentionally or not.
How can you still detect tampering?
Well, you can easily encode bits and pieces of whatever she might want to tamper with into you key as well.
Especially, you should ensure that you generate a hash key or something that includes references to every single bit of data that is relevant for you.
If you have an algorithm to rebuild your key at will from the current state of the family, you can check the following:
- Does the original key exist? If not, alarm.
- Recompute the current key from the current state of the family. does it match the stored key? If not, alarm.
[Creating your own key](http://thebuildingcoder.typepad.com/blog/2012/03/great-ocean-road-and-creating-your-own-key.html#2) can
be useful in numerous ways, so I already discussed this topic and suggested doing so back in 2012.
Therefore, I hope that all the families that you care about are already adequately protected against tampering.
If not, go ahead and do so now.
#### Implementing a Canonical Key for Geometrical Objects
To further illustrate what I mean by the key I mentioned above, let's illustrate how to generate a checksum or hash code from geometry, preferably using
a [canonical form](https://en.wikipedia.org/wiki/Canonical_form).
Let's explore defining a simple canonical form for a point, line (two ordered points), curve, etc.
A very simple and useful way to go is to define your canonical form as a string.
You can use this string as a dictionary key, if that fits your needs.
You can also calculate a hash from it, which should ensure that any change in the string will also cause a change in the hash.
To ensure that this works, you need to create a hash of sufficient length.
If you care about protection, you might also want to encrypt the hash after you have created it.
Here are some simplistic examples of defining canonical forms for real numbers, points, lines, and curves:
- Real number `a`, using as many decimal places as required to achieve the precision you need:
```csharp
public static string RealString( double a )
{
return a.ToString( "0.######" );
}
```
- `XYZ p`:
```csharp
public static string PointString( XYZ p )
{
return string.Format( "({0},{1},{2})",
RealString( p.X ),
RealString( p.Y ),
RealString( p.Z ) );
}
```
- A sorted array of points:
```csharp
public static string PointArrayString( IList pts )
{
return string.Join( ", ",
pts.Select(
p => PointString( p ) ) );
}
```
- A bounded line `(p,q)`:
- Sort `p` and `q` lexicographically
- Return `PointArrayString( [p,q] )`
- A curve `c`:
- Return `PointArrayString( c.Tessellate() )`
Most of these are implemented
in [The Building Coder samples](https://github.com/jeremytammik/the_building_coder_samples)
[Util module](https://github.com/jeremytammik/the_building_coder_samples/blob/master/BuildingCoder/BuildingCoder/Util.cs).
I'll leave it up to your imagination to improve these and define canonical keys for more complex data as required.
Migliori saluti a Stefano!