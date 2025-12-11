---
ai_optimized: true
author: Jeremy Tammik
complexity_score: 2.8
content_type: qa
optimization_date: '2025-12-11T11:44:15.728762'
original_url: https://thebuildingcoder.typepad.com/blog/1305_va3c_guid_driving.html
post_number: '1305'
reading_time_minutes: 7
series: general
slug: va3c_guid_driving
source_file: 1305_va3c_guid_driving.htm
tags:
- family
- parameters
- revit-api
- views
title: Duplicate Add-In GUID and Driving Revit from Outside
word_count: 1302
---

### Duplicate Add-In GUID and Driving Revit from Outside

Lots of stuff going on; I have a hard time keeping up.

Due to popular request, I updated the [RvtVa3c](https://github.com/va3c/RvtVa3c) readme.

I am continuing to answer issues on the
[Revit API discussion forum](http://forums.autodesk.com/t5/revit-api/bd-p/160) as
well as ADN cases.

I am struggling to find time for [The 3D Web Coder](http://the3dwebcoder.typepad.com)...

One ADN case that I answered fits in very nicely with a forum suggestion provided by Arnošt Löbel.

Here are my topics for now:

- [Duplicate add-in GUID](#2)
- [Driving Revit from outside](#3)
- [Popular AppStore add-ins in France](#4)

#### Duplicate Add-In GUID

A developer worked on generating a suitable JSON representation of a Revit building model and raised the following query:

**Question:** I tried directly uploading an .rvt file and it gave an "unsupported file error".

I tried reading the code for the viewer; it uses a three.js loader function, but three.js doesn't support .rvt, and the samples shown are .rvt.js which are json files of the Revit models.

How were the Revit files converted to JSON?

**Answer:**
You have to install and run the RvtVa3c JSON exporter add-in in Revit:

- [Discussion](http://thebuildingcoder.typepad.com/blog/va3c)
- [Source](https://github.com/va3c/RvtVa3c)

**Response:**
I was able install the plugin, with minor changes.
I suggest a readme along with the repository.
I tried to write one, which is incomplete, as it doesn't give details about all the dependencies, as to the Revit version, the .NET version etc.

Also, when I installed the plugin, it conflicted with an already installed plugin by CL3VER, stating:

- External Tools – Duplicate AddInId – ... The AddInId Node is a GUID used to identify the add-in application. Revit already has an application registered with the indicated GUID.

This is the error message displayed:

![CL3VER exporter causes duplicate add-in GUID error](img/va3c_cl3ver_duplicate_add_in_guid_error.png)

Guide me, if this can be patched easily.

**Answer:**
Thank you for your report and good suggestions!

Congratulations on getting it to work!

I integrated the readme material you provided into a new section on setup and installation in the main
[RvtVa3c GitHub repository readme](https://github.com/va3c/RvtVa3c).

The issue you report with CL3VER means that the guys who implemented that copied my RvtVa3c add-in and the GUID in my add-in manifest.

They even copied my idea of replacing the character 'e' by a numeric '3' in their name, which I proposed for the vA3C project to ensure it is easily found and identified by a global Internet search.

I am obviously honoured   :-)

I will tell them about that. It is trivial to replace: simply generate a new GUID and edit to add-in manifest file RvtVa3c.addin to use that instead.

In this case, however, the CL3VER guys ought to replace theirs, since I originally generated mine myself!

Please refer to this more detailed discussion on
[the add-in GUID and my Guidizer tool](http://thebuildingcoder.typepad.com/blog/2010/04/addin-manifest-and-guidize.html) that
I sometimes use for this purpose.

**To CL3VER:**
Dear Madam or Sir,

A developer reported a duplicate Revit add-in GUID conflict when using your cl3ver add-in together with my RvtVa3c application:

![CL3VER exporter causes duplicate add-in GUID error](img/va3c_cl3ver_duplicate_add_in_guid_error.png)

Please modify your CL3VER Revit add-in manifest to use an own GUID instead of my RvtVa3c one.

Thank you!

#### Driving Revit from Outside

**Question:**
I am using Revit Architecture 2015.

I want to read parameters in a Revit family without opening Revit using the .NET API.

Please guide me in doing this.

**Answer:**
In future, please perform a minimal Internet search before submitting an ADN case.

The answer to your question appears in a large number of discussions that you will see if you search for
"[revit api without opening](https://duckduckgo.com/?q=revit+api+without+opening)".

For more detailed and targeted information, please refer to The Building Coder topic group on
[driving Revit from outside](http://thebuildingcoder.typepad.com/blog/about-the-author.html#5.28).

After this initial answer, I just happened to note a thread on the Revit API discussion that addresses the exact same issue you raise,
[calling a Revit plug-in without Revit open](http://forums.autodesk.com/t5/revit-api/calling-revit-plug-in-without-revit-open/m-p/5574286),
and suggests a workaround that may be of interest to you as well:

**Question:**
I'm writing software that extracts and analyses information from Revit using a Revit API plug-in, but as it stands the plug-in has to be activated from the External Tools menu in Revit, and ideally I'd like the start point to be within my application, instead of Revit.

I understand that Revit probably has to be open to run the plug-in within its own runtime, but I'm wondering if it's possible to start a hidden instance of Revit and call the plug-in like that instead of having to explicitly start Revit and select the plug-in from the menu. Is something like this possible?

**Answer by Arnošt Löbel**, Sr. Principal Engineer, Autodesk Revit R&D:

As you've found out, there is no direct support out of the box for what you want.
A Revit instance must be running in order for an add-in to be executed.
However, you can possibly sort of work around it if you figure out how to instantiate a Revit process that is hidden.
Perhaps you can launch it in a separate desktop, one that is set to be out of screen.
Even then, though, it is still a limitation that you cannot invoke your add-in from outside of Revit.
The call must always come from within Revit.
External command is out of the question, I presume, because there will be no user to click it and invoke it.
However, I assume you do not need an external command anyway.
My guess is that you want to do something to a model, and for that all you ought to need is one of the document events, most likely DocumentOpened. Or, as an alternative, you can do your task in an ApplicationIitialized event or in OnStartup method your external application implements.
The following steps suggest one possible workflow:

1. Implement your functionality in an add-in (external application) and have it installed in Revit.
2. Design a simple standalone application (exe) that starts Revit in a separate process.
3. Use the standalone application to start Revit hidden somehow (or not hidden, if you do not care).
4. In the OnStartup method your application implements, subscribe to ApplicationInitialized event.
5. When your add-in receives the event, open the document you want to analyse.
6. When the document is opened, perform the analysis; output whatever you need to output.
7. Close the document.
8. Notify the standalone application that your task is finished. Perhaps some kind of inter-process messaging can be used – I suggest making it asynchronous.
9. Exit the ApplicationInitialized event handler.
10. When the standalone application receives the inter-process message from your add-in, kill the Revit process.

#### Popular AppStore Add-ins in France

To wrap up, here is an example of the Exchange Apps Community at work.

A French Revit user commented: "thank you – i reported about your software, recommending
[three free, easy and useful Revit add-ins](http://www.hexabim.com/fr/blog/trois-plugins-gratuits-faciles-et-utiles-pour-revit) on
my blog."

Check out the website link and the additional comments from other customers.

This may be the ideal chance to test your built-in browser web page translation functionality   :-)