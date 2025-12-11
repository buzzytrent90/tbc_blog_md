---
ai_optimized: true
author: Jeremy Tammik
complexity_score: 2.8
content_type: qa
optimization_date: '2025-12-11T11:44:14.408772'
original_url: https://thebuildingcoder.typepad.com/blog/0693_rest_copy_model.html
post_number: 0693
reading_time_minutes: 2
series: general
slug: rest_copy_model
source_file: 0693_rest_copy_model.htm
tags:
- references
- revit-api
- views
title: Copy a Model from a Revit Server
word_count: 400
---

### Copy a Model from a Revit Server

Yesterday evening we had dinner together with developer partners at
[Heaven 23](http://www.heaven23.se) again,
the same place we visited
[last year](http://thebuildingcoder.typepad.com/blog/2010/12/vsta-to-stay-and-updater-to-go.html),
with its unique reflection-accessible cloud.

Today we are holding a DevLab here in the Autodesk office in Gothenburg in Sweden, providing an opportunity for developers to come visit us with their day-to-day work in progress and current issues.

Meanwhile, here is a question related to the
[Revit Server and its API](http://thebuildingcoder.typepad.com/blog/2011/11/revit-server-rest-api.html):

**Question:** How can I copy a Revit file out of Revit server 2012 to a network location?
I need something similar to the
[RevitServerViewer sample](http://thebuildingcoder.typepad.com/blog/2011/11/revit-server-rest-api.html),
but instead of just relaying the information of the file I would like to get the actual file definition, assign it to an object in .NET, and use standard I/O operations to copy it to multiple other servers.

**Answer:** Look at the Application.CopyModel method.

```
  public void CopyModel(
    ModelPath sourceModelPath,
    string destFilePath,
    bool overwrite )
```

It copies an existing model to a new file, which can then be copied elsewhere as needed.

It takes the path of the file and does not require opening the file.
It accepts the path of the server (or file) based source file and the destination path, besides a Boolean specifying whether you wish to overwrite the destination file or not.

If you have to copy multiple files, you can call this method multiple times specifying different source and destination information for each file.

This method is a Revit API method, so it does require Revit to be up and running and must be called in a valid Revit API context.
As you can see in the "Revit Server REST API Reference.pdf" in the Revit Server SDK, the REST API supports copying or moving a specified folder or model to another folder within the Revit Server without starting up Revit, but not copying a model to a network location outside of the server.

#### Addendum

Please note that [downloading from Revit Server requires an `RSN.INI` entry.](http://thebuildingcoder.typepad.com/blog/2017/09/download-from-revit-server-and-hide-a-point-cloud-scan.html#2)