---
post_number: "0745"
title: "GetElement method and Get Element Type"
slug: "get_element_type"
author: "Jeremy Tammik"
tags: ['csharp', 'elements', 'parameters', 'revit-api', 'views']
source_file: "0745_get_element_type.htm"
original_url: "https://thebuildingcoder.typepad.com/blog/0745_get_element_type.html"
---

### GetElement method and Get Element Type

In the far distant past, the Revit API boasted a property on the Element class that gave direct access to the element type.
It was declared obsolete, and a new way to access the element type had to be implemented, and we discussed
[two years back](http://thebuildingcoder.typepad.com/blog/2010/08/get-element-type.html).
This can still raise an issue even today:

**Question:** I have an old plug-in that I am trying to update to Revit 2013. The code uses a property (element.ObjectType) that has been made obsolete and that is no longer in the 2013 API. The obsolete call uses objElement.ObjectType.Parameters. The suggested replacement (Element.GetElementType) does not have a Parameters property. So, my question is, in Revit 2013, how do I get the Type parameters from an element in a project?
Here is a section of my code containing this obsolete call:
```csharp
  // Repeat the loop for the Type parameters
  // on the element and assign values
  foreach( Parameter param in
    objElement.ObjectType.Parameters )
  {
    // . . .
  }
```

**Answer:** The Element ObjectType property was actually declared deprecated quite a few releases back, and still remained dangling until a whole host of obsolete calls were cleaned up in the Revit 2013 API.

The replacement is simple:

Use GetTypeId to obtain the element id of the type of your element, and then use the document GetElement method to access the type object itself from its id. Here is the code to achieve that:
```csharp
  ElementId id = e.GetTypeId();

  ElementType type = doc.GetElement( id )
    as ElementType;
```

I discussed this
[two years back](http://thebuildingcoder.typepad.com/blog/2010/08/get-element-type.html).
Since then, another change occurred as well, which Saikat mentioned in his
[macro migration notes](http://thebuildingcoder.typepad.com/blog/2012/04/migrating-vsta-macros-to-sharpdevelop.html):

The Document.Element property taking an element id argument has been converted to a method named GetElement in Revit 2013.

The obsolete Element property was converted to a method with a "get\_" prefix in C#, so it had to be accessed by calling get\_Element, which sometimes
[caused a bit of confusion](http://thebuildingcoder.typepad.com/blog/2009/09/document-elements.html).

Anyway, translated to your example code snippet, the new access might look like this in Revit 2013:
```csharp
  // Repeat the loop for the element
  // type parameters and assign values

  Element elementType = doc.GetElement(
    e.GetTypeId() );

  foreach (Parameter param in elementType.Parameters)
```

#### Element Display Overrides and Visibility Hierarchy

Revit provides a large number of ways to affect and override the display of an element, and it may not always be clear which method has the final say.

We had a look at one of these ways when describing how to
[change element colour using ProjColorOverrideByElement](http://thebuildingcoder.typepad.com/blog/2011/03/change-element-colour.html).

The Revit Clinic provides a useful overview of the
[Revit visibility hierarchy](http://revitclinic.typepad.com/my_weblog/2012/02/revit-visibility-hierarchy.html) which
helps understand the various possibilites and how they relate to each other.

#### Happy Easter!

I wish a Happy Easter to everybody celebrating this holiday, and a good time to everybody else as well :-)

![Easter bunny postcard](img/Easter_Bunny_Postcard_1907.jpg)

Good luck and much success with your Revit add-in development efforts and all other important aspects of life, such as love, peace, and happiness for all beings :-)