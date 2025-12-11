---
ai_optimized: true
author: Jeremy Tammik
complexity_score: 2.9
content_type: qa
optimization_date: '2025-12-11T11:44:16.747457'
original_url: https://thebuildingcoder.typepad.com/blog/1788_testrunner.html
post_number: '1788'
reading_time_minutes: 5
series: general
slug: testrunner
source_file: 1788_testrunner.md
tags:
- references
- revit-api
- sheets
- views
title: Testrunner
word_count: 939
---

### TestRunner – Run Unit Tests in Revit
A new Revit add-in unit testing framework, a short note on support assembly locations, and an article on importing PDF files:
- [`Revit.TestRunner` – run unit tests in Revit](#2)
- [Getting started with `TestRunner`](#3)
- [Unconfusing support assemblies](#4)
- [Importing PDFs made easy](#5)
#### Revit.TestRunner – Run Unit Tests in Revit
We mentioned several
different [Revit add-in unit testing frameworks](https://thebuildingcoder.typepad.com/blog/about-the-author.html#5.16) in
the past.
Tobias Flöscher of [Geberit](https://www.geberit.com) now
shares a new implementation based on [NUnit 3](https://nunit.org) that looks very promising indeed,
in the [Revit API discussion forum](http://forums.autodesk.com/t5/revit-api-forum/bd-p/160) thread
on [`Revit.TestRunner` – run unit tests in Revit](https://forums.autodesk.com/t5/revit-api-forum/revit-testrunner-run-unittests-in-revit/m-p/9070621),
saying:
Hello Developers,
In 2018, I started to program in the Revit API environment.
It was completely new to me, so I had to get familiar with the API and its peculiarities.
I found The Building Coder blog, which helped me a lot.
We insourced an add-in that was originally developed by an external company.
The whole code lives in an assembly which references the Revit API.
Then I realized, testing is hard.
I refactored the code so that it was as much as possible independent from the Revit API.
This code was testable as I knew it, using tests run
on [continuous integration](https://en.wikipedia.org/wiki/Continuous_integration).
But a lot of code remained in the project referencing the Revit API.
I asked Google what to do, but apparently there is not THE way to solve this problem.
There is some stuff around that seems cool, e.g., [RevitTestFramework](https://thebuildingcoder.typepad.com/blog/2018/08/revit-unit-test-framework-improvements.html),
but I was not happy with that.
Furthermore, all I found uses NUnit 2.6, whereas all my Revit independent code is tested by the NUnit 3.x.
The build server doesn't like the mix.
As a result, I started to write my own TestRunner using NUnit 3.
It is far from perfect; nevertheless, I would like to share it with the community.
Please have a look at it in
the [Revit.TestRunner GitHub repository](https://github.com/geberit/Revit.TestRunner).
tf_testrunner_start.png
Features so far:
- Using NUnit 3
- No reference to Revit.TestRunner needed in test assembly
- Support of NUnit attributes SetUp and TearDown
- Injection of API objects in test method
- Works as an add-in with GUI
- Installation script for libraries
- Post build event in VS project to place add-in manifest
Feedback welcome!
#### Getting Started with TestRunner
Get the code from GitHub and compile it.
The \*Revit.TestRunner.addin\* file will be automatically placed in the \*ProgramData\* add-in folder of the selected Revit version,
It is also possible to download the precompiled binaries.
Start the \*InstallAddin v20xx.cmd\* of your favourite Revit version and run Revit.
The add-in hooks into the Revit 'Add-Ins' ribbon.
![TestRunner ribbon tab](img/tf_testrunner_start.png)
By pressing the button, a dialogue will appear.
By choosing your testing assembly, the view will show all your tests:
![TestRunner user interface](img/tf_testrunner_ui.png)
Select the node you want to test and press the 'Run'. All tests below the selected node will be executed.
![TestRunner results](img/tf_testrunner_ui_executed.png)
Very many thanks to Tobias for implementing, documenting and sharing this very important new tool!
#### Unconfusing Support Assemblies
A quick note on
a [confusing assembly location when using one assembly for two Revit add-in projects](https://forums.autodesk.com/t5/revit-api-forum/confusing-assembly-location-when-use-one-assembly-for-two-revit/m-p/9070282):
\*\*Question:\*\* I created a support library project implement custom classes and functions and then used this library reference in two Revit add-in projects.
I want to install each project separately, so each project should contain their own library assembly in their setup folder.
In this way, I can easily manage every single project without caring about others.
I use the method `Assembly.GetExecutingAssembly` to check where the library.dll is referring to.
When I first run the project 1, I get the correct assembly path – the project 1 folder.
Then, I run project 2 and get the unexpected result: the path is in the project 1 folder again.
This should return project 2 folder as my expectation.
A similar problem occurs if I run the project 2 first.
\*\*Answer:\*\* If you load an assembly `A` into the .NET framework from directory `Da` and then load another version of assembly `A` from a different location `Db`, the .NET framework will determine that `A` has already been loaded and reuse the existing instance from `Da`, ignoring your request to load a second instance from `Db`. There are ways to work around this, presumably, but I will not even try to dive into that. I suggest that you design your two add-ins so that they do not rely on the location of the assembly `A` that you load in any way. You can easily move your assembly location determination code out of `A` into the main two add-ins instead.
#### Importing PDFs Made Easy
Konrad Sobon of [@arch_laboratory](https://twitter.com/arch_laboratory) shared a new Dynamo solution
on [archi+lab](http://archi-lab.net)
for [importing PDFs made easy](https://archi-lab.net/importing-pdfs-made-easy),
using and expanding on the Revit 2020 ability to import PDF files and making it a little bit easier to place multi-page documents.