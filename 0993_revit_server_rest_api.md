---
ai_optimized: true
author: Jeremy Tammik
complexity_score: 3.1
content_type: qa
optimization_date: '2025-12-11T11:44:15.075311'
original_url: https://thebuildingcoder.typepad.com/blog/0993_revit_server_rest_api.html
post_number: 0993
reading_time_minutes: 3
series: general
slug: revit_server_rest_api
source_file: 0993_revit_server_rest_api.htm
tags:
- revit-api
title: The Revit Server REST API
word_count: 690
---

### The Revit Server REST API

Let's take a look at the material available for programmatic access to the Revit Server.

This came up in a developer query on automatically extracting data from a Revit model housed there.

The automatic extraction of data requires batch mode opening of a Revit model and launching of a command, for which there is no direct Revit API support.

However, this can be easily worked around using normal batch files to start Revit with a given project file, an add-in implementing OnStartup and OnDocumentOpened event handlers, and possibly the Idling event. The unsupported journaling mechanism may also come in handy, but should and mostly can easily be avoided.

Simply put:

- Implement a batch file that calls Revit.exe and provides a project file command line argument.
- Revit will start up and load that file.
- If you have installed an external application, its OnStartup method will be called and can perform certain actions, e.g. subscribe to the document opened event.
- In the document opened event handler, you can start manipulating the document in the way you need.
- If that event is too restrictive, you can subscribe to the Idling event in the event handler.
- The Idling event handler is free to do almost anything you can imagine and should be enough to fulfil your needs.

With that in place, we can proceed with the Revit Server question:

**Question:** Yes, we have built a console application for executing Revit.exe to open the Revit file and extract the data using the add-in we developed.
This is working perfectly fine.

Our next goal is to automate the extraction process from the Revit Server standpoint.

In out current scenario, we have Revit Server and Revit Client running in multiple locations with 50-75 users, all updating their Revit files to the server.

Question 1: How can I determine which models have been modified or updated in Revit Server.
Can the Revit Server API provide this information?
We obviously only want to extract the data from modified models.

Question 2: Where do I locate the RVT project files in the Revit Server?
When working in Revit I see a folder structure like this:
![Miniscule RVT file](img/miniscule_rvt.png)

Please note the miniscule file size of the highlighted Model.rvt, which is just 1 KB.

If I try open this file in Revit MEP, it fails and throws an exception.

**Answer:** Congratulations on completing your command line tool for driving Revit and automatically extracting the required data.

Regarding the Revit Server API, I presented some basic information and a sample application making use of the
[Revit Server REST API](http://thebuildingcoder.typepad.com/blog/2011/11/revit-server-rest-api.html) back
in 2011, followed by a discussion on how to use it to
[copy a model from a Revit Server](http://thebuildingcoder.typepad.com/blog/2011/12/copy-a-model-from-a-revit-server.html).
The Revit Server API has not changed since then.

You might want to start off by looking at the user interface tools for
[managing Revit Server](http://wikihelp.autodesk.com/Revit/enu/2014/Help/3663-BIM_Mana3663/3691-Revit_Se3691/3721-Revit_Se3721/3724-Managing3724) to get an idea of how the system works.

My colleague Adam Nagy presented the Revit Server REST API at DevCamp using this slide deck,
[Revit Server API.pdf](zip/Revit Server API.pdf),
which focuses mainly on the REST API aspect in general and includes several useful pointers to further information at the end.

All further documentation and samples are provided in the Revit Server SDK, which is part of and lives in its own sub-folder of the standard Revit SDK.

You can also find a number of further examples on the Internet by
[googling for 'revit server rest api'](http://lmgtfy.com/?q=revit+server+rest+api), e.g.:

- [Get a listing of all Revit Server projects through the REST interface](http://blog.rodhowarth.com/2011/09/revit-server-how-to-get-listing-of-all.html).
- [Revit Server model list using the REST API](http://blog.ericstimmel.com/2012/05/08/revit-server-model-list).