---
ai_optimized: true
author: Jeremy Tammik
complexity_score: 7.6
content_type: qa
optimization_date: '2025-12-11T11:44:14.457398'
original_url: https://thebuildingcoder.typepad.com/blog/0718_revitwebcam_2012.html
post_number: 0718
reading_time_minutes: 10
series: general
slug: revitwebcam_2012
source_file: 0718_revitwebcam_2012.htm
tags:
- csharp
- elements
- geometry
- levels
- references
- revit-api
- walls
title: Revit Webcam 2012
word_count: 1927
---

### Revit Webcam 2012

I recently discussed the
[elimination of warnings](http://thebuildingcoder.typepad.com/blog/2012/01/eliminating-compiler-warnings-and-deprecated-calls.html) about
use of deprecated API methods from The Building Coder samples.
In a similar vein, and for similar reasons, I also decided to migrate to Revit 2012 and clean up the
[RevitWebcam](http://thebuildingcoder.typepad.com/blog/2010/06/display-webcam-image-on-building-element-face.html) sample
that I implemented to demonstrate the use of the
[Idling event](http://thebuildingcoder.typepad.com/blog/idling) when
it was first introduced in the Revit 2011 API.

Please note that this sample is intentionally not related to any BIM specific workflow.
A webcam just provides an obvious and accessible sample of a continuously updated and changing graphical dataset driven by something outside of Revit.
I use this to demonstrate that we can sample this data and create a graphical display of it within the Revit model with no user interaction whatsoever.
Other add-ins such as the MultithreadedCalculation Revit SDK sample simulate an externally changing data source.
A real-world example might be displaying the graphical analysis visualisation of an asynchronous cloud-based calculation.

I migrated the 2011 version to 2012 in several steps:

1. [Flat migration](#1) and [adaption of Idling event handler](#2)- Removal of [obsolete API warnings](#3)- Final [clean-up](#4)

#### Flat migration

I could have simply copied the existing project and source code files and modified them one by one as required for the 2012 API.

Instead, I used a different approach that I find more efficient, since it requires less manual steps:
I created a completely new project using the Visual Studio
[Revit add-in wizard](http://thebuildingcoder.typepad.com/blog/2011/10/product-and-add-in-wizard-updates.html) and
then copied the relevant bits of source code that I wished to preserve from the original project to the new one.

Since the add-in wizard creates all the required settings, including generating and installing the add-in manifest, all in one single click, this approach requires less manual intervention.
As an additional bonus, it also guarantees that all the application settings are up-to-date and nothing is forgotten.

Here is
[RevitWebcam2012\_1\_initial\_migration.zip](zip/RevitWebcam2012_1_initial_migration.zip) containing
the result of the flat migration, which compiles all right but exhibits one little problem when executed:

#### Adaption of the Idling Event Handler

The flat migration enables the add-in to be compiled successfully.
On running it, though, an exception is immediately thrown on the first call to the Idling event handler, since the type of the 'sender' argument changed from Application in the Revit 2011 API to UIApplication in the Revit 2012 API.

I presented a fully transparent method to support both of these versions simultaneously in the
[modeless loose connector navigator update](http://thebuildingcoder.typepad.com/blog/2011/08/modeless-loose-connector-navigator-update.html#2) plus its
[subsequent enhancement](http://thebuildingcoder.typepad.com/blog/2011/09/yet-another-modeless-update.html).

The code is still making use of some obsolete API calls, though, so I continued the migration to remove those as well:

#### Removal of Obsolete API Warnings

Compiling the flat migrated code described above results in the following warnings (copy and paste to an editor to see the untruncated lines):

```
------ Rebuild All started: Project: RevitWebcam, Configuration: Debug Any CPU ------
C:\a\src\revit\webcam\RevitWebcam\RevitWebcam\RevitWebcam\Command.cs(232,11): warning CS0618: 'Autodesk.Revit.DB.Analysis.AnalysisDisplayLegendSettings.SetTextTypeId(Autodesk.Revit.DB.ElementId, Autodesk.Revit.DB.Document)' is obsolete: 'this method will be obsolete from 2012.'
C:\a\src\revit\webcam\RevitWebcam\RevitWebcam\RevitWebcam\Command.cs(398,11): warning CS0618: 'Autodesk.Revit.DB.Analysis.SpatialFieldManager.UpdateSpatialFieldPrimitive(int, Autodesk.Revit.DB.Analysis.FieldDomainPoints, Autodesk.Revit.DB.Analysis.FieldValues)' is obsolete: 'This method is obsolete in Revit 2012; use the overload accepting the result index instead.'
C:\a\src\revit\webcam\RevitWebcam\RevitWebcam\RevitWebcam\Command.cs(386,23): warning CS0618: 'Autodesk.Revit.DB.Reference.GeometryObject' is obsolete: 'Property will be removed. Use Element.GetGeometryObjectFromReference(Reference) instead'
C:\a\src\revit\webcam\RevitWebcam\RevitWebcam\RevitWebcam\Command.cs(431,31): warning CS0618: 'Autodesk.Revit.DB.Reference.Element' is obsolete: 'Property will be removed. Use Document.GetElement(Reference) instead'
C:\a\src\revit\webcam\RevitWebcam\RevitWebcam\RevitWebcam\Command.cs(434,31): warning CS0618: 'Autodesk.Revit.DB.Reference.GeometryObject' is obsolete: 'Property will be removed. Use Element.GetGeometryObjectFromReference(Reference) instead'

Compile complete -- 0 errors, 5 warnings
```

To analyse the exact changes made to fix these, unpack the initial migration and the final version below in two directories side-by-side and use a file comparison tool.

#### Final Clean-up

Final clean-up included changes to comments and line breaks.

Here is
[RevitWebcam2012\_3\_final\_cleanup.zip](zip/RevitWebcam2012_3_final_cleanup.zip) containing
the cleaned up code with all obsolete API usage removed.

Here is a snapshot of RevitWebcam up and running in Revit 2012:

![RevitWebcam in Revit 2012](img/webcam_on_wall_2012.png)

The image is upside-down now, which does not worry me, since this is all about getting it to display and update automatically at regular intervals, and all else is of no concern to us.

**Disclaimer:** This application does not adhere to the
[recommended rules](http://thebuildingcoder.typepad.com/blog/2011/09/yet-another-modeless-update.html) for
interacting with an external asynchronous process, e.g. a modeless dialogue, so this code should not be used as is for production use.

The code presented here is just a test showing some aspects of possible uses of the Revit API functionality.
It is by no means a production-level solution.
This actually applies to all the code provided in this blog anyway.
In truth, it applies just as well to many a commercial application that I find myself forced to use, much to my chagrin :-)

---

#### RevitWebcam 2013

I went through pretty similar steps to the ones required to
[migrate RevitWebcam to 2012](http://thebuildingcoder.typepad.com/blog/2012/02/revit-webcam-2012.html)
to make the next step from Revit 2012 to 2013:

1. [obsolete API usage.](#1>Flat migration</a> creating a new project and copying the code to retain.
   <li>Remove the warning about <a href=)

The reason for a ofthis is the following part, which is where it get interesting:

3. Make use of the new Revit 2013 Idling features to control [external event](#3>Idling event repetition</a>.
   <li>Convert the implementation using the Idling event to an <a href=) instead.

The last step is by far the most interesting and useful, and one of the very important new features in Revit 2013.

What is an external event?

That merits a blog post of its own.

#### 1. Initial Migration

Here is
[RevitWebcam2013\_1\_initial\_migration.zip](zip/RevitWebcam2013_1_initial_migration.zip) containing
the result of the flat migration of the Revit 2012 code.

I created a brand new Revit 2013 add-in using the
Revit 2013 add-in wizard and
copied the Revit 2012 source code into it.
As I explained last time around, creating a new project using the ne-click wizard and copying the existing code into it is less work than copying the entire existing project and trying to modify all the required settings.

The add-in compiles and executes all right straight out of the box, but produces the following compilation warnings:

```
'Autodesk.Revit.DB.Document.get_Element(Autodesk.Revit.DB.ElementId)' is obsolete:
This method has been obsoleted. Use GetElement() instead.
C:\a\src\revit\webcam\RevitWebcam\RevitWebcam2013_2_removed_warnings\RevitWebcam\RevitWebcam\Command.cs	417
C:\a\src\revit\webcam\RevitWebcam\RevitWebcam2013_2_removed_warnings\RevitWebcam\RevitWebcam\Command.cs	496
C:\a\src\revit\webcam\RevitWebcam\RevitWebcam2013_2_removed_warnings\RevitWebcam\RevitWebcam\Command.cs	499
```

Before we do anything to make use of the new API functionality, let's clean up thos warnings.

#### 2. Remove Obsolete API Calls

The one single obsolete API usage warning that the code generates can be easily removed.

They are generated by calls to the document get\_Element method taking an element id argument.
The strange naming of this method wth a lower-case first letter and an underscore is due to the fact that it is actually declared as a property named simply 'Element', and is used as such in VB.
In C#, however, due to the fact that it takes an argument, the property is translated to a method and the 'get\_' prefix added.

This is similar to the differences between C# and VB accessing the [Document.Elements method](http://thebuildingcoder.typepad.com/blog/2009/09/document-elements.html).

Here is
[RevitWebcam2013\_2\_removed\_warnings.zip](zip/RevitWebcam2013_2_removed_warnings.zip) containing
the cleaned-up code.

#### 3. Idling Event Repetition

By simply migrating the existing Revit 2012 Idling event code to Revit 2013, we are implicitly acception the new default Event repetition behaviour:

In the default mode, a single raise of the event is made each time Revit begins an idle session.
When the user is active in the Revit user interface, idle sessions begin whenever the mouse stops moving for a moment or when a command completes.
However, if the user is not active in the user interface at all, Revit may not invoke additional idling sessions for quite some time; this means that an add-in subscribing to the Idling event may not be able to take advantage of time when the user leaves the machine completely idle for a period of time.

The old behaviour, which was the only available optin in Revit 2012, is now the non-default mode:

To request the non-default mode, an add-in must force Revit to keep the idling session open and to make repeated calls to its event handler.
In this mode, Revit will continue to make Idling calls even if the user is totally inactive.
However, this can result in performance degradation for the system on which Revit is running, because the CPU remains fully engaged in serving Idling events during the Revit application's idle time.

In the case of RevitWebcam, if you wish to see the webcam image continue bein updated repeatedly on the selected BIM element face even if no user interaction is taking place, the non-default behaviour must be requested.

This is done by calling the new SetRaiseWithoutDelay method on the IdlingEventArgs instance passed into the Idling event handler.

The call must be made each time the Idling event handler is called.
Revit reverts to the default Idling repetition mode if this method is not called every time in your callback.

In RevitWebcam, I defined a new static global Boolean variable '\_raise\_without\_delay' with which I can control this behaviour at runtim from the debugger.
In the beginning of the OnIdling, I added the following lines:
```csharp
  if( \_raise\_without\_delay )
  {
    e.SetRaiseWithoutDelay();
  }
```

#### 4. External Event

By far the largest change in this series, and actually the whole point of this exercise, as well as the original moticvation to
[port to 2012](http://thebuildingcoder.typepad.com/blog/2012/02/revit-webcam-2012.html) at all.

I implemented a new WebcamEvent external event class, removed the Idling event handler, and moved most of its code into the new class instead.

I ended up significantly cleaning up and simplifying my GreyscaleBitmapData class to avoid storing the bitmap and instead just calculate and retain the data that I need to display the AVF results.
This also helps avoid an error saying that the bitmap is already in use.
That also provided an opportunity to very simply flip the image upside-down so that it appears the right way up in Revit again.

```

```