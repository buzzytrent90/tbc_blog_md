---
post_number: "1248"
title: "Robot Structural Analysis and Mac App Tabbing"
slug: "rst_robot"
author: "Jeremy Tammik"
tags: ['references', 'revit-api', 'views', 'windows']
source_file: "1248_rst_robot.htm"
original_url: "https://thebuildingcoder.typepad.com/blog/1248_rst_robot.html"
---

### Robot Structural Analysis and Mac App Tabbing

Let's look at these two little issues today:

- [Adapting Robot Structural Analysis](#2)
- [Tabbing between applications on Mac](#3)

#### Adapting Robot Structural Analysis

**Question:** I installed the Revit SDK and am looking at customising Revit Structure and Robot for my needs.

I could only find tutorials for examples of API, but I can't find any information about the database code.

I would like to enhance the implementation based on [Eurocode](https://en.wikipedia.org/wiki/Eurocode) for Brazil.

How can I open and create a new database, like a 'Save As' from Eurocode?

**Answer:** When you say database... are you referring to a content database, like steel profiles or materials according to Eurocode catalogues?

For creating the Code itself, we recommend the third party Code Checking framework. It makes it very easy to build and check codes.

Here is a set of useful references:

- Review the BIM and Beam blog post providing a detailed description of the
  [Structural Analysis and Code Checking Toolkit](http://bimandbeam.typepad.com/bim_beam/2013/04/structural-analysis-and-code-checking-toolkit-2014-on-autodesk-exchange-apps.html).
- Install the
  [Structural Analysis Toolkit](http://apps.exchange.autodesk.com/RVT/Detail/Index?id=appstore.exchange.autodesk.com%3astructuralanalysisandcodecheckingtoolkit2014%3aen) that
  activates the Results Storage and the code checking framework.
- Look at the Code Checking SDK provided within the Revit SDK that you already installed.
  The folder 'Structural Analysis SDK' contains all the supporting material and full documentation.
- Finally, here is another interesting discussion describing a
  [first look at the Structural Analysis SDK](http://adndevblog.typepad.com/aec/2013/05/structural-analysis-sdk-first-look.html)
  to review.

#### Tabbing Between Applications on Mac

I like my screen uncluttered.

Under Windows, I practiced for decades minimising windows and then restoring them again using Alt-Tab.

Unfortunately, on the Mac, this does not work equally well.

If you minimise a window using the round yellow '-' minus button in the upper left hand corner of each window, it disappears happily into the dock and cannot be restored by ⌘-Tab.

I found two solutions to this: the wrong way and the right way.

The wrong way involves some keyboard acrobatics: ⌘-Tab back to the desired window, and then, keeping the ⌘ key pressed all the time, press the Option key as well. Let go of the ⌘ key, and, lo and behold, the window reappears.

Here are some pretty old discussion threads explaining that trick:

- [Shortcut to 'un-minimize' something from the dock](https://discussions.apple.com/thread/1615114)
- [How to 'unhide' apps](https://discussions.apple.com/message/3065018)

This is the wrong way, however, because it perpetuates my unhealthy Windows habits and mentality.

I just realised that a much more efficient solution is to avoid minimising the windows, and hiding them instead.

Mac OS provides a shortcut key for that, ⌘-H.

When hidden, a window is restored just as I want simply using ⌘-Tab.

I have been trying to retrain myself for almost a week now, and am still mostly falling back into the old habit...