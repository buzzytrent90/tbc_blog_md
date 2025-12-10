---
post_number: "1275"
title: "Revit 2015 R2 and the Read-Write Workset API"
slug: "r2_workset_api"
author: "Jeremy Tammik"
tags: ['elements', 'references', 'revit-api', 'schedules', 'transactions', 'views', 'walls']
source_file: "1275_r2_workset_api.htm"
original_url: "https://thebuildingcoder.typepad.com/blog/1275_r2_workset_api.html"
---

### Revit 2015 R2 and the Read-Write Workset API

Finally, the Revit API provides a complete Workset API including creation and modification capabilities.

The one snag is that it is currently limited to the subscription update release [Revit 2015 R2](#3).

This issue was discussed a while back on the Revit API discussion forum, in the threads on
[R2 versus UR4](http://forums.autodesk.com/t5/revit-api/r2-vs-ur4/m-p/5382029) and
[worksets](http://forums.autodesk.com/t5/revit-api/worksets/td-p/5360275);
I searched and re-searched for them frequently enough now to finally decide to promote the gist to a main blog post topic.

#### Read-Write Workset API

[The Workset API](http://thebuildingcoder.typepad.com/blog/2011/11/read-only-workset-api.html) was a very long time coming.

Until now, it was still read-only. Now we can also create a workset, rename and activate:

- Creation:

- The new static method: Workset.Create

- Workset Modification:

- New method: WorksetTable.RenameWorkset
- New method: WorksetTable.SetActiveWorksetId

Making use of the new workset creation method in VB might look like this:

```vbnet
  Private Sub CreateWorkset(name As String)
    Using t As New Transaction(m\_Doc)
      t.Start("Create Workset")
      Workset.Create(m\_Doc, name)
      t.Commit()
    End Using
  End Sub
```

#### Revit 2015 R2

Revit 2015 R2 is a file format compatible subscription update release and was made available towards the end of 2014.

The other efficiency and productivity enhancements it provides are user features including improvements to rendering, enhancements to documentation and schedules, performance improvements, etc.:

- Platform & Architectural enhancements

- Batch wall join editing
- Schedules
- Shaft openings
- Perspective views editing tools
- Reference other view
- View updates
- Select host for tags
- Annotate stair treads and risers

- For Structural engineers

- Rebar placement
- Reverse the orientation of structural framing elements
- Snapping to model lines
- Setback reference enhancements
- User interface for structural elements

- For MEP engineers

- Panel list search enhancement
- Most recently used panel list
- Circuit default to last used
- Move circuits in panel schedule enhancement
- Pressure loss table visibility

Exceptionally, this subscription release includes the small API enhancement noted above.
Therefore, the [Revit Developer Centre](http://www.autodesk.com/developrevit) provides a separate SDK for it as well.

#### RevitLookup and Other GitHub Repository Updates

I am gradually working my way through the various Revit API GitHub repositories and updating the sample code copyright years from 2014 to 2015 in readiness for more significant updates later this year.

Yesterday, I updated
[RevitLookup](https://github.com/jeremytammik/RevitLookup),
the interactive Revit BIM database exploration tool to view and navigate element properties and relationships, and
[AdnRme](https://github.com/jeremytammik/AdnRme), the ADN sample add-in for Revit MEP HVAC and electrical.