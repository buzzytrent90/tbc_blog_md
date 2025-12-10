---
post_number: "0338"
title: "User Visible Enumeration Value Labels"
slug: "label_utils"
author: "Jeremy Tammik"
tags: ['elements', 'geometry', 'parameters', 'revit-api', 'windows']
source_file: "0338_label_utils.htm"
original_url: "https://thebuildingcoder.typepad.com/blog/0338_label_utils.html"
---

### User Visible Enumeration Value Labels

The first Revit 2011 Programming Introduction class in Warsaw last week went well.
In a way, it was quite an extreme event, because on one hand the training material is still new and under development for updating to the new version, and on the other, we had 40 participants, which is the largest group I have ever led single-handedly through a hands-on programming training.
So I was pretty busy and very happy that all were satisfied.

Getting back into everyday life again, here is an answer by my colleague Joe Ye to an issue that has come up a few times in the past and now has a very satisfying new resolution:

**Question:** How can I determine the element property group names as they are displayed in the 'Element Properties' window?
When iterating through parameters of an element, we can only access the BuiltInParameterGroup of the parameter definition, such as PG\_CONSTRAINTS, PG\_MECHANICAL or PG\_GEOMETRY.
Is there a way, no matter how complicated, to obtain a true group name?
I need a general, language independent, method.

**Answer:** In previous version, there was no way to achieve what you are asking for, but it is resolved now in the Revit 2011 release by the introduction of the LabelUtils class, which is described as follows in the Revit API help file What's New section:

#### Labels matching commonly used enumerated type values

The new class LabelUtils provides methods to obtain the user-visible label corresponding to certain enum values.
These routines obtain the label corresponding to the name in the current Revit language.
Support is offered for:

- BuiltInParameter- BuiltInParameterGroup- DisplayUnitType- gbXMLBuildingType- ParameterType- UnitType

Many thanks to Joe for handling this case!