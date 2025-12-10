---
post_number: "0501"
title: "VSTA to Stay and Updater to Go"
slug: "vsta_and_updater"
author: "Jeremy Tammik"
tags: ['elements', 'levels', 'references', 'revit-api', 'transactions', 'vbnet', 'views', 'walls']
source_file: "0501_vsta_and_updater.htm"
original_url: "https://thebuildingcoder.typepad.com/blog/0501_vsta_and_updater.html"
---

### VSTA to Stay and Updater to Go

As you may have heard from Kean, we have safely
[arrived in Munich, Germany](http://through-the-interface.typepad.com/through_the_interface/2010/12/arrived-in-mnchen.html) for
the next developer conference, the biggest one in our European series.
Actually, he only talks about his own arrival, but our arrival worked all right as well.

One topic that we are discussing at these conferences is cloud computing, so it was a funny coincidence to run into a nice big tangible cloud when we went out to have a dinner with our developer partners in
[Heaven 23](http://www.heaven23.se)
in Gothenburg before departing for Munich:

![Heaven 23 cloud](img/heaven23_cloud.jpg)

The restaurant is on the 23rd floor of the
[Gothia Towers](http://sv.wikipedia.org/wiki/Hotel_Gothia_Towers) with
a nice view down over
[Liseberg amusement park](http://en.wikipedia.org/wiki/Liseberg):

![Heaven 23 view](img/heaven23_view.jpg)

As you can see, the cloud can be accessed through reflection as well as the web service :-)

Meanwhile, here are some questions that cropped up several times at
[Autodesk University](http://thebuildingcoder.typepad.com/blog/2010/12/the-revit-api-track-at-au-2010.html) and
were mentioned during some of our meetings here in Europe as well:

- Can we rely on [VSTA remaining](#1)? We heard that VBA for AutoCAD went away; is there any risk of that happening with VSTA for Revit as well?- Can I use the Dynamic Model Update technology if I wish to distribute a model to customers without my add-in? [A warning message is displayed](#2). This is incomprehensible and worrying for the uninitiated user. How can I handle this gracefully, and best of all suppress it completely?- Can I [automate the removal of an updater](#3) reference?

#### VSTA to Stay

VSTA is an acronym for Visual Studio Tools for Applications.
It is included in every Revit installation, so every user has access to it.
It provides an IDE or Integrated Development Environment for exploring the API and creating macros.
The code is almost completely compatible with the .NET API we use for external commands and applications defined in standard add-ins.

Macros written in VSTA can easily be moved to an external add-in, and vice versa.

One advantage of VSTA is the ability to edit and continue, which is currently not supported by the Visual Studio environment when creating standard external add-ins.

Some developers and users have been careful about jumping onto VSTA, because it is a Microsoft product, analogous to the old VBA, Visual Basic for Applications.
VBA has been around for quite a while and was included in AutoCAD.
It is now being phased out, since Microsoft is reducing support for it.

As said, VSTA is here to stay.
Here is a statement on this by Anthony Hauck, Revit Design Product Line Manager:

Autodesk is happy to announce an agreement with Microsoft to include the Visual Studio Tools for Applications (VSTA) development environment in all Revit products for the foreseeable future.
Previous communications touching on the status of VSTA in Revit expressed some uncertainty as to its inclusion with future versions, but with the agreement concluded earlier this year we are pleased that VSTA will continue to be an important development option for users of the Revit API.

#### Removing an Updater Reference

One of the most powerful new API features in Revit 2011 is the Dynamic Model Update or DMU framework.
It enables you to register an updater to be triggered by certain events, and also to react on these events within the same transaction that triggered them, as demonstrated by the
[DynamicModelUpdate and DistanceToSurfaces SDK samples](http://thebuildingcoder.typepad.com/blog/2010/04/element-level-events.html#2) and Saikat's
[Structural DMU sample](http://thebuildingcoder.typepad.com/blog/2010/08/structural-dynamic-model-update-sample.html).

However, when a Revit model is modified by an updater and then distributed to a client without the add-in installed, a warning message is displayed which is not comprehensible for an uninitiated user.

For example, I implemented a minimal DMU application and ran the updater in a model, saved it, and reopen it later without the add-in defining the updater loaded.
The following message appeared:

![Updater warning](img/updaterwarning.png)

For completeness' sake, here is the
[DummyUpdater](zip/DummyUpdater.zip)
application that I used to cause this message to appear.

How can I remove the reference to this updater from the model, in order to distribute it to a customer with no access to my add-in?

The very simplest answer is to tell you user to click on 'Do not warn about this updater again and continue working with the file', save the model, and continue working with it as normal.

However, many developers would prefer this message not to appear at all.

To achieve this, simply click on 'Do not warn about this updater again and continue working with the file' and save the model yourself and then pass it on to the customer.

The next time it is opened, the warning will no longer appear, because the reference to the updater has been removed.

Obviously, you need to well understand the consequences of removing this reference, and your application needs to be prepared to handle the situation in case it is ever loaded into the model again at a later stage, when changes may have been applied in its absence.

#### Automating Updater Removal

If you downloaded my sample DMU application above, you may have noted an empty add-in skeleton in it named RemoveUpdater.
I was originally planning to implement an add-in which automates the process of opening a Revit model containing an updater reference to remove, clicking on the option mentioned above, and saving and closing the file.

When I looked at this a bit more closely, I thought that I could save myself some work by creating a journal file to do the job instead.
It works really well.
I did the following:

1. Copied my Revit document to C:\tmp\wall\_updater\_triggered.rvt.- Started up Revit.- Opened the file.- The warning message appears, I selected the appropriate option.- Saved the file to a new path wall\_updater\_triggered\_removed.rvt.- Quit Revit.- Copied the resulting journal file journal.0575.txt to journal\_remove\_updater.txt.

As you probably know, the journal files are by default located in the Journals subfolder of the Revit installation directory.

Now I can automate the removal of the updater from any Revit model by creating a batch file which:

1. Copies the model to C:\tmp\wall\_updater\_triggered.rvt.- Starts up Revit with the journal file, e.g. using the command line listed below.- Moves the updated model from C:\tmp\wall\_updater\_triggered\_removed.rvt to its final destination.

Here is the command line to drive Revit using the journal file; it is one single (long) line:

```
"C:\Program Files\Autodesk\Revit Architecture 2011\Program\Revit.exe"
  "C:\Program Files\Autodesk\Revit Architecture 2011\Journals
    \journal_remove_updater.txt"
```

Obviously this could also be more cleanly implemented using the API, but that is left as an exercise for the reader.

Best regards to all from Munich!