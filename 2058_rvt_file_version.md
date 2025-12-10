---
post_number: "2058"
title: "Rvt File Version"
slug: "rvt_file_version"
author: "Jeremy Tammik"
tags: ['family', 'filtering', 'python', 'revit-api', 'sheets', 'views']
source_file: "2058_rvt_file_version.md"
original_url: "https://thebuildingcoder.typepad.com/blog/2058_rvt_file_version.html"
---

### Determine RVT Version and Add Data from EXE
A fresh look at the venerable topic of determining the RVT file version, a short example of driving Revit from outside to add extensible storage data, and lots of AI- and tech-related snippets:
- [Determine RVT file version](#2)
- [RVT file version with RevitExtractor](#3)
- [Add extensible storage data from EXE](#4)
- [Tesla autonomous vehicles and robots](#5)
- [State of AI Report 2024](#6)
- [Nobel prize for science](#7)
- [AI illiteracy and misuse](#8)
- [The techno-pro attitude](#9)
- [Jevons paradox](#10)
#### Determine RVT File Version
The RVT file version is the version of Revit that was used to save an RVT project document.
Revit can open RVT files saved by earlier versions of Revit, and not those saved by later versions.
However, opening an earlier version requires additional time on opening to convert the database from the earlier version to the current one.
Therefore, it is often desirable to open an RVT document with the same version of Revit used to save it.
Consequently, it is useful to know the RVT file version before launching the corresponding version of Revity to open it.
The Building Coder and
many [Revit API discussion forum](http://forums.autodesk.com/t5/revit-api-forum/bd-p/160) threads discuss how this can be achieved:
- [RVT File Version](http://thebuildingcoder.typepad.com/blog/2008/10/rvt-file-version.html)
- [RFA File Version](http://thebuildingcoder.typepad.com/blog/2009/06/rfa-version-grey-commands-family-context-and-rdb-link.html#1)
- [Basic File Info and RVT File Version](http://thebuildingcoder.typepad.com/blog/2013/01/basic-file-info-and-rvt-file-version.html)
- [Determining RVT File Version Using Python](http://thebuildingcoder.typepad.com/blog/2017/06/determining-rvt-file-version-using-python.html)
- [RvtVerFileOpen – Automatically Open Correct RVT File Version](https://thebuildingcoder.typepad.com/blog/2020/05/automatically-open-correct-rvt-file-version.html#4)
- [Doc Session Id, API Context and External Events](https://thebuildingcoder.typepad.com/blog/2020/11/document-session-id-api-context-and-external-events.html#4)
- [Know the version of a RVT file before opening the file in Revit](https://forums.autodesk.com/t5/revit-api-forum/i-would-like-to-know-the-version-of-a-rvt-file-before-opening/td-p/8403150)
The official approach is to use the Revit API `BasicFileInfo`;
Since it is a Revit API method, it requires a Revit session up and running with an appropriate add-in loaded, but it does not require opening the RVT file in question.
Some of the previously discussed solutions above work by extracting and analysing strings directly from the raw RVT file, or using the OLE document structure;
apparently, those solutions no longer work.
By the way, another use of `BasicFileInfo` was discussed last week
to [unload links with transmission data](https://thebuildingcoder.typepad.com/blog/2024/10/unload-links-offline-and-filter-for-types.html#5).
How to determine RVT Version came up again in the following discussion:
\*\*Question:\*\*
We use [Autodesk Platform Services APS](https://aps.autodesk.com/) APIs to monitor model health via dashboards, but we're having trouble retrieving model version years and cloud GUIDs (e.g., `994d0d57-67d7-4288-a3e8-e9bce5d0cae5`).
Is there an easier way to access this data than using the Model Derivative API?
\*\*Answer:\*\*
The Regex approach described in the early blog posts does not work for recent Revit versions, e.g. Revit 2024 and Revit 2025.
In these versions, you can access the saved version in a formal, reliable way using the Revit API `BasicFileInfo` class, which doesn’t require opening the file in question.
a Design Automation Sample of extracting `BasicFileInfo` from RVT or RFA file is provided by
the [DA4R-RevitBasicFileInfoExtract](https://github.com/yiskang/DA4R-RevitBasicFileInfoExtract) project on GitHub.
If I remember correctly, the regex method doesn’t return version info when testing with Revit 2024 models. (seemed due to scheme changes).
A customer reported this issue in a ticket on \*Identifying Revit File Version\*.
We used DA4R-RevitBasicFileInfoExtract with Design Automation API for Revit instead to address this and reliably retrieve the Revit file version year, cf. the article on
how to [check the version of a Revit file using Design Automation API](https://aps.autodesk.com/blog/check-version-revit-file-using-design-automation-api).
The DA approach is much more reliable, based on my tests.
I tried the Regex one with the Revit 2024 file and spent the whole afternoon trying to find out why it didn’t work, but failed;
then I wrote the DA4R-RevitBasicFileInfoExtract sample addin for DA instead.
With Design Automation API, the customer does not have to install Revit on the local machine, just run the workitem.
The Revit 2025 version can read all RVT versions equal to and lower than 2025.
So, no need to make a separate app bundle for each Revit version.
Many thanks to Eason Kang for this explanation and all his research and documentation.
#### RVT File Version with RevitExtractor
[Chuong Ho](https://chuongmep.com/) suggested another solution for this topic in
the [Revit API discussion forum](http://forums.autodesk.com/t5/revit-api-forum/bd-p/160) thread
on how to [know the version of a RVT file before opening the file in Revit](https://forums.autodesk.com/t5/revit-api-forum/i-would-like-to-know-the-version-of-a-rvt-file-before-opening/td-p/8403150):
A simple solution is provided by
using [Revit Extractor](https://github.com/chuongmep/revit-extractor),
a Python library to easily read data and export Revit data from the native Revit format.
Make sure you installed the package and use the right command:

```
from revit_extract import RevitExtractor
rvt_path = r"D:_WIP\Download\Sample Office Building Model V1.rvt"
version = RevitExtractor.get_version(rvt_path)
print(version)
```

![RevitExtractor](img/revitextractor.png "RevitExtractor")
#### Add Extensible Storage Data from EXE
[Mohamed Arshad](https://forums.autodesk.com/t5/user/viewprofilepage/user-id/8461394) explains the detailed steps to launch Revit from an external EXE to modify and save an RVT or RFA document, e.g.,
to [set `DataStorage` `Entity` from external application](https://forums.autodesk.com/t5/revit-api-forum/set-datastorage-entity-from-external-application/td-p/13085263):
\*\*Question:\*\*
Exploring options to create `DataStorage` and set entity in Revit document from external application.
After setting DataStorage entity, I need to save the document.
This needs to be done running a Revit instance in the background.
Currently, I was able to create storage and update entity in an add-in.
How can I achieve the same from a separate `exe`?
The question is, how to start a new Revit instance from an `exe` and access the Revit API to update the data storage entity.
\*\*Answer:\*\*
You can start Revit from an external `EXE` using [Process.Start](https://duckduckgo.com/?q=process.start).
All the other steps you describe require a valid Revit API context: to create `DataStorage`, set extensible storage data in its `Entity` and save the document.
One way to obtain a valid Revit API context is to use
an [external event](https://www.revitapidocs.com/2024/05089477-4612-35b2-81a2-89c4f44370ea.htm).
Many examples are provided in The Building Coder topic group
on [`Idling` and external events for modeless access and driving Revit from outside](https://thebuildingcoder.typepad.com/blog/about-the-author.html#5.28).
Here is a possible step-by=step approach:
- Implement your Extensible storage helper methods in a static class.
- Design a simple standalone application (exe) that starts Revit in a separate process.

```
static void Main(string[] args)
{
  string exeLocation = @"C:\Program Files\Autodesk\Revit 2023\Revit.exe";
  string arguments = @"/languageENG /runhidden /nosplash";

  ProcessStartInfo revitInfo= new ProcessStartInfo(exeLocation,arguments);
  Process.Start(revitInfo);
}
```

- In the `OnStartup` method your application implements, subscribe to `ApplicationInitialized` event
- When your add-in receives the event, open the document you want to add data storage by calling the helper methods
- When the document is opened, perform the process; output whatever you need to output
- Save and close the document
- Exit the `ApplicationInitialized` event handler
- When the standalone application receives the inter-process message from your add-in, kill the Revit process
Many thanks to Mohamed for spelling out the detailed steps.
#### Tesla Autonomous Vehicles and Robots
Moving away from the Revit API to other AI-related and technical issues,
Tesla presented plans for autonomous vehicles and humanoid robots in an 8-minutes video of
their [We, Robot Event](https://youtu.be/Mu-eK72ioDk):
> CEO Elon Musk unveils Robotaxi, a fully autonomous car for less than $30,000, Robovan, a 20 passenger vehicle, and new updates to its humanoid robot, Optimus, for less than the cost of a car.
#### State of AI Report 2024
Further AI-related news is presented in
the [State of AI Report 2024](https://www.stateof.ai/).
#### Nobel Prize for Science
The Nobel prize for science is also related to AI and deep learning, awarded to
[John J. Hopfield and Geoffrey E. Hinton, training early artificial neural networks](https://www.nobelprize.org/prizes/physics/2024/press-release/).
They used [physics to find patterns in information](https://www.nobelprize.org/uploads/2024/10/popular-physicsprize2024-2.pdf);
Further [Scientific Background to the Nobel Prize in Physics 2024, For Foundational Discoveries And Inventions That Enable Machine Learning With Artificial Neural Networks](https://www.nobelprize.org/uploads/2024/09/advanced-physicsprize2024.pdf).
#### AI Illiteracy and Misuse
One view of the risks associated with AI proposes that
[AI will not destroy the world – AI illiteracy and misuse could](https://www.forbes.com/sites/luisromero/2024/10/08/ai-will-not-destroy-the-world-ai-illiteracy-and-misuse-could/).
#### The Techno-Pro Attitude
Another interesting philosophical discussion looks critically at scientific progress in general:
analysing [The Techno-Pro Attitude](https://cacm.acm.org/article/do-all-problems-have-technical-fixes/),
Robin K. Hill suggests we look at the underlying presumption of the technology imperative.
#### Jevons Paradox
Let's end mentioning another common fallacy,
the [Jevons paradox](https://en.wikipedia.org/wiki/Jevons_paradox);
it occurs when technological progress increases the efficiency with which a resource is used (reducing the amount necessary for any one use), but the falling cost of use induces increases in demand enough that resource use is increased, rather than reduced.
Governments, both historical and modern, typically expect that energy efficiency gains will lower energy consumption, rather than expecting the Jevons paradox.
On November 24, Switzerland will vote on whether to spend heavily to increase motorway bottleneck capacity, hoping to reduce traffic jams, possibly heading straight into another unfortunate example of the Jevons paradox.