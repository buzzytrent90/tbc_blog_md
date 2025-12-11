---
ai_optimized: true
author: Jeremy Tammik
complexity_score: 2.2
content_type: qa
optimization_date: '2025-12-11T11:44:16.651664'
original_url: https://thebuildingcoder.typepad.com/blog/1742_asstringvalue_batch.html
post_number: '1742'
reading_time_minutes: 4
series: parameters
slug: asstringvalue_batch
source_file: 1742_asstringvalue_batch.md
tags:
- elements
- parameters
- revit-api
- schedules
- sheets
- views
title: Asstringvalue Batch
word_count: 746
---

### Batch Processing and Aspects of AsStringValue
I avoid answering non-confidential questions in private, as I tell everyone who tries to do so.
However, some non-confidential queries still come in via ADN, the Autodesk Developer Network.
Here are two that came in today that might be of general interest and therefore seem worth sharing:
- [Batch processing Revit families and documents](#2)
- [`AsString` and `AsValueString` results differ](#3)
#### Batch Processing Revit Families and Documents
\*\*Question:\*\* Revit 2019 is leaking memory when importing families.
We need to do this 1000s of times using automation.
It is a blocking issue.
Please can you advise?
\*\*Answer:\*\* Revit on the desktop is an end user product designed for manual use through the manual user interface.
If you are using it to perform any kind of operation thousands of times, you are using it in an unexpected manner.
You should not be surprised if you run into problems eventually.
The standard method to handle such tasks is something like this:
- Keep exact track of processing so you always know what has been processed so far and what still need to be done.
- Monitor the process health.
- Shut down, restart and continue where you left off if the process starts deteriorating or terminates.
Here is more on the topic of using Revit as a server
for [batch processing Revit documents](http://thebuildingcoder.typepad.com/blog/2015/08/batch-processing-dwfx-links-and-future-proofing.html#4).
However, [Revit is not designed to be used as a server](https://thebuildingcoder.typepad.com/blog/2016/04/fireratingcloud-context-and-architecture.html#3), and the EULA actually prohibits such use.
A better and more robust alternative nowadays that also saves you the maintenance of a local Revit installation and enables integration of your batch processing into other web-based workflows is to perform your batch processing using
the [Forge Design Automation API for Revit](https://forge.autodesk.com/en/docs/design-automation/v3/developers_guide/overview).
Here are more articles describing specific aspects
of [DA4R, or Design Automation for Revit](https://thebuildingcoder.typepad.com/blog/about-the-author.html#5.55)
#### AsString and AsValueString Results Differ
\*\*Question:\*\* I am running into a problem using `AsValuesString` and `AsString`.
They return different values.
For instance, a view has a parameter named "Sheet Number".
Its storage type is string.
`AsValueString` returns a blank value for it, whereas `AsString` returns "---":
![Sheet number](img/asvaluestring_sheet_number.png)
Similarly for the Duct Accessory parameter "Loss Method":
![Loss method](img/asvaluestring_loss_method.png)
The same problem occurs with many other element and parameters.
Is there a way to identify which value is correct and to identify which method to use to always to get the correct value?
I am running into the issue exporting schedule data to Excel. In some cases, I export a blank value for a non-blank data item.
When importing back the data, I cannot determine whether the user modified it or it was incorrectly exported.
\*\*Answer:\*\* The Revit Parameter class has
a [storage type](https://apidocs.co/apps/revit/2019/3dbebcb8-792b-a3dd-fe63-faaa05704f3c.htm) that
can take one of the following values:
- None – None represents an invalid storage type. This value should not be used.
- Integer – The internal data is stored in the form of a signed 32-bit integer.
- Double – The data will be stored internally in the form of an 8-byte floating point number.
- String – The internal data will be stored in the form of a string of characters.
- ElementId – The data type represents an element and is stored as the id of the element.
Corresponding to the four valid storage types, there are four accessors to read the stored value from the database:
- `AsDouble` – Provides access to the double precision number within the parameter.
- `AsElementId` – Provides access to the Autodesk::Revit::DB::ElementId^ stored within the parameter.
- `AsInteger` – Provides access to the integer number within the parameter.
- `AsString` – Provides access to the string contents of the parameter.
Use those four, and you will have no problem.
`AsValueString` is a completely different creature that returns the parameter value as a string with units, like the user would see it.
It may perform complex conversions while rendering the string.
Furthermore, a string-valued parameter value can only be retrieved using `AsString`, and `AsValueString` returns an empty string for it, as you already noticed.
I would avoid using `AsValueString` at all in your situation, and keep track of the parameter storage type as well as its value.