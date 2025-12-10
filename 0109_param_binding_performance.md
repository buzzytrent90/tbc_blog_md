---
post_number: "0109"
title: "Parameter Binding Performance"
slug: "param_binding_performance"
author: "Jeremy Tammik"
tags: ['csharp', 'parameters', 'revit-api']
source_file: "0109_param_binding_performance.htm"
original_url: "https://thebuildingcoder.typepad.com/blog/0109_param_binding_performance.html"
---

### Parameter Binding Performance

Here is a short but important question that has come up repeatedly concerning the performance of parameter bindings.

**Question:**
There is a big problem in the early versions of Revit 2009 with the performance of the Document ParameterBindings property, for example in the last line of the following code:

```csharp
Application app = commandData.Application;
Document doc = app.ActiveDocument;
BindingMap bindingMap = doc.ParameterBindings;
```

The last statement takes a huge amount of time to execute, in the order of minutes.
Are there any known solutions for this?

**Answer:**
Good news: this performance delay was analysed and reported by Saikat Bhattacharya and fixed in the
[Revit 2009 Web Update 3](http://usa.autodesk.com/adsk/servlet/item?siteID=123112&id=11017599).
The issue is mentioned in the associated PDF file containing the
[list of enhancements](http://revit.downloads.autodesk.com/download/2009/RAC2009_WU3_List.pdf):

#### API Enhancements

- Parameter binding performance has been improved.
- The JoinType method has been implemented for the LocationCurve of structural members.
- Mullion LocationCurves are now accessible through the API.
- External programs may now suppress VSTA startup warning messages.
- The built-in parameter MATERIAL\_PARAM\_TRANSPARENCY now returns the correct set value.

So all you need to do is install the updated version of Revit 2009 and the problem disappears.
Many thanks to Saikat for pointing this out.