---
post_number: "0971"
title: "More on Structural Analysis Code Checking"
slug: "rst_code_check"
author: "Jeremy Tammik"
tags: ['elements', 'parameters', 'revit-api', 'views']
source_file: "0971_rst_code_check.htm"
original_url: "https://thebuildingcoder.typepad.com/blog/0971_rst_code_check.html"
---

### More on Structural Analysis Code Checking

I am busy making the final preparations for my trip to Moscow tomorrow.
However, I also received some new input to share with you first on the
[Structural Analysis SDK](http://thebuildingcoder.typepad.com/blog/2013/06/structural-analytical-code-checking-and-results-builder.html) that
I mentioned last week, introduced in the
[Revit 2014 SDK](http://images.autodesk.com/adsk/files/Revit2014SDK_RTM.exe).

It enables engineering experts to provide specialised structural code checking applications on top of Revit, and now also supports storage of analysis results.

[Augusto Goncalves](http://adndevblog.typepad.com/aec/augusto-goncalves.html) recently took
a more in-depth
[first look](http://adndevblog.typepad.com/aec/2013/05/structural-analysis-sdk-first-look.html) at
it from a technical point of view.

The results storage API and code checking framework requires installation of the
[Structural Analysis and Code Checking Toolkit](http://apps.exchange.autodesk.com/RVT/Detail/Index?id=appstore.exchange.autodesk.com%3astructuralanalysisandcodecheckingtoolkit2014%3aen) from the Autodesk Exchange Apps, discussed in more detail in this
[BIM and Beam blog post](http://bimandbeam.typepad.com/bim_beam/2013/04/structural-analysis-and-code-checking-toolkit-2014-on-autodesk-exchange-apps.html).

Here is a sequence of images illustrating the code checking process:

#### Code Checking Process in Revit

Analytical results in Revit

![Analytical results in Revit](img/rst_cc/03.jpg)

Physical and analytical model

![Physical and analytical model](img/rst_cc/04.jpg)

Analysis results:

![Analysis results](img/rst_cc/05.jpg)

Step 1 – code parameter settings:

![Step 1 – code parameter settings](img/rst_cc/06.jpg)

Step 2 – element type parameter settings:

![Step 2 – element type parameter settings](img/rst_cc/07.jpg)

Step 3 – type parameter assignment to elements:

![Step 3 – type parameter assignment to elements](img/rst_cc/08.jpg)

Step 4 – execute and explore reports:

![Step 4 – execute and explore reports](img/rst_cc/09.jpg)

Step 5 – explore graphical results:

![Step 5 – explore graphical results](img/rst_cc/10.jpg)

Collective approach for a code checking solution:

![Collective approach for a code checking solution](img/rst_cc/11.jpg)

Exchange Apps:

![Exchange Apps](img/rst_cc/12.jpg)

Software Development Kit for Code Checking Documentation:

- Getting Started
- Step by Step instructions
- Samples
- API documentation

![Software Development Kit for Code Checking](img/rst_cc/14.jpg)

Visual Studio:

- Data management
- UI definition
- Interactions
- Report generation
- Localization support

![Visual Studio](img/rst_cc_16.png)

Source code:

![Source code](img/rst_cc_17.png)

#### Closing Notes

As a last little note, Robot is now available as part of the cloud-based
[Autodesk SIM 360](http://usa.autodesk.com/adsk/servlet/pc/index?id=20414035&siteID=123112&mktvar001=509406&mktvar002=509406) simulation
package providing access to the simulation functionality from the regular Revit Structure user interface.

Now I really have to get back to the final touches on my
[Moscow Revit DevCamp](http://www.autodesk.ru/adsk/servlet/pc/index?id=21516340&siteID=871736)
presentation preparations...