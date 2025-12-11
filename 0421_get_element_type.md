---
ai_optimized: true
author: Jeremy Tammik
complexity_score: 3.5
content_type: qa
optimization_date: '2025-12-11T11:44:13.921122'
original_url: https://thebuildingcoder.typepad.com/blog/0421_get_element_type.html
post_number: '0421'
reading_time_minutes: 2
series: elements
slug: get_element_type
source_file: 0421_get_element_type.htm
tags:
- csharp
- elements
- family
- parameters
- revit-api
- sheets
- views
title: Get Element Type
word_count: 430
---

### Get Element Type

In the Revit 2011 API, the Symbol class was been renamed to ElementType and some other changes were made to clarify the usage of element types in Revit, which were previously referred to as both types and symbols.
Some remnants of the previous naming conventions still exists, such as the FamilySymbol class and the FamilyInstance.Symbol property, but the long-term intention is to move toward calling these things element types instead of symbols.

This renaming and restructuring has causes some confusion here and there in porting existing applications, giving rise to questions such as the following:

**Question:** I have been updating my Revit 2010 API code to the 2011 version and realized that the Element.ObjectType property no longer exists.

I want to retrieve the AssemblyCode value from the element type properties in the Revit project.
Here is the old code for the Revit 2010 API:
```csharp
  string assembly\_code
    = e.ObjectType.get\_Parameter(
      BuiltInParameter.UNIFORMAT\_CODE )
      .AsString();
```

How can I rewrite this code for Revit 2011?

**Answer:** The first place to look in cases like this is the What's New section of the Revit API help file RevitAPI.chm, which includes the following section:

#### Replacement for Symbol and properties that access types

The Symbol class has been renamed to ElementType.

The properties that access types from Elements have been replaced:

- Old property: Element.ObjectType
  - New interfaces: Element.GetTypeId(), Element.ChangeTypeId()- Notes: ChangeTypeId() returns the id of a new element; in rare cases, changing the element type will result in deletion of the current element. Static versions which operate on sets of elements are also supplied.- Old property: Element.SimilarObjectTypes
    - New interfaces: Element.GetValidTypes()

So you can simply use GetTypeId and then open the associated ElementType object to access the parameter.

Some samples of using these new methods are given in the discussions on

- [Getting the type id](http://thebuildingcoder.typepad.com/blog/2010/05/get-type-id-and-preview-image.html)- [Determining sheet size](http://thebuildingcoder.typepad.com/blog/2010/05/determine-sheet-size.html)- [RevitWebcam](http://thebuildingcoder.typepad.com/blog/2010/06/display-webcam-image-on-building-element-face.html)- [Setting a tag type](http://thebuildingcoder.typepad.com/blog/2010/06/set-tag-type.html)- [Populating a data grid view](http://thebuildingcoder.typepad.com/blog/2010/07/populating-a-data-grid-view.html)

The code that you show above can thus be rewritten as follows for the Revit 2011 API:
```csharp
  ElementId id = e.GetTypeId();

  ElementType type = m\_doc.get\_Element( id )
    as ElementType;

  string AssemblyCode = type.get\_Parameter(
    BuiltInParameter.UNIFORMAT\_CODE )
      .AsString();
```