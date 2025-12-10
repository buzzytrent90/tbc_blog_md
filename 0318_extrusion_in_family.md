---
post_number: "0318"
title: "Access Extrusions in a Family"
slug: "extrusion_in_family"
author: "Jeremy Tammik"
tags: ['elements', 'family', 'geometry', 'parameters', 'revit-api', 'schedules']
source_file: "0318_extrusion_in_family.htm"
original_url: "https://thebuildingcoder.typepad.com/blog/0318_extrusion_in_family.html"
---

### Access Extrusions in a Family

I am very busy right now preparing for the next Revit API training class which will be held in Warsaw, Poland, just after Easter.
For a full list of upcoming Revit API classes, please refer to the ADN
[API training schedule](http://www.adskconsulting.com/adn/cs/api_course_sched.php).

Meanwhile, here is a short and sweet little issue which clarifies the relationship between project elements such as family instances and the geometric primitives contained within them, which are family elements:

**Question:** I am using families of category Curtain Panel that have extrusions in them.
I can access all the panels from the Revit model.
Once I have the panel, how can I retrieve all the extrusions in the family and access the extrusion parameters?

**Answer:** The panel instances that you insert into your model live in the project document.
The extrusion elements that are used to define their geometry live in the family document, i.e. in a completely different space.
To access the extrusion elements, you have to open the family document and read the elements from there.
Here are some previous topics which should help understand the situation better:

- [The Revit Family API](http://thebuildingcoder.typepad.com/blog/2009/08/the-revit-family-api.html).
- [The Revit Family Creation API Labs](http://thebuildingcoder.typepad.com/blog/2009/08/the-revit-family-api.html).
- [Removing all geometry from a family](http://thebuildingcoder.typepad.com/blog/2009/09/remove-all-geometry-from-a-family.html).
- [Creating a nested family](http://thebuildingcoder.typepad.com/blog/2009/11/nested-family.html).

One issue that you need to tackle in this context is how to open the family file.
If it is already opened by Revit, it will be listed in the application Documents collection.
Otherwise, you need to determine the family document path name, as explained in
[accessing linked file geometry](http://thebuildingcoder.typepad.com/blog/2009/04/access-to-linked-file-geometry.html).
Finally, you should be able to adapt the example that
[opens all family documents and searches them for certain elements](http://thebuildingcoder.typepad.com/blog/2009/05/imports-in-families.html) to
search for the extrusions instead.