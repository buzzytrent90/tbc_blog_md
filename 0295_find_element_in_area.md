---
post_number: "0295"
title: "Find an Element in an Area"
slug: "find_element_in_area"
author: "Jeremy Tammik"
tags: ['elements', 'filtering', 'geometry', 'levels', 'revit-api', 'selection']
source_file: "0295_find_element_in_area.htm"
original_url: "https://thebuildingcoder.typepad.com/blog/0295_find_element_in_area.html"
---

### Find an Element in an Area

Another last interlude before concluding the series on the Revit API geometry library based on Scott Conover's Autodesk University presentation on
[analysing building geometry](http://thebuildingcoder.typepad.com/blog/2010/01/analyse-building-geometry.html),
also dealing with geometrical analysis of Revit elements:

**Question:** How can I select a Revit element within a particular area, for instance the rectangle defined by the points (0,0) and (500,500) in the XY plane?

**Answer:** There are several different approaches to this, and you will obviously have to decide for yourself which one best suits your needs.
Basically, you need to determine the location of your Revit element in some way, and then test whether this location lies within the specified area.
You can use different approaches to determine the element location.
Some properties and data that might be useful for this include:

- The BoundingBox property.- The Location property.- The element geometry.

In previous posts, I presented the
[GetVertices](http://thebuildingcoder.typepad.com/blog/2009/05/nested-instance-geometry.html) method
as an example of retrieving a list of vertices from the element geometry, and the
[GetElementLocation](http://thebuildingcoder.typepad.com/blog/2009/10/unrotate-north.html) method
determining a single point from the element location.
These will probably both provide useful starting points and still need to be adapted for your specific needs.

Once you have determined the element location in some way, you need to decide what exactly it means that the element lies within the area.
For instance, do you want to just determine one specific point for an element, and check whether that lies within the area, or use the entire location curve for elements having one, or require the entire solid or even the bounding box to be contained?

Once you have defined both how to define the element location and its containment within the area, you can start to implement the selection process.

Whatever approach you choose, you should definitely also apply additional filtering to your algorithm, so that it will ignore all the objects you are not interested in, for instance by checking the element category, level, type and other properties.

Making use of the Revit API filters, you can retrieve the subset of all elements that may possibly be of interest to you.
Looking at those elements one by one, determine their location and check whether it is contained in your area of interest, for instance according to the following pseudo code algorithm:

```
List<Element> a = get candidate
  elements from Revit database;

foreach( Element e in a )
{
  get location L of e;

  if( L is in area )
  {
    process e;
  }
}
```