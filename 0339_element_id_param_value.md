---
ai_optimized: true
author: Jeremy Tammik
complexity_score: 3.2
content_type: qa
optimization_date: '2025-12-11T11:44:13.771969'
original_url: https://thebuildingcoder.typepad.com/blog/0339_element_id_param_value.html
post_number: 0339
reading_time_minutes: 2
series: parameters
slug: element_id_param_value
source_file: 0339_element_id_param_value.htm
tags:
- elements
- filtering
- parameters
- revit-api
- selection
title: ElementId Parameter Value
word_count: 440
---

### ElementId Parameter Value

This week I should be recording the
[VBA migration guide DevTV](http://through-the-interface.typepad.com/through_the_interface/2010/02/updated-devtv-autocad-vba-to-vbnet-migration-basics.html)
in German, and once again I cannot keep away from blogging instead.
This is just a short and sweet little support case that I thought worth while describing for the general public:

**Question:** I am interested in the possible values of some specific parameters whose storage type is ElementId.
One of them is the 'Start Connection' parameter on beams.
If I want to set that parameter to something, I need the appropriate element id.
Are there any enumerations like the built-in parameters that would help me determine the element id I need to use, for example, to set the Start Connection to 'Cantilever Moment Connection'?

I tried determining the proper value to use by setting it manually through the user interface and then retrieving it through the API.
That returned an element id like 233422, which makes me think that there should be a cleaner way.

**Answer:** If the element id is positive, I would assume that it refers to a real element in the Revit database. Element ids that do not refer to real existing elements in the database are generally negative.

To test this in the user interface and
[RevitLookup](http://thebuildingcoder.typepad.com/blog/2010/04/revitlookup-and-textnote-alignment.html),
you can simply go to Manage > Inquiry > Select by ID, type in the element id you have obtained, e.g. 233422, and examine its type and other properties in RevitLookup by selecting Add-Ins > Revit Lookup > Snoop Current Selection...

I searched a randomly selected Revit Structure model on my system for an element named 'Cantilever Moment Connection' and could not find any, but there are some other interesting elements that seem to me like they may be similar to what you are looking for (copy to an editor if the lines are truncated):

```
Id=137086; Class=StructuralConnectionType; Name=Moment Frame;
Id=137087; Class=StructuralConnectionType; Name=Cantilever Moment;
Id=137088; Class=StructuralConnectionType; Name=Shear Column Connection;
Id=137089; Class=StructuralConnectionType; Name=Moment Column Connection;
Id=137090; Class=StructuralConnectionType; Name=Base Plate Symbol;
```

These are normal Revit elements of the class or System.Type StructuralConnectionType, which is derived from the Revit Element class.
Their Category property is null, so we cannot use that to identify them.

I suspect that your 'Cantilever Moment Connection' object may be such an element as well.

If so, you can use the StructuralConnectionType class to filter for them, and the element Name property to determine each such object's name.