---
ai_optimized: true
author: Jeremy Tammik
complexity_score: 4.2
content_type: qa
optimization_date: '2025-12-11T11:44:15.485258'
original_url: https://thebuildingcoder.typepad.com/blog/1192_view_template_discipl.html
post_number: '1192'
reading_time_minutes: 3
series: views
slug: view_template_discipl
source_file: 1192_view_template_discipl.htm
tags:
- csharp
- elements
- filtering
- levels
- revit-api
- views
title: Accessing Discipline and Duplicating View Template
word_count: 585
---

### Accessing Discipline and Duplicating View Template

I am back again in the land of the living... working, blogging.

I'll jump right in with two recent cases related to disciplines and views:

- [Accessing view and project browser disciplines](#2)
- [Duplicating a view template](#3)

#### Accessing View and Project Browser Disciplines

**Question:** When changing the Revit project browser organization in Revit to 'Discipline', the project browser shows different subfolders depending on the document template.

For example, in a project created from an architectural template, it shows ‘Architectural':

![Project browser discipline label Architecture](img/discipline_1.png)

If the project was created from a structural template, it shows ‘Structural':

![Project browser discipline label Structural](img/discipline_2.png)

Is it possible to retrieve this discipline name programmatically?

**Answer:** My understanding is that this is for organization.
If you choose 'discipline', it shows discipline as a top categorization.

If you change the discipline of a view, say from 'Architecture' to 'Structure' for an elevation, it will show up under 'Structure':

![View discipline](img/discipline_3.png)

This depends on the current setting of the View.Discipline property providing a ViewDiscipline enumeration value.

**Response:** Yes, View.Discipline returns the discipline for a given view.

The returned discipline label is English text, even in a localised (e.g., Japanese) version of Revit.

I implemented this test code to list the disciplines of all views in the current document:

```csharp
  FilteredElementCollector collector
    = new FilteredElementCollector( doc )
      .OfClass( typeof( View ) );

  string s = "";

  foreach( View v in collector )
  {
    if( ( v.ViewType == ViewType.Elevation ||
      v.ViewType == ViewType.FloorPlan ||
      v.ViewType == ViewType.ThreeD ) &&
      v.CanBePrinted )
    {
      if( !s.Contains( v.Discipline.ToString() ) )
      {
        s = s + "\n" + v.Discipline.ToString();
      }
    }
  }
  TaskDialog.Show( "専門分野名", s );
```

Is there any other more direct way to retrieve the discipline label displayed in the project browser?

**Answer:** The following code retrieves the string '構造' (Structural) for the current view (Level 1, in this case) in the structural discipline:

![Project browser discipline label](img/discipline_4.jpeg)
```csharp
  BrowserOrganization bo = BrowserOrganization
    .GetCurrentBrowserOrganizationForViews( doc );

  IList<FolderItemInfo> folderItems
    = bo.GetFolderItems( doc.ActiveView.Id );

  foreach( FolderItemInfo folder in folderItems )
  {
    string name = folder.Name;
  }
```

Please note that the
[BrowserOrganization class](http://thebuildingcoder.typepad.com/blog/2014/04/whats-new-in-the-revit-2015-api.html#3.13) was
added in Revit 2015.

#### Duplicating a View Template

Madhu Das raised the issue of duplicating a view template in a
[comment](http://thebuildingcoder.typepad.com/blog/2010/04/filter-for-views-and-istemplate-predicate.html?cid=6a00e553e16897883301a73e03bcff970d#comment-6a00e553e16897883301a73e03bcff970d) on
the discussion on
[filtering for views and the IsTemplate predicate](http://thebuildingcoder.typepad.com/blog/2010/04/filter-for-views-and-istemplate-predicate.html):

**Question:** How can I programmatically duplicate a view template?

The following code returns False if v1 is a view template:

```csharp
v1.CanViewBeDuplicated(ViewDuplicateOption.Duplicate)
```

**Answer:** We currently have an open wish list item CF-1509 [API wish: create view template -- 09693106] for programmatic creation of a view template.

For duplication of an existing one, you may be able to make use of the copy and paste API.

Look at the numerous
[copy and paste API usage examples](http://thebuildingcoder.typepad.com/blog/2013/05/copy-and-paste-api-applications-and-modeless-assertion.html).

**Response:** Thank you very much for the help.

It is possible by using the Copy and Paste API.

I successfully tried the following code:

```csharp
  Dim elementIds As New List(Of ElementId)

  elementIds.Add(v.Id)

  copiedIds = ElementTransformUtils.CopyElements( \_
    activeDoc, elementIds, activeDoc, \_
    Transform.Identity, Options)

  vNew = activeDoc.GetElement(copiedIds(0))

  vNew.Name = "New View Template"
```