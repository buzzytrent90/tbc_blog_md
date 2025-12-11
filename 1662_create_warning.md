---
ai_optimized: true
author: Jeremy Tammik
complexity_score: 3.6
content_type: qa
optimization_date: '2025-12-11T11:44:16.485910'
original_url: https://thebuildingcoder.typepad.com/blog/1662_create_warning.html
post_number: '1662'
reading_time_minutes: 3
series: general
slug: create_warning
source_file: 1662_create_warning.md
tags:
- doors
- elements
- filtering
- revit-api
- rooms
- sheets
- views
- windows
title: Create Warning
word_count: 555
---

### Creating a Warning Using the Failure API
The [Failure API](http://thebuildingcoder.typepad.com/blog/2010/04/failure-api.html) was
introduced a long time ago, back in 2010, for Revit 2011.
Time enough to forget that it enables both failure definition and handling capabilities:
- The ability to define and post failures from within API code when a user-visible problem has occurred.
- The ability to respond to failures posted by Revit and by API code through code in your application.
Luckily, Frank [@Fair59](https://forums.autodesk.com/t5/user/viewprofilepage/user-id/2083518) Aarssen is
there to help and remind us, in his answer to
the [Revit API discussion forum](http://forums.autodesk.com/t5/revit-api-forum/bd-p/160) thread
on [how to create a warning](https://forums.autodesk.com/t5/revit-api-forum/how-to-create-a-warning/m-p/8058817):
\*\*Question:\*\* I've a routine which moves elements around, but on some occasions, it comes across an element that can't be moved.
I can get the routine to filter them out, so it's not an issue, but I'd like to create a warning message for the user.
I can do this with the `TaskDialog` class, and it has rather a large number of options
– cf., the [Revit UI API labs in 2_Revit_UI_API](https://github.com/jeremytammik/AdnRevitApiLabsXtra) –
however, it cannot be used in a modeless manner, so it always stops the user and forces them to hit OK.
Really, I'd like to create a non-critical Warning message that warns the user, but does not stop them working – the same sort of warning you get if you delete a room from the model:
![Warning](img/warning.jpeg)
I can't see a way of creating this in the API – can it be done?
\*\*Answer:\*\* For a non-Revit-API approach, you could indeed use a modeless Windows form and close it automatically after a specified time period. Here is a description
of [showing and closing modeless a form](https://drive-cad-with-code.blogspot.com/2014/02/showing-and-closing-modeless-formdialog.html).
It was written with AutoCAD in mind, and exceptionally applies to Revit as well, since it has nothing to do with either of them, really.
Much better, though, is to implement exactly what you ask: the Revit API does indeed provide complete support for creating your own modeless warning messages just like you require.
To do so, you can define your own failure, and let Revit display it.
First, you create a `FailureDefinitionId` and `FailureDefinition`:
```csharp
// Create youw own new failure definition Ids
Guid guid1 = new Guid(
"d0cf2412-08fe-476b-a6e6-e9a98583c52c" );
FailureDefinitionId m_idWarning
= new FailureDefinitionId( guid1 );
// Create failure definitions and add resolutions
FailureDefinition m_fdWarning
= FailureDefinition.CreateFailureDefinition(
m_idWarning, FailureSeverity.Warning,
"Textvalue is changed for all instances "
+ "in textchain" );
m_fdWarning.SetDefaultResolutionType(
FailureResolutionType.SetValue );
```
Then, you can display the warning:
```csharp
FailureMessage fm = new FailureMessage(
ExternalApplication.m_idWarning );
doc.PostFailure( fm );
```
These steps are also demonstrated in the Revit SDK ErrorHandling and PerformanceAdviserControl samples, in the files Command.cs and FlippedDoorCheck.cs, respectively.
The failure is displayed as a Revit warning as depicted above.
You can totally ignore it, and continue working.
One possible caveat: it will probably be displayed when control reverts back to Revit (not sure).
Many thanks to Frank for his invaluable help!