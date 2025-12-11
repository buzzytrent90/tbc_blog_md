---
ai_optimized: true
author: Jeremy Tammik
complexity_score: 1.9
content_type: qa
optimization_date: '2025-12-11T11:44:15.295250'
original_url: https://thebuildingcoder.typepad.com/blog/1095_post_to_revit_server.html
post_number: '1095'
reading_time_minutes: 2
series: general
slug: post_to_revit_server
source_file: 1095_post_to_revit_server.htm
tags:
- levels
- revit-api
- views
title: REST POST Request to Revit Server 2014
word_count: 414
---

### REST POST Request to Revit Server 2014

I just arrived in Gothenburg, Sweden, for a mini-web-workshop.
I'll let you know what I learn here soon.

Meanwhile, here is a quick little important note on a change to the Revit Server 2014 REST API that affected some:

**Question:** I am using a POST request to copy a RVT file from one Revit Server folder to another.
This call worked fine in Revit Server 2013, but returns an error code 404 in Revit Server 2014.
I looked at the documentation of the request and can see no changes.

I can perform GET requests on my folders in 2014 and get valid responses, but a POST will not work.
Here are the strings that I am using for my testing, corresponding to the SDK document examples:

- "POST"
- "http://ABC-REV-T01/RevitServerAdminRESTService2014/AdminRESTService.svc/10Folder/descendent?sourceObjectPath=00Folder|01CentralModel.rvt&pasteAction=Move&duplicateOption=CopyIncrement"

I reviewed all the
[Revit Server API access](http://thebuildingcoder.typepad.com/blog/2013/08/revit-server-api-access-and-vbscript.html)
sample code I could find to no avail.

All of the other requests work fine with 2014.
I also compared my POST lines for 2013 and 2014.
They are identical, and so are the ones provided in the 2013/2014 PDFs.
If I take the post lines I have created for 2014 and change them back to 2013 they work just fine as well.

What in heaven's name is the problem, please, and, above all, how can I fix it?

**Answer:** Oops.
Sorry.
The signature of that API call changed since the documentation you are using was created.

The new syntax is:

- Method: POST
- URI: {base URI}/{sourceObjectPath}?destinationObjectPath={destinationObjectPath}&pasteAction={pasteAction}&replaceExisting={replaceExisting}
  - {destinationObjectPath} = destination path of the operation
  - {pasteAction} = Move or Copy
  - {replaceExisting} = true or false

**Example:** Given a model called WorkingModel.rvt in a folder called OldFolder at the root of the Projects tree that needs to be moved to a new folder at the same level called NewFolder.
Here is the call to achieve that:

- Method: POST
- URI:

  {base\_URI}/OldFolder|WorkingModel.rvt

  ?destinationObjectPath=NewFolder|WorkingModel.rvt

  &pasteAction=Move

  &replaceExisting=true

**Response:** Works like a charm.

I have been adding quite a bit of functionality to my original copy down tool, e.g., the ability for users to create projects, add RVT's with specified names, and create a CMD file to batch copy down central files for our Navisworks integration.

Thank you.