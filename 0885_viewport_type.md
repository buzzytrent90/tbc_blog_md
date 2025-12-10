---
post_number: "0885"
title: "Changing Viewport Type"
slug: "viewport_type"
author: "Jeremy Tammik"
tags: ['elements', 'family', 'levels', 'parameters', 'revit-api', 'schedules', 'sheets', 'vbnet', 'views', 'walls']
source_file: "0885_viewport_type.htm"
original_url: "https://thebuildingcoder.typepad.com/blog/0885_viewport_type.html"
---

### Changing Viewport Type

Here is a question on setting the type of an element by using the built-in ELEM\_FAMILY\_AND\_TYPE\_PARAM parameter value, explored by Bettina Zimmermann of
[NTI Cadcenter A/S](http://www.nti.dk):

**Question:** I'm inserting viewports on sheets and I'd like to change the viewport type to my own defined type.

How can I this programmatically?

By default, the API call creates a new viewport with type "Title w Line".
I'd like to change that to e.g. my own type "Test".

Here is my own self-defined viewport type "Test":

![Change viewport type](img/change_vp_type_1.png)

I created it manually by pressing Duplicate.

The API method to create viewports does not support any viewport type parameter:

```
  Autodesk.Revit.DB.Viewport.Create(
    Document, viewSheet.Id, View.Id, zero )
```

I hope the viewport type can be changed in some other way.

**Answer:** When dealing with views and viewports, one of the first places to take a look is Steve Mycynek's AU class
[CP3133 Using the Revit Schedule and View APIs](http://thebuildingcoder.typepad.com/blog/2012/11/au-classes-on-the-view-mep-and-link-apis.html#2),
which demonstrates just about everything you can do with the Revit View API.

Further, I would suggest that you explore the elements of interest in depth using RevitLookup.

The Element.GetTypeId method may provide read access to the data you seek.
Unfortunately, of course, it is read-only.
Maybe you can find some parameter that also provides access to this data?

**Response:** I did indeed find a parameter, well hidden in an unexpected location.

My first thought when searching for it is that the viewport is a system family, just like a wall, and walls allow you to change their type by setting
[wall.WallType = newWallType](http://thebuildingcoder.typepad.com/blog/2011/02/system-family-creation.html).

Walls also expect a type argument when a new wall is created:

```
  DB.Wall.Create( Document, Line, WallType.Id,
    Level.Id, 11, 0, False, IsStructural );
```

However, as said, the Viewport Create method does not take any type argument:

```
  Autodesk.Revit.DB.Viewport.Create(
    Document, viewSheet.Id, View.Id, zero );
```

In spite of this, the parameter I found that I can use for this is BuiltInParameter.ELEM\_FAMILY\_AND\_TYPE\_PARAM.

I created a VB.NET sample add-in
[ChangeViewPortType](zip/ChangeViewPortType.zip) to
demonstrate its use.
Here is how to use it:

- Set a sheet view active with two viewports on it – could be two floor plans on a sheet.- Change one of the viewports to another type – see below.- Select both viewports and run sample project – the last selected viewport will be changed to have the same type as the first.

Here is a sheet with a viewport.

![Sheet with a viewport](img/change_vp_type_2.png)

The default type of a newly created viewport is 'Title w Line';
I made a new type called 'Test' that has no 'Title' and 'Extension Line'.
'Title' and 'Extension Line' is the circle with a line and some text:

I made the type by clicking 'Edit Type' and duplicate.

![Edit type and duplicate](img/change_vp_type_3.png)

Many thanks to Bettina for her exploration and thorough documentation!

**Addendum:** As pointed out below by Alexander Buschmann, the Revit API also provides the more user-friendly Element.ChangeTypeId method to directly change the type of any element having a type, with no need to go through the parameter to access it.
Thanks, Alexander, for adding that!