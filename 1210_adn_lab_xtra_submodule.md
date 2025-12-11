---
ai_optimized: true
author: Jeremy Tammik
complexity_score: 8.8
content_type: code_example
optimization_date: '2025-12-11T11:44:15.515836'
original_url: https://thebuildingcoder.typepad.com/blog/1210_adn_lab_xtra_submodule.html
post_number: '1210'
reading_time_minutes: 9
series: general
slug: adn_lab_xtra_submodule
source_file: 1210_adn_lab_xtra_submodule.htm
tags:
- elements
- levels
- parameters
- python
- revit-api
- transactions
- views
title: ADN Labs Xtra on GitHub and RvtVa3c in Three.js
word_count: 1796
---

### ADN Labs Xtra on GitHub and RvtVa3c in Three.js

I had several cases recently discussing advanced Revit API exploration issues with experienced application developers new to the Revit API.

Therefore, once again, the topic of available tools and their various uses came up.

One important tool for me is the simple element lister, which is currently still part of the ADN Xtra labs, the precursor to the official ADN Revit API training labs.

I had not yet migrated those to Revit 2015, so I now finally did so and posted them to GitHub.

Another interesting GitHub learning step today was integrating the Revit JSON model exporter RvtVa3c for the vA3C AEC viewer into the official list of three.js exporters.

So this is what I discuss today:

- [Getting Started with the Revit API](#2)
- [Revit database exploration tools](#3)
- [ADN Revit API Xtra training labs for Revit 2015](#4)
- [Integrating RvtVa3c into three.js](#5)
- [Updated integration of RvtVa3c into three.js](#6)

#### Getting Started with the Revit API

One important step before starting to think about working on a Revit add-in is to understand the basic Revit concepts and workflows from a user point of view.

Revit is
[significantly different from most traditional CAD systems](http://thebuildingcoder.typepad.com/blog/about-the-author.html#5.41),
and this is thoroughly reflected in the Revit API as well.

Once that is understood, the next and obvious step is to work through the
[Revit API getting started material](http://thebuildingcoder.typepad.com/blog/about-the-author.html#2).

It provides both a 'My First Revit Plug-in' and DevTV tutorials discussing all the basics of add-in development and leading you through the entire process step by step, including setting up the development environment, SDK, basic tools, understanding the add-in architecture, installation, coding, compilation and main API principles.

Often some research is required to determine how to programmatically address a specific task.

The main important documentation components that you absolutely must be aware of are:

- The Revit SDK, including:

- The Revit API help file RevitAPI.chm- The extensive SDK sample collection and solution SDKSamples2015.sln
  - The SDK external application add-in RvtSamples that loads all the external command samples

- The online Revit developer guide
- The database exploration tool RevitLookup

The help file documents all classes, their properties and methods. It does not say much about how they work together, though.

The developer guide explains concepts, workflows, more complex and higher-level relationships, and how the classes, properties and methods work together.

The SDK samples demonstrate real code executing most of the workflows in a simple manner.
The sample solution file enables compilation and debugging of all samples in one fell swoop, and RvtSamples provides a method to load all external command samples in one go, instead of manually installing a hundred or so separate add-ins.

In addition to this brief overview, please do not miss this
[more detailed explanation](http://thebuildingcoder.typepad.com/blog/2013/04/getting-started-with-the-revit-api.html) and the recommendations on
[preparing for a hands-on Revit API training](http://thebuildingcoder.typepad.com/blog/2012/01/preparing-for-a-hands-on-revit-api-training.html).

Even with all the documentation in place and at your disposal, there are a number of undocumented areas and workflows.

You will often need to research properties, relationships and methods used to create and modify the model for your specific needs yourself.

One of the most powerful ways to approach this research is to create a simple sample model manually, store that starting point, perform the required changes by hand in the user interface, store the ending point, and compare the differences.

A detailed comparison of the models before and after requires database exploration tools.

#### Revit Database Exploration Tools

The first and foremost Revit database exploration tool is **RevitLookup**.

You absolutely must install and understand the usage of this tool.
It is actually also pretty useful for end users.

Here are a number of discussions of various [uses of RevitLookup](http://thebuildingcoder.typepad.com/blog/revitlookup).

It is a simple interactive dialogue based tool that enables you to navigate the Revit database, its elements, their properties and relationships with each other.

It is a statically compiled add-in and provided in source code in the [RevitLookup GitHub repository](https://github.com/jeremytammik/RevitLookup).

Once you have found the objects and properties you are interested in via the RevitLookup user interface, you can use the Visual Studio environment and debugger to explore how that property or relationship can be reached programmatically.

Once you have acquainted yourself with that static interactive exploration method and understand the database structure, you can move on to more powerful
[interactive, dynamic, interpreted API interaction](http://thebuildingcoder.typepad.com/blog/2013/11/intimate-revit-database-exploration-with-the-python-shell.html), e.g. using the **Revit Python or Ruby shells**.

Both RevitLookup and the interactive shells will enable you to explore elements and list their parameters. The shells will also allow you to manually set up transactions and modify the database from the command line in real time.

Another statically compiled tool specifically developed for exploring the parameters attached to elements is **BipChecker**, the built-in parameter checker:

- [Duplicate Built-in parameter values and BipChecker update](http://thebuildingcoder.typepad.com/blog/2013/01/built-in-parameter-enumeration-duplicates-and-bipchecker-update.html)
- [Retrieving ElementType parameters and BipChecker](http://thebuildingcoder.typepad.com/blog/2013/09/10000000000th-post-and-element-type-parameters.html#2)
- [BipChecker for Revit 2015 on GitHub](http://thebuildingcoder.typepad.com/blog/2014/05/bipchecker-for-revit-2015-on-github.html)

Yet another very simple and often surprisingly useful tool for me is the **element lister**.

I recently described its use and pointed to other discussions of it while determining the
[relationship between an image element and the associated ImageType](http://thebuildingcoder.typepad.com/blog/2014/09/debugging-and-maintaining-the-image-relationship.html).

#### ADN Revit API Xtra Training Labs for Revit 2015

The element lister is part of my ADN Xtra labs:

- [Xtra ADN Revit 2013 API Training Labs](http://thebuildingcoder.typepad.com/blog/2012/04/xtra-adn-revit-2013-api-training-labs.html)
- [Exporting Parameter Data to Excel, and Re-importing](http://thebuildingcoder.typepad.com/blog/2012/09/exporting-parameter-data-to-excel.html)
- [ADN Xtra Labs and BipChecker for Revit 2014](http://thebuildingcoder.typepad.com/blog/2013/09/10000000000th-post-and-element-type-parameters.html#2)
- [ADN Revit API Training Material on GitHub](http://thebuildingcoder.typepad.com/blog/2013/10/revit-2013-api-developer-guide-pdf.html#3)

Answering one of these cases, I noted that I had not yet migrated them to Revit 2015, and finally got around to doing so.

The migration was pretty straightforward, and the ADN Revit API training material including the unofficial Xtra labs now resides in its own
[AdnRevitApiLabsXtra GitHub repository](https://github.com/jeremytammik/AdnRevitApiLabsXtra).

To avoid confusion and be absolutely clear, I will repeat the contents of the repository readme here:

> This repository contains the source code and Visual Studio solution of the ADN Revit API Training Labs including the old historical Xtra samples.
>
> The official collection excluding the Xtra labs is available from the Autodesk Developer Network ADN
> [Revit Developer page](http://www.autodesk.com/developrevit).
> Look there under Samples and Documentation.
> It lives in its own ADN DevTech
> [RevitTrainingMaterial GitHub repository](https://github.com/ADN-DevTech/RevitTrainingMaterial).
>
> If you have no need for the Xtra labs or do not know what they are, please stick with the official version provided there.

#### Integrating RvtVa3c into Three.js

I recently mentioned the
[significant progress on the vA3C project](http://thebuildingcoder.typepad.com/blog/2014/08/threejs-aec-viewer-progress-on-two-fronts.html#4)
([home](https://va3c.github.io), [git](https://github.com/va3c)),
the generic three.js based AEC viewer.

One important aspect is the discovery that the vA3C JSON models are fully compatible and can be merged and mingled with standard three.js ones, providing the models are stored as objects, not as scenes, as discussed in the issue
[#5 on exporters and JSON output](https://github.com/va3c/va3c.github.io/issues/5).

I modified
[RvtVa3c](http://thebuildingcoder.typepad.com/blog/va3c),
the Revit vA3C JSON model exporter to produce such a JSON file and tried to add it to the official
[three.js exporter list](https://github.com/mrdoob/three.js/tree/master/utils/exporters):

![Three.js exporters](img/three_js_exporters_1.png)

The way to do this is to fork the repository, apply the required changes, and submit a pull request.

Here is what I ended up doing to hopefully achieve that:

- Fork the three.js `dev` branch into my personal collection of repositories.
- Clone my forked version to my local system:

```
$ git clone https://github.com/jeremytammik/three.js
```

- Add RvtVa3c as a new submodule in the utils/exporters folder:

```
$ git submodule add https://github.com/va3c/RvtVa3c utils/exporters/revit
```

- Commit

```

```

- Push
- In the forked repository, click 'Pull request', describe the submission, and submit it.

The result in my local repository looks promising enough:

![Three.js exporters in my forked repository](img/three_js_exporters_2.png)

I hope it works :-)

#### Updated Integration of RvtVa3c into Three.js

The linking of the RvtVa3c repository into the list of three.js exporters described above worked, but...

... as discussed in the
[three.js issue #5297](https://github.com/mrdoob/three.js/pull/5297),
mrdoob thinks "the folder approach was better.
This repo currently has no submodule.
And submodules don't get included when downloading the zip of the repo...
the idea of just having a README.md file in a revit/ folder with a url to the other repo should do the trick."

I therefore deleted eveything I did above, reforked the three.js repository to start again from scratch and performed the following steps on it:

- Fork the three.js `dev` branch into my personal collection of repositories.
- Clone my forked version to my local system:

```
$ git clone https://github.com/jeremytammik/three.js
```

- Add a new subfolder 'revit' to the utils/exporters folder:

```
$ cd three.js/utils/exporters/
$ mkdir revit
```

- Edit and add the readme file

```
$ touch README.md
$ edit README.md
$ git add .
```

- Commit

```
$ git commit -m "added Revit three.js JSON exporter RvtVa3c"
```

- Push

```
$ git push
```

- In the forked repository, click 'Pull request', describe the submission, and submit it.

The result is a new pull request
[#5305](https://github.com/mrdoob/three.js/pull/5305)
"added the revit exporter RvtVa3c: simply a subfolder utils/exporter/revit with a readme file pointing to the RvtVa3c repository".

I agree that this is simpler, of course, and more suitable, considering the facts stated above.

As an added advantage, I am getting the hang of this now...