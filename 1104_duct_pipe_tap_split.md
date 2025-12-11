---
ai_optimized: true
author: Jeremy Tammik
complexity_score: 2.4
content_type: qa
optimization_date: '2025-12-11T11:44:15.311719'
original_url: https://thebuildingcoder.typepad.com/blog/1104_duct_pipe_tap_split.html
post_number: '1104'
reading_time_minutes: 3
series: mep
slug: duct_pipe_tap_split
source_file: 1104_duct_pipe_tap_split.htm
tags:
- elements
- family
- parameters
- revit-api
- views
- walls
- mep
title: Daylighting Extension and Splitting with Taps
word_count: 688
---

### Daylighting Extension and Splitting with Taps

Here is a quick update on the status of the
[daylighting analysis](#2) technology preview and some helpful advice for anyone trying to
[split a duct or pipe with taps](#3).

#### Daylighting Analysis

The
[Daylighting Analysis for Revit](http://thebuildingcoder.typepad.com/blog/2013/12/starting-to-clean-up-for-the-break.html#4) technology
preview free analysis period has been extended until March 31 or until a commercial version becomes available, whichever comes first.

![Daylighting Analysis for Revit](img/daylighting_analysis_preview.png)

Find out more and download the updated plug-in from the
[Daylighting Analysis for Revit](https://beta.autodesk.com/callout/?callid=976EB2E05A1D46B5818473BDF2FAABC5) Autodesk
Labs page.

#### Splitting a Duct or Pipe with Taps

**Question:** I would like to split a Duct or a Pipe in a way that retains the intact Revit functionality.

The user interface provides a command to split walls via Modify > Split Element > Split with Gap.

How can I programmatically achieve something similar to split a duct or a pipe?

**Answer:** The Duct and Pipe classes do not provide any built-in split functionality.

Here are a discussion on
[splitting a duct or pipe](http://thebuildingcoder.typepad.com/blog/2012/03/split-a-duct-or-pipe.html) from
2012 and
[an older one](http://thebuildingcoder.typepad.com/blog/2009/04/revit-api-cases-1.html#2) from
2009.

**Response:** Splitting a pipe by shortening it and then creating a new one for the other end works fine for simple cases.

However, what can I do if there are taps connected to it?

In that case, shortening the pipe will remove the connection between the tap and the pipe and cause errors.

Creating a new pipe at the old one's location will not automatically reattach the tap.

So how can I split a pipe or a duct that has taps attached to it?

**Answer:** As always, the Revit API will not provide any additional functionality beyond what you have in the user interface.

From the UI standpoint, one would need to:

1. Remove the taps.
   In some cases, when you shorten a duct, Revit will throw an error saying that you need to delete the tap.
   In other cases it will not.
   Regardless, you need to get rid of them to create new ones.
   So, of course, you'll want to remember their type, location, orientation, etc.
2. Reconnect the branch to the newly created duct that supplements the old shortened one.
   In the UI, the 'extend' command will insert a tap for you.
   The API will as well, as we saw in some of the various cases during the development of the
   [simpler rolling offset](http://thebuildingcoder.typepad.com/blog/2014/01/newelbowfitting-easily-places-rolling-offset-elbow-fittings.html).

**Response:** I accept that the API will not provide any additional functionality beyond what you have in the user interface.

However, besides the 'Split with Gap' functionality for walls, the UI also provides a 'Split Element' command that does work for Ducts and Pipes.
Are you saying this functionality is available programmatically?
If so, where can I find it, please?

Secondly, you suggest to remove the taps and then recreate them.

In the UI, the extend method will only create the tap again if it is also defined in the DuctType and if the Preferred Junction Type is set to Tap.
Furthermore, another thing that needs to be taken into account is that certain parameters might be set for the duct or pipe. and all of these parameter values will need to be copied to the newly created segment as well.

The function 'Split Elements' provided by the user interface does all of this automatically: it will just split the Duct and add a union family in between.
That is exactly what I would like to achieve programmatically as well.

**Answer:** Thank you for the clarification and sorry for the detour.

I am sorry to say that you are right in not finding that access and this functionality is currently not available through the API, so
I submitted a wish list item on your behalf for it.