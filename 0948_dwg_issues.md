---
ai_optimized: true
author: Jeremy Tammik
complexity_score: 3.2
content_type: qa
optimization_date: '2025-12-11T11:44:14.971003'
original_url: https://thebuildingcoder.typepad.com/blog/0948_dwg_issues.html
post_number: 0948
reading_time_minutes: 6
series: general
slug: dwg_issues
source_file: 0948_dwg_issues.htm
tags:
- elements
- geometry
- parameters
- revit-api
- rooms
- views
title: DWG Issues and Various Other Updates
word_count: 1123
---

### DWG Issues and Various Other Updates

One of the highlights mentioned in the
[overview of the Revit 2014 API](http://thebuildingcoder.typepad.com/blog/2013/03/revit-2014-api-and-room-plan-view-boundary-polygon-loops.html#2) is
the FreeForm element API that enables modification of solid geometry imported from DWG or SAT.

Several other new DWG and DXF related features include import of DXF markup, import and link of SAT and SketchUp, and access to the DWG, IFC and DGN layer, linetype, lineweight, font and pattern tables.

One result of these enhancements is the possibility to access 3D coordinates in DWG import for the first time.

Besides looking at that, I'll also point out a few other issues further down:

- [Differentiate Label and Text in DWG export](#2)
- [Element Ids in extensible storage](#3)
- [All-zero language codes in the Revit product GUID](#4)
- [Another dockable panel sample](#5)

#### Access to 3D Coordinates in DWG Import

One DWG import aspect that has often been requested by developers in the past is the ability to explode 3D DWG files and obtain the resulting coordinates in the Revit project.

Revit 2014 families now provide the ability to explode and retain such 3D geometry.

Daniel Gijsbers discusses making use of this functionality to
[correlate Civil 3D alignments with Revit Structure](http://danielgijsbers.blogspot.de/2013/04/revit-2014-3d-dwg-in-families.html).

In Revit 2013, one alternative way to achieve this was to implement an AutoCAD add-in that analyses the geometry and exports the 3D coordinate data.
A separate Revit add-in could read the coordinate data and correlate it with the Revit geometry.

#### Differentiate Label and Text in DWG Export

On a related topic, how can I differentiate between Revit text and label elements after they have been exported to DWG?

Revit Export to DWG results in Labels becoming Text rather than Attributes, as most users might prefer.

It would be interesting to have some flag or embedded information in the exported text that could be used in an AutoCAD application to post-process the information received from Revit and restore the attribute information.

Happily, the exported Labels have Extended Entity Data like this attached:

```
  TypedValue 0 - type: 1001, value: REVIT
  TypedValue 1 - type: 1002, value: {
  TypedValue 2 - type: 1070, value: 1
  TypedValue 3 - type: 1000, value: 245287
  TypedValue 4 - type: 1002, value: }
  TypedValue 5 - type: 1002, value: {
  TypedValue 6 - type: 1070, value: 5
  TypedValue 7 - type: 1000, value: 243658
  TypedValue 8 - type: 1002, value: }
  TypedValue 9 - type: 1002, value: {
  TypedValue 10 - type: 1070, value: 2
  TypedValue 11 - type: 1000, value: -2000280
  TypedValue 12 - type: 1002, value: }
```

The xdata on Text elements, on the other hand, look like this instead:

```
  TypedValue 0 - type: 1001, value: REVIT
  TypedValue 1 - type: 1002, value: {
  TypedValue 2 - type: 1070, value: 1
  TypedValue 3 - type: 1000, value: 261323
  TypedValue 4 - type: 1002, value: }
  TypedValue 5 - type: 1002, value: {
  TypedValue 6 - type: 1070, value: 5
  TypedValue 7 - type: 1000, value: 209618
  TypedValue 8 - type: 1002, value: }
  TypedValue 9 - type: 1002, value: {
  TypedValue 10 - type: 1070, value: 2
  TypedValue 11 - type: 1000, value: -2000300
  TypedValue 12 - type: 1002, value: }
```

The differing TypedValue 11 is consistently -2000300 for a text and -2000280 for a label element.

What does this mean?

Well, is actually quite easy.

On seeing these large negative numbers in this specific range, an experienced Revit developer will quickly suspect built-in category or parameter enumeration values.

You can check what they actually represent in the Visual Studio debugger, by jumping to the definition of these enumerations and searching for the specific values.

Looking back at an ancient blog post on the
[DWG and DXF export Xdata specification](http://thebuildingcoder.typepad.com/blog/2010/08/dwg-and-dxf-export-xdata-specification.html)
confirms that these numbers do indeed represent the built-in category of the source element and thus can be used to distinguish the two.

Many thanks to Dale Bartlett, CAD-BIM System Manager at
[Atkins (Oman)](http://www.atkins-me.com), for suggesting this topic.

Dale adds:

It will be interesting to see if this generates any comments.

As far as the AutoCAD geometry export, the purpose was to build a 3D Space Frame from an engineer’s analysis drawing.
I did in fact write an XML file from AutoCAD of the geometry and imported it into Revit to build the frame.

It took an hour to build 20,000 elements, which was about a year less than doing it manually!

I’ll revise it with 2014 to make it a one-step-inside-Revit process.

#### Element Ids in Extensible Storage

**Question:** I need to store ElementId data using schemas within my Revit add-in.

However, the API documentation states that "ids are subject to change during an Autodesk Revit session and as such should not be retained and used across repeated calls to external commands."

My add-in will be used in a worksharing environment.
My testing shows that if I try to save the integer value of an element id, and it changes, my saved id value will be invalid.
However, if I store the actual ElementId in a schema I do not have this problem.

Is saving the ElementId using a Schema safe?
If not, do you have any suggestions or examples of how to safely store an identifier to elements in a schema?

**Answer:** Yes, it is completely safe to store ElementId data in extensible storage for the purposes you describe.

The statement you quote refers to the fact that the integer value of an element id will change between sessions.
Basically, you can think of it as a pointer.
When stored as an element id in extensible storage, the actual links between the associated elements are stored, and the resulting integer values are recalculated from scratch in every new session, maintaining the links intact.
The integer value of an element id may be changed by worksharing updates as well, and these changes are also automatically handled for element ids stored in extensible storage, so it is perfectly safe to save them there.

#### All-zero Language Codes in the Revit Product GUID

Watch out for all-zero language code identifiers in the Revit Product GUID due to the new Multiple Language Interface, or MUI, used in Revit 2014.

For details, please refer to the
[Revit Product 2014 GUID addendum](http://thebuildingcoder.typepad.com/blog/2013/04/perpetual-guid-algorithm-and-revit-2014-product-guids.html#5).

#### Another Dockable Panel Sample

Guy Robinson published another dockable dialogue sample which does not require a command to be registered.

For details, please refer to the
[simpler dockable panel addendum](http://thebuildingcoder.typepad.com/blog/2013/05/a-simpler-dockable-panel-sample.html#2).