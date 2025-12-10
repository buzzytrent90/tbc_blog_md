---
post_number: "0929"
title: "Changing a Family Parameter from Type to Instance"
slug: "param_type_inst"
author: "Jeremy Tammik"
tags: ['family', 'parameters', 'references', 'revit-api']
source_file: "0929_param_type_inst.htm"
original_url: "https://thebuildingcoder.typepad.com/blog/0929_param_type_inst.html"
---

### Changing a Family Parameter from Type to Instance

Here is a short and simple question that just cropped up and required some digging around until my colleague Phil Xia came up with the right answer:

**Question:** Is it possible to change the type of a family parameter programmatically?

This is possible in the family editor.

I only see the IsInstance property, though, and it is read-only.

I could achieve the same effect by deleting the parameter and creating a new one.
Obviously that would require updating all the existing values, though, if there are any, to change from the old parameter to the new one.
I don't really see this as an option.
Besides, the parameter could be referenced from other parameters.

Seeing that this is possible from within the family editor, I actually don’t see any need for this property to be read-only.

I don't think changing a family parameter from type to instance and vice versa should not be so hard.

Is there some other method that allows me to achieve this?

**Answer:** Yes, sure, you are lucky, and so are we all: the following two FamilyManager methods set the given family parameter to instance or type:

- MakeInstance – Set the family parameter as an instance parameter.- MakeType – Set the family parameter as a type parameter.

I hope this will save you from repeating the same digging all over again :-)

Many thanks to Phil for explicitly pointing this out!