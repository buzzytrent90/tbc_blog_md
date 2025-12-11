---
ai_optimized: true
author: Jeremy Tammik
complexity_score: 2.7
content_type: qa
optimization_date: '2025-12-11T11:44:15.768647'
original_url: https://thebuildingcoder.typepad.com/blog/1322_a360_widget_dwg_export.html
post_number: '1322'
reading_time_minutes: 3
series: general
slug: a360_widget_dwg_export
source_file: 1322_a360_widget_dwg_export.htm
tags:
- csharp
- elements
- filtering
- revit-api
- views
title: A360 Viewer Widget and Selecting a DWG Export Setup
word_count: 692
---

### A360 Viewer Widget and Selecting a DWG Export Setup

Today, let's look at the new [A360 embeddable viewer widget](#2) and [selecting a DWG export setup](#3).

#### A360 Embeddable Viewer Widget

The Autodesk View and Data API makes easy it to interactively view and explore a 3D model in the browser without any further software whatsoever.

As I just mentioned in the discussion on
[50 ways to view your model](http://the3dwebcoder.typepad.com/blog/2015/05/50-ways-to-view-your-model.html),
the new A360 Viewer Widget simplifies it further still, using a widget that can be embedded in web pages to view design files that are dragged and dropped onto it, enabling you to view them as you would in A360 inside any web page.

Try it out yourself right here; drop any model file of your choice onto this widget:

For all further details, please refer to the discussion on
[50 ways to view your model](http://the3dwebcoder.typepad.com/blog/2015/05/50-ways-to-view-your-model.html).

**Addendum:** Oops.
As explained by [Stephen](http://adndevblog.typepad.com/cloud_and_mobile/2015/05/a360-widget.html),
you might notice that the embedded widget works fine on its home page for all browsers, but drag and drop on Typepad (i.e. the widget above) apparently only works in Chrome.
The widget should work fine on your own webpage, just not in a (Typepad) blog.
We've reported the problem to the dev team.

#### Selecting a DWG Export Setup

**Question:** How can I access the names of all the ExportLayerTables through the Revit API?

I can see how to get or set one single one for DWGExportOptions but can't see where they are listed.

**Answer:** Does this documentation snippet help?

#### Remarks

This table is structured as a mapping from ExportLayerKey to ExportLayerInfo members.
The ExportLayerKey contains the identification information for the layer table: the Revit category and subcategory names.
In addition, the key contains a SpecialType member used only to represent non-Revit categories that can be assigned specific layer information on export.
The ExportLayerInfo contains the exported layer name, colour name, and layer modifiers for standard and cut representations.

The table can be accessed via direct iteration as a collection of KeyValuePairs, or by traversal of the stored keys obtained from GetKeys(), or via specific lookup of a key constructed externally. In all cases, the ExportLayerInfo returned will be a copy of the ExportLayerInfo from the table. In order to make changes to the ExportLayerInfo and use those settings during export, set the modified ExportLayerInfo back into the table using the same key.

**Response:** I don't think so – that will help me once I have an ExportLayerTable but I need a list of all the ExportLayerTables currently in the project.

Actually I'm not convinced there is such a list.

This will get me all the named Export Setups, e.g., those in the main dialogue:

![DWG export](img/dwg_export_01.png)
```csharp
  ExportLayerTable lt;

  var filter2 = new ElementClassFilter(
    typeof( ExportDWGSettings ) );

  FilteredElementCollector settings
    = new FilteredElementCollector( mDoc );

  settings = settings.WherePasses( filter2 );

  foreach( ExportDWGSettings element in settings )
  {
    DWGExportOptions options
      = element.GetDWGExportOptions();

    lt = options.GetExportLayerTable();
    if( lt != null )
    {

    }
  }
```

But once I have an ExportLayerTable it doesn't have a name, which now is making sense as once you're in the Modify DWG/DXF Export Setup dialog, you can't select the layer mappings by name:

![DWG export setup](img/dwg_export_02.png)

So I think I was looking for the wrong thing, and can make do by selecting the named ExportDWGSettings.

**Later:** I understand this now – I'd say since it was called an ExportLayerTable I was expecting something AutoCAD like (when will I ever learn :-).

I ended up with:

```csharp
  var filter2 = new ElementClassFilter(
    typeof( ExportDWGSettings ) );

  mExportDWGSettings = new FilteredElementCollector(
    mDoc );

  mExportDWGSettings = mExportDWGSettings
    .WherePasses( filter2 );

  foreach( ExportDWGSettings element
    in mExportDWGSettings )
  {
    mExportSetupComboBox.Items.Add( element.Name );
  }
```

That builds me a combo list of all the DWG Export Setups that a user can select.
That provides the layer table and all the other export settings, which actually makes a lot more sense anyway.

Many thanks to Simon Jones for this little exploration!