---
ai_optimized: true
author: Jeremy Tammik
complexity_score: 2.1
content_type: qa
optimization_date: '2025-12-11T11:44:15.011338'
original_url: https://thebuildingcoder.typepad.com/blog/0967_rst_code_check.html
post_number: 0967
reading_time_minutes: 2
series: general
slug: rst_code_check
source_file: 0967_rst_code_check.htm
tags:
- elements
- levels
- revit-api
- views
title: Structural Analytical Code Checking and Results Builder
word_count: 371
---

### Structural Analytical Code Checking and Results Builder

In case you have not noticed, I would like to point out one important specialised addition to the Revit 2014 SDK: the new Structural Analysis Software Developer Kit.

It lives in its own sub-directory *Structural Analysis SDK* and consists of documentation, sample code and Visual Studio templates for two main components:

- Code checking framework

- Facilitate code checking workflows
- For both users and developers
- Complete extensible add-in
- User interface, report generator, data storage
- Documentation, samples, Visual Studio templates

- Results builder framework

- Store and access results data in Revit
- High-level API for consumption, exchange, remote providers

Here is an overview of the sample applications provided with this toolkit:

- ExtensibleStorageUI and ExtensibleStorageDocumentation – exercise the ExtensibleStorage framework user interface and reporting functionality.
- CodeCheckingConcreteExample – a concrete code checking application with step-by-step document.
- ConcreteCalculationsExample – concrete calculations component for all cases listed in the calculation manual.
- SectionPropertiesExplorer – demonstrate use the engineering component.
- StoringResults and QueryingResults – store and query structural analysis results stored in BIM using the results builder component.

If you are active in this area, this provides a big lump of functionality you definitely need to be aware of, similarly to the
[REX SDK](http://thebuildingcoder.typepad.com/blog/2011/12/rex-content-generator.html) and content generator.

In a related vein, here is a question that just arose today:

**Question:** I would like to start working programmatically with Autodesk Robot to read relevant data result from structural elements such as beam, column, foundations, slab etc.

Where can I find some information about that, please?

**Answer:** Please take a look at the discussion of the
[Robot Structural Analysis API](http://thebuildingcoder.typepad.com/blog/2011/01/optimisation-using-robot-structural-analysis-api.html).

Good luck!

**Addendum:** Here is the complete
[Robot API documentation and tutorial](zip/APITutorial.zip) from April 2010, copied
from the
[API Tutorial](http://forums.autodesk.com/t5/Autodesk-Robot-Structural/API-Tutorial/td-p/3939601) thread
on the
[Autodesk Robot Structural Analysis](http://forums.autodesk.com/t5/Autodesk-Robot-Structural/bd-p/351) discussion
group.
Note that this information is pretty dated by now, so I would not recommend putting in any significant effort developing new projects based on this today.