---
ai_optimized: true
author: Jeremy Tammik
complexity_score: 2.9
content_type: qa
optimization_date: '2025-12-11T11:44:13.299954'
original_url: https://thebuildingcoder.typepad.com/blog/0059_pickone.html
post_number: 0059
reading_time_minutes: 1
series: general
slug: pickone
source_file: 0059_pickone.htm
tags:
- csharp
- elements
- revit-api
- selection
- windows
title: PickOne
word_count: 277
---

### PickOne

Here is a small note on how to make use of the PickOne Revit element selection method.

For any source code example of how to use any Revit API method, one obvious to start looking is always the global solution SDKSamples2009.sln in the SDK Samples folder.

PickOne is used in the Revit SDK samples VisibilityControl and PowerCircuit. You can see this by opening SDKSamples2009.sln and searching globally for PickOne(). This returns a list of hits, e.g.:

```
C:\SDK\Samples\VisibilityControl\CS\VisibilityCtrl.cs(138):
m_document.Selection.PickOne();

C:\SDK\Samples\PowerCircuit\CS\CircuitOperationData.cs(518):
if (!m_revitDoc.Selection.PickOne())
```

Here is an excerpt from the VisibilityControl sample, slightly cleaned up and reformatted for readability, and with some numbered comments added in bold:

```csharp
public void Isolate()
{
  // 1. clear selection:
  m\_document.Selection.Elements.Clear();
  switch (m\_isolateMode)
  {
    case IsolateMode.PickOne:
      // 2. pick one element:
      m\_document.Selection.PickOne();
      break;

    case IsolateMode.WindowSelect:
      m\_document.Selection.WindowSelect();
      break;
  }

  // 3. retrieve selected elements:
  Autodesk.Revit.ElementSet elements
    = m\_document.Selection.Elements;

  // hide all categories elements
  foreach( Category cat in
    m\_document.Settings.Categories )
  {
    SetVisibility(false, cat.Name);
  }

  // 4. make use of selected elements:
  // set the selection elements visibility
  foreach( Element element in elements )
  {
    Category cat = element.Category;
    if( null != cat
      && !string.IsNullOrEmpty( cat.Name ) )
    {
      SetVisibility( true, cat.Name );
    }
  }
}
```

The PickOne method manipulates the document's current selection set, accessible through the document Selection property.
The bold numbered comments correspond to the critical steps:

1. Clear the current selection.
2. Execute the PickOne method.
3. Access the updated selection set.
4. Do something with the selected elements.

Hopefully this clarifies the use of the PickOne method.