---
ai_optimized: true
author: Jeremy Tammik
complexity_score: 2.7
content_type: code_example
optimization_date: '2025-12-11T11:44:13.625233'
original_url: https://thebuildingcoder.typepad.com/blog/0252_new_project_doc.html
post_number: '0252'
reading_time_minutes: 1
series: general
slug: new_project_doc
source_file: 0252_new_project_doc.htm
tags:
- csharp
- elements
- revit-api
title: New Project Document
word_count: 167
---

### New Project Document

A short and simple
[question](http://thebuildingcoder.typepad.com/blog/2009/09/detail-lines.html?cid=6a00e553e168978833012875c7c99d970c#comment-6a00e553e168978833012875c7c99d970c) from Nadim on the NewProjectDocument method:

**Question:** I am using
```csharp
Document doc = app.NewProjectDocument(
string templateFile );
```

Revit shows that it is loading the template as it does usually when it creates a new document; however, it does not really create the new document.

Any idea why?

**Answer:** I implemented a sample external command CmdNewProjectDoc to test it for you and it works fine for me:
```csharp
const string \_template\_file\_path
  = "C:/Documents and Settings/All Users"
  + "/Application Data/Autodesk/RAC 2010"
  + "/Metric Templates/DefaultMetric.rte";

public IExternalCommand.Result Execute(
  ExternalCommandData commandData,
  ref string message,
  ElementSet elements )
{
  Application app = commandData.Application;

  Document doc = app.NewProjectDocument(
    \_template\_file\_path );

  doc.SaveAs( "C:/tmp/new\_project.rvt" );

  return CmdResult.Failed;
}
```

After running the command, the new project file is created and saved in the expected location:

```
2009-11-23  12:26  2,056,192 new_project.rvt
```