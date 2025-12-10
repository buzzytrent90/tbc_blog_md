---
post_number: "1801"
title: "Revit Batch Processor"
slug: "revit_batch_processor"
author: "Jeremy Tammik"
tags: ['csharp', 'elements', 'family', 'python', 'revit-api', 'schedules', 'sheets', 'windows']
source_file: "1801_revit_batch_processor.md"
original_url: "https://thebuildingcoder.typepad.com/blog/1801_revit_batch_processor.html"
---

### The Revit Batch Processor RBP
There are so many truly wonderful pieces of software sitting around there that I am unaware of, real works of art, pinnacles of perfection, that I only happen upon by chance.
In this case, \*@alexandrecafarofr\* happens to mention a powerful utility in
his [Revit API discussion forum](http://forums.autodesk.com/t5/revit-api-forum/bd-p/160) thread
on [automatic heating and cooling load analysis](https://forums.autodesk.com/t5/revit-api-forum/automatic-heating-and-cooling-load-analysis-with-revit-api/m-p/9149375):
The [Revit Batch Processor (RBP)](https://github.com/bvn-architecture/RevitBatchProcessor) a project maintained
by [Daniel Rumery](https://github.com/DanRumery), ex-[BVN](http://www.bvn.com.au):
- [Revit Batch Processor (RBP)](#2)
- [Latest version](#3)
- [FAQ](#4)
- [Use cases](#5)
- [Features](#6)
- [Unlimited power](#7)
- [Addendum – questions](#8)
![Captain Dan](img/captain_dan.jpg)
Here is an excerpt of the GitHub project readme to make your mouth water:
#### Revit Batch Processor (RBP)
Fully automated batch processing of Revit files with your own Python or Dynamo task scripts!
#### Latest version
[Installer for Revit Batch Processor v1.5.3](https://github.com/bvn-architecture/RevitBatchProcessor/releases/download/v1.5.3/RevitBatchProcessorSetup.exe)
See the [Releases](https://github.com/bvn-architecture/RevitBatchProcessor/releases) page for more information.
#### FAQ
See the [Revit Batch Processor FAQ](https://github.com/bvn-architecture/RevitBatchProcessor/wiki/Revit-Batch-Processor-FAQ).
#### Use Cases
This tool doesn't _do_ any of these things, but it _allows_ you to do them:
- Open all the Revit files across your Revit projects and run a health-check script against them. Keeping an eye on the health and performance of many Revit files is time-consuming. You could use this to check in on all your files daily and react to problems before they get too gnarly.
- Perform project and family audits across your Revit projects.
- Run large scale queries against many Revit files.
- Mine data from your Revit projects for analytics or machine learning projects.
- Automated milestoning of Revit projects.
- Automated housekeeping tasks (e.g. place elements on appropriate worksets)
- Batch upgrading of Revit projects and family files.
- Testing your own Revit API scripts and Revit addins against a variety of Revit models and families in an automated manner.
- Essentially anything you can do to one Revit file with the Revit API or a Dynamo script, you can now do to many!
![Revit batch processor user interface](img/BatchRvt_Screenshot.png)
#### Features
- Batch processing of Revit files (.rvt and .rfa files) using either a specific version of Revit or a version that matches the version of Revit the file was saved in. Currently supports processing files in Revit versions 2015 through 2020. (Of course, the required version of Revit must be installed!)
- Custom task scripts written in Python or Dynamo! Python scripts have full access to the Revit API. Dynamo scripts can of course do whatever Dynamo can do :)
- Option to create a new Python task script at the click of a button that contains the minimal amount of code required for the custom task script to operate on an opened Revit file. The new task script can then easily be extended to do some useful work. It can even load and execute your existing functions in a C# DLL (see [Executing functions in a C# DLL](#executing-functions-in-a-c-dll)).
- Option for custom pre- and post-processing task scripts. Useful if the overall batch processing task requires some additional setup / tear down work to be done.
- Central file processing options (Create a new local file, Detach from central).
- Option to process files (of the same Revit version) in the same Revit session, or to process each file in its own Revit session. The latter is useful if Revit happens to crash during processing, since this won't block further processing.
- Automatic Revit dialog / message box handling. These, in addition to Revit error messages are handled and logged to the GUI console. This makes the batch processor very likely to complete its tasks without any user intervention required!
- Ability to import and export settings. This feature combined with the simple [command-line interface](#command-line-interface) allows for batch processing tasks to be setup to run automatically on a schedule (using the Windows Task Scheduler) without the GUI.
- Generate a .txt-based list of Revit model file paths compatible with RBP. The \*New List\* button in the GUI will prompt for a folder path to scan for Revit files. Optionally you can specify the type of Revit files to scan for and also whether to include subfolders in the scan.
#### Unlimited Power
> "With great power come great responsibility"
[-- Spiderman](https://quoteinvestigator.com/2015/07/23/great-power/)
This tool enables you to do things with Revit files on a very large scale. Because of this ability, Python or Dynamo scripts that make modifications to Revit files (esp. workshared files) should be developed with the utmost care! You will need to be confident in your ability to write Python or Dynamo scripts that won't ruin your files en-masse. The Revit Batch Processor's 'Detach from Central' option should be used both while testing and for scripts that do not explicitly depend on working with a live workshared Central file.
Ever so many thanks to Dan for implementing, sharing, and, above all, so wonderfully documenting and maintaining this very impressive and powerful application!