---
ai_optimized: true
author: Jeremy Tammik
complexity_score: 3.3
content_type: qa
optimization_date: '2025-12-11T11:44:15.302627'
original_url: https://thebuildingcoder.typepad.com/blog/1099_failure_api.html
post_number: '1099'
reading_time_minutes: 3
series: general
slug: failure_api
source_file: 1099_failure_api.htm
tags:
- csharp
- elements
- revit-api
- rooms
- transactions
title: Skip Invalid Element Generation Using Failure API
word_count: 543
---

### Skip Invalid Element Generation Using Failure API

We already took a couple of looks at the Failure API in the past, e.g.:

- [Failure API](http://thebuildingcoder.typepad.com/blog/2010/04/failure-api.html)
- [Failure API Take Two](http://thebuildingcoder.typepad.com/blog/2010/11/failure-api-take-two.html)
- [Failure Rollback](http://thebuildingcoder.typepad.com/blog/2012/04/failure-rollback.html)

Here is a slightly more convoluted situation that motivates revisiting the topic from a new angle, raised and solved by Stephen Faust of
[Revolution Design, Inc.](http://www.revolutiondesign.biz/) together
with Arnošt Löbel of the Revit API development team, who also invested significant time and research in the issue:

**Question:** I am generating some floor objects from rooms in a project.
In certain cases the sketch is not valid because of some oddities in the rooms.
This is OK if I can just delete or not create the floors and hopefully not show a dialogue to the user.
Without any failure handling it gives the user the 'can't make extrusion' error, but you can hit delete elements and be on your way.
I thought I could do this with a failures preprocessor, but it doesn't seem to be working the way I hoped.
Here is the functional part of my class that inherits IFailuresPreProcessor:

```csharp
  IList<FailureMessageAccessor> fmas
    = failuresAccessor.GetFailureMessages();

  if( fmas.Count == 0 )
  { return FailureProcessingResult.Continue; }

  foreach( FailureMessageAccessor fma in fmas )
  { failuresAccessor.ResolveFailure( fma ); }

  return FailureProcessingResult.ProceedWithCommit;
```

When I use this with the transaction, it still gives me an error dialog, only this time it shows (Deleted) next to all of the elements.
The only thing I can do is cancel, which is actually worse, because then you don't get any of the floors.
What am I doing wrong here?
What do I need to do to just handle the dialog and tell it to remove those floors that aren't valid?

**Answer:** What you probably need to do is to set the resolution on each failure.

There are also some checks required to avoid attempting to resolve a failure that may not be resolved (anymore).

I suggest the following starting point for an approach, plus further experimenting with it:

- Test if invalid elements (floors) can actually be deleted by calling FailuresAccessors.IsElementsDeletionPermitted.
- Possibly, for each failure, it may be tested if its resolution by deleting the invalid elements is permitted. The method to do so is FailuresAccessors.IsFailureResolutionPermitted taking a FailureResolutionType.DeleteElements argument.
- Assuming you can, failures should be resolved by FailuresAccessors.ResolveFailures.
- I am also guessing that before returning ProceedWitCommit, FailuresAccessors.IsTransactionBeingCommitted is tested.
- Return ProceedWitCommit.

**Response:** Thanks for the tips.

Here is what I ended up doing that seems to be working:

```csharp
  if( failuresAccessor.GetTransactionName()
    == "Create Temporary Roofs" )
  {
    IList<FailureMessageAccessor> fmas
      = failuresAccessor.GetFailureMessages();

    if( fmas.Count == 0 )
    { return FailureProcessingResult.Continue; }

    foreach( FailureMessageAccessor fma in fmas )
    {
      FailureSeverity s = fma.GetSeverity();
      if( s == FailureSeverity.Warning )
      { failuresAccessor.DeleteWarning( fma ); }
      else if( s == FailureSeverity.Error )
      {
        if( failuresAccessor
          .IsElementsDeletionPermitted(
            fma.GetFailingElementIds().ToList() ) )
        {
          failuresAccessor.DeleteElements(
            fma.GetFailingElementIds().ToList() );
        }
        failuresAccessor.ResolveFailure( fma );
      }
    }
    return FailureProcessingResult.ProceedWithCommit;
  }
  else
  { return FailureProcessingResult.Continue; }
```

Basically I separated out warnings from errors.
Then warnings can be deleted, errors can be processed and delete elements can be directly called for specific elements causing the error.