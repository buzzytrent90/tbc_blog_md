---
ai_optimized: true
author: Jeremy Tammik
complexity_score: 3.9
content_type: qa
optimization_date: '2025-12-11T11:44:15.404853'
original_url: https://thebuildingcoder.typepad.com/blog/1148_handle_warning_failure.html
post_number: '1148'
reading_time_minutes: 4
series: general
slug: handle_warning_failure
source_file: 1148_handle_warning_failure.htm
tags:
- csharp
- doors
- family
- filtering
- levels
- revit-api
- rooms
- selection
- views
- windows
title: On Handling Warnings and Failures
word_count: 820
---

### On Handling Warnings and Failures

Many aspects of
[detecting and handling dialogues and failures](http://thebuildingcoder.typepad.com/blog/about-the-author.html#5.32) have
been discussed in the past.

Let's look at this area again, based on two recent questions, a basic and an advanced one, and also mention some Revit 2015 enhancements to the
[Revit Developer Centre](http://www.autodesk.com/developrevit):

- [Programmatic handling of warning messages](#2)
- [Resolving failures using the Failure API](#3)
- [Revit 2015 enhancements to the Revit developer centre](#4)

#### Programmatic Handling of Warning Messages

**Question:** Is it possible with the Revit MEP 2011 API to handle warning messages?

For instance, when batch processing files, if a warning message occurs, i.e. a modal dialogue halting the program is displayed, I would like to intercept the event, capture the warning message and disable the display of the modal dialog.

Furthermore, if warning messages can be handled, can error messages be handled via the API as well? For instance, if an error occurs within the application while processing files, can I programmatically determine the error number, take appropriate action and then cancel the modal error message that would normally be displayed interactively?

**Answer:** Yes, this handling of warning messages as you describe is possible, and more so, absolutely common, in three different levels of increasing complexity and power:

- Simple: DialogBoxShowing event
- More complex and powerful: Failure API
- Massive, beyond the Revit API: Windows Hook

Here is a grand overview over all three approaches to
[detect and handle warning and error messages and failures](http://thebuildingcoder.typepad.com/blog/about-the-author.html#5.32).

The answer is to the second question is yes as well, and the explanation is also included in that material.

For handling errors gracefully, the Failure API is probably the most suitable approach, as illustrated by the following situation.

#### Resolving Failures Using the Failure API

**Question:** In my add-in, editing a door family via Revit API throws an error message.

The failure API cannot resolve the error and the error dialog is shown to the user.

In Revit 2014, I was able to silently resolve the error, and no message was shown to the user.

**Answer:** You need to call the SetProcessingResult method on the event argument and specify ProceedWithCommit like this:

```csharp
    public void DoFailureProcessing(
      object sender,
      FailuresProcessingEventArgs args )
    {
      FailuresAccessor fa = args.GetFailuresAccessor();

      // Inside event handler, get all warnings

      IList<FailureMessageAccessor> a
        = fa.GetFailureMessages();

      int count = 0;

      foreach( FailureMessageAccessor failure in a )
      {
        TaskDialog.Show( "Failure",
          failure.GetDescriptionText() );

        fa.ResolveFailure( failure );

        ++count;
      }

      if( 0 < count
        && args.GetProcessingResult()
          == FailureProcessingResult.Continue )
      {
        args.SetProcessingResult(
          FailureProcessingResult.ProceedWithCommit );
      }
    }
```

The code snippet in the Revit API developer guide on
[handling failures](http://help.autodesk.com/view/RVT/2015/ENU/?guid=GUID-52A45CC1-3BB4-48B4-BFC7-F6F8666C2AA4) does
not make that call to SetProcessingResult, because it is only deleting warnings.

That works fine, because deleting a warning is an "instant" operation: leaving the event result unchanged (it defaults to "continue") produces the desired result – there are no warnings left to display.

Errors are not removed by resolveErrors – only subsequent regeneration will actually remove them.
The removal may also fail – resolution is not guaranteed to succeed.
So a call to SetProcessingResult with FailureProcessingResult.ProceedWithCommit is required.

#### Revit 2015 Enhancements to the Revit Developer Centre

As you certainly know, the most up-to-date version of the Revit SDK is always provided in the
[Revit Developer Centre](http://www.autodesk.com/developrevit).

Besides that, several other important learning resources provided there have now also been updated for Revit 2015:

- [Revit 2015 SDK (Update April 11, 2014)](http://images.autodesk.com/adsk/files/REVIT2015SDK0.msi) (msi – 242779Kb)
- [Revit API Developer Guide](http://www.autodesk.com/revitapi-help) online on Autodesk Help to learn from and contribute to
- DevTV Introduction to Revit 2015 Programming Part 1 – a short video tutorial demonstrating the basic steps to start developing with the Revit .NET API –
  [View online](http://download.autodesk.com/media/adn/RevitDevTV2015Recording/Revit%20DevTV%202015.html)
  | [Download](http://download.autodesk.com/media/adn/Revit_2015_DevTV_EN.zip)
- DevTV Introduction to Revit 2015 Programming Part 2 – a short video tutorial demonstrating selection and filtering API through a Room Renumbering application –
  [View online](http://www.youtube.com/watch?v=zL8pQRJbcyA)
  | [Download](http://download.autodesk.com/media/adn/DevTVRevitNet2015.zip)
- [Revit 2015 API Labs](http://images.autodesk.com/adsk/files/Revit2015APITraining.zip)

I just noticed that the Revit Developer Centre link to the Developers Guide is currently still pointing to the Revit 2014 version.

As I mentioned when discussing
[What's New in the Revit 2015 API](http://thebuildingcoder.typepad.com/blog/2014/04/whats-new-in-the-revit-2015-api.html),
the updated
[Revit 2015 API Developers Guide](http://help.autodesk.com/view/RVT/2015/ENU/?guid=GUID-F0A122E0-E556-4D0D-9D0F-7E72A9315A42) is
already online as well, and the link on the Developer Centre needs to be updated to point to that instead.