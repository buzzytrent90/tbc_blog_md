---
ai_optimized: true
author: Jeremy Tammik
complexity_score: 2.2
content_type: qa
optimization_date: '2025-12-11T11:44:14.594839'
original_url: https://thebuildingcoder.typepad.com/blog/0789_central_filename.html
post_number: 0789
reading_time_minutes: 2
series: general
slug: central_filename
source_file: 0789_central_filename.htm
tags:
- revit-api
title: Worksharing Central Filename
word_count: 361
---

﻿

### Worksharing Central Filename

Here is a simple question that several people stumbled over and apparently can use some clarification:

**Question:** In Revit 2012, I used the Document WorksharingCentralFilename property to get the location of the central file.

This is marked as deprecated in Revit 2013, and the API suggests using GetWorksharingCentralModelPath instead.

In my experiments with a central file, I can use this function to obtain a ModelPath object which provides a property named CentralServerPath, but it is blank.

How can I recreate the functionality of the deprecated WorksharingCentralFilename property?

**Answer:** CentralServerPath is the wrong thing to call to get the path to the central model.
It is the path to Revit Server itself.

You obviously need to be clear about whether you are using Revit Server or file-based centrals.
The central server path for a file-based central is obviously blank.

The ModelPath class belongs to the
[linked model related functionality](http://thebuildingcoder.typepad.com/blog/2011/03/many-issues-resolved.html) provided
in the Revit 2012 API, which also includes the ModelPathUtils class.

A ModelPath stores a path to a location on disk, a network drive, or a Revit Server location.

ModelPathUtils provides methods for converting between ModelPath and String.

When working with ModelPath objects, you can use the Compare method to test for equality or for ordering, the predicates IsServerPath and IsFilePath to test whether the path is file-based or server-based, and the ModelPathUtils ConvertModelPathToUserVisiblePath method to convert a ModelPath to a string.

The latter is demonstrated in the discussion of
[listing linked files and the TransmissionData class](http://thebuildingcoder.typepad.com/blog/2011/05/list-linked-files-and-transmissiondata.html).

Other examples of using ModelPathUtils are given in the
[ImportExport update](http://thebuildingcoder.typepad.com/blog/2011/07/importexport-update.html),
[accessing central file TransmissionData on Revit Server](http://thebuildingcoder.typepad.com/blog/2011/11/access-transmissiondata-of-central-file-on-revit-server.html),
[removing DWF links](http://thebuildingcoder.typepad.com/blog/2012/03/remove-dwf-links.html),
and most esoterically for
[selecting a face in a linked file](http://thebuildingcoder.typepad.com/blog/2012/05/selecting-a-face-in-a-linked-file.html).