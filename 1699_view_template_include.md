---
ai_optimized: true
author: Jeremy Tammik
complexity_score: 2.1
content_type: qa
optimization_date: '2025-12-11T11:44:16.550631'
original_url: https://thebuildingcoder.typepad.com/blog/1699_view_template_include.html
post_number: '1699'
reading_time_minutes: 1
series: views
slug: view_template_include
source_file: 1699_view_template_include.md
tags:
- elements
- parameters
- revit-api
- sheets
- views
title: View Template Include
word_count: 193
---

### View Template Include Setting
A quick note to highlight a solution shared by Teocomi to solve a longstanding question in
the [Revit API discussion forum](http://forums.autodesk.com/t5/revit-api-forum/bd-p/160) thread
on [view template 'include'](http://forums.autodesk.com/t5/revit-api/view-template-quot-include-quot/m-p/5410347):
\*\*Question:\*\* Does the Revit API provide any access to the view template 'include' settings defined by the check boxes in this form?
![View template include checkboxes](img/view_template_include_check_boxes.jpg)
\*\*Answer:\*\* I can get the 'includes' via `viewTemplate.GetNonControlledTemplateParameterIds`.
The method returns a list of parameter ids, and you can then use `viewTemplate.Parameters` to map them.
The same also works for setting them, cf. the following example:
```csharp
// Create a list so that I can use linq
var viewparams = new List();
foreach( Parameter p in viewTemplate.Parameters )
viewparams.Add( p );
// Get parameters by name (safety checks needed)
var modelOverrideParam = viewparams
.Where( p
=> p.Definition.Name == "V/G Overrides Model" )
.First();
var viewScaleParam = viewparams
.Where( p
=> p.Definition.Name == "View Scale" )
.First();
// Set includes
viewTemplate.SetNonControlledTemplateParameterIds(
new List {
modelOverrideParam.Id, viewScaleParam.Id } );
```
Thank you, Teocomi, for sharing this!