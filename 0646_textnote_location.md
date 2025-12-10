---
post_number: "0646"
title: "TextNote Lost in Space?"
slug: "textnote_location"
author: "Jeremy Tammik"
tags: ['elements', 'levels', 'revit-api', 'rooms', 'views']
source_file: "0646_textnote_location.htm"
original_url: "https://thebuildingcoder.typepad.com/blog/0646_textnote_location.html"
---

### TextNote Lost in Space?

Here is the result of another interesting discussion I had with Rudolf Honke of
[Mensch und Maschine acadGraph GmbH](http://www.acadgraph.de).

We already looked at some aspects of text formatting such as
[alignment](http://thebuildingcoder.typepad.com/blog/2010/04/revitlookup-and-textnote-alignment.html),
[rotation](http://thebuildingcoder.typepad.com/blog/2010/06/textnote-rotation.html) and
[size](http://thebuildingcoder.typepad.com/blog/2011/07/text-size.html) in
the past.

So whilst there are some manipulations we can perform on text notes, it is rather irking that their Location property is not accessible.

Seen through the API, the Location property value is always either a LocationPoint or LocationCurve instance.
I assume that neither of these two fit the internal location data structure of a text note position, and therefore its location is simply not exposed at all by the API.
Very sad.

As Rudolf pointed out, however, the TextNote provides a Coord property instead.

You can retrieve the X and Y coordinates of a TextNote element from its Coord property.

The Z value of the Coord property is sometimes valid, e.g. for a text note in a section view, and sometimes not, e.g. in plan view, where it is zero, in all cases we have seen so far.

These coordinate values are 'model space' coordinates, i.e. they really represent the text note location in 3D space.

As said, the Coord.Z property depends on the view type.
In a floor plan, the correct Z value may be obtained via the GenLevel.Elevation property.
The Z behaviour for other 2D view types such as drafting, detail and area plan is still unknown :-)

I also noticed that room tags in plan views have invalid Z coordinate values.
They have a valid Location property, but like the TextNote.Coord, the Z coordinate is zero.
I observe this on all my RoomTags, independent of the RoomTag.Room.Level.

So the position of several different kinds of text objects seems to be a mixture of 2D and 3D if they belong to plan views.

One hypothesis is that the 'zero Z in plan view' behaviour is shared by all types derived from SpatialElementTag, i.e. area, room and space tags.

No guarantees on any of this information, but it might a least provide some additional understanding and a starting point to locate your text note elements.

Thank you Rudolf for raising this issue and pointing out the workaround!

#### Registration for Autodesk University 2011

Registration for
[Autodesk University 2011](http://autode.sk/nftv78) in Las Vegas opens today.