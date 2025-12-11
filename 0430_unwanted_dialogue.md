---
ai_optimized: true
author: Jeremy Tammik
complexity_score: 5.4
content_type: code_example
optimization_date: '2025-12-11T11:44:13.933679'
original_url: https://thebuildingcoder.typepad.com/blog/0430_unwanted_dialogue.html
post_number: '0430'
reading_time_minutes: 4
series: general
slug: unwanted_dialogue
source_file: 0430_unwanted_dialogue.htm
tags:
- csharp
- elements
- family
- python
- revit-api
- transactions
- walls
- windows
title: Suppress Unwanted Dialogue
word_count: 849
---

### Suppress Unwanted Dialogue

I am posting this in advance from Pattaya before leaving laptop-less for Cambodia, so that you have a little something for the duration of my absence.

Before I left on vacation, we had several queries on suppressing unwanted dialogues, and we also looked at this issue in some depth discussing
[editing elements inside groups](http://thebuildingcoder.typepad.com/blog/2010/08/editing-elements-inside-groups.html).
Here are some further examples of the questions and solutions we found:

**Question:** I am importing a DWG containing AutoCAD solids into a Revit document.
The DWG imports just fine but we always get a dialogue stating that "Some ACIS objects could not be imported. To import them, use AutoCAD to convert them into polymesh objects and reimport."
Oddly, when launching the same process manually in the Revit user interface, we get the same successful end result without any dialogue.
Is there a way to suppress the unwanted dialogue?

**Answer:** As mentioned in the
[last post](http://thebuildingcoder.typepad.com/blog/2010/08/editing-elements-inside-groups.html),
there are several different possible ways to suppress unwanted dialogues in Revit 2011.

1. The old method that was already available in Revit 2010 was to use the
   [DialogBoxShowing event](http://thebuildingcoder.typepad.com/blog/2009/06/autoconfirm-save-using-dialogboxshowing-event.html),
   which can be used to dismiss a dialogue programmatically.- The Revit 2011 API also includes a comprehensive
     [failure processing API](http://thebuildingcoder.typepad.com/blog/2010/04/failure-api.html),
     which can do much more than the DialogBoxShowing event.- If all else fails and you just want to get rid of the dialogue box by any means, you can also use the Windows API to detect when it is displayed and dismiss it using my
       [dialogue clicker application](http://thebuildingcoder.typepad.com/blog/2009/10/dismiss-dialogue-using-windows-api.html) to
       suppress it.

**Question:** I used the following code to suppress a dialogue box that appears when reloading modified families:
```csharp
void myDialogBoxShowing(
  object sender,
  DialogBoxShowingEventArgs e )
{
  TaskDialogShowingEventArgs e1
    = e as TaskDialogShowingEventArgs;

  if( e1 != null )
  {
    if( e1.DialogId.Equals(
      "TaskDialog\_Family\_Already\_Exists" ) )
    {
      e1.OverrideResult(
        ( int ) TaskDialogResult.CommandLink1 ); // 1001
    }
  }
}
```

How can I suppress this dialogue box that is displayed when I delete a linked RVT file?

![Delete linked RVT file dialogue](img/unwanted_dlg_linked_rvt.jpg)

**Answer:** I would suggest using the Failure API to suppress that dialogue.

**Response:** Thanks you for your quick reply.
It works fine and is easier to control.

I still have to figure out how to override with another option instead of the default one, in case of a warning dialogue box with more than one button.
In my case, I want to pick "Remove Link" instead of "OK".
Here is the code:
```python
public class WarningSwallower
  : IFailuresPreprocessor
{
  public FailureProcessingResult
    PreprocessFailures( FailuresAccessor a )
  {
    IList<FailureMessageAccessor> failures
      = a.GetFailureMessages();

    foreach( FailureMessageAccessor f in failures )
    {
      //if (f.GetSeverity().ToString() == "Warning")
      //a.DeleteWarning(f);

      FailureDefinitionId id
        = f.GetFailureDefinitionId();

      if( id == BuiltInFailures.LinkFailures
        .DeleteLinkSymbolPrompt )
      {
        // only default option being choosen,
        // not good enough!
        a.DeleteWarning( f );
      }
    }
    return FailureProcessingResult.Continue;
  }
}
```

Is there a way within the Failure API to suppress the warning and specifying a non-default option?

The closest possibility I see is to set the FailureResolutionType.
I tried that, but it does not work.

**Answer:** Could you please try to use the ResolveFailure method to mimic clicking the 'Remove Link' button?
Here is a code snippet showing how to resolve a failure, excerpted from the Revit 2011 SDK ErrorHandling sample:
```csharp
public FailureProcessingResult ProcessFailures(
  FailuresAccessor failuresAccessor )
{
  IList<FailureMessageAccessor> fmas
    = failuresAccessor.GetFailureMessages();

  if( fmas.Count == 0 )
  {
    return FailureProcessingResult.Continue;
  }

  String transactionName
    = failuresAccessor.GetTransactionName();

  if( transactionName.Equals(
    "Error\_FailuresProcessor" ) )
  {
    foreach( FailureMessageAccessor fma in fmas )
    {
      FailureDefinitionId id
        = fma.GetFailureDefinitionId();

      if( id == Command.m\_idError )
      {
        failuresAccessor.ResolveFailure( fma );
      }
    }
    return FailureProcessingResult
      .ProceedWithCommit;
  }
  else
  {
    return FailureProcessingResult.Continue;
  }
}
```

**Response:** I just tested using the ResolveFailure method, and it still seems not to be working, i.e. the dialogue box is still showing up using this code:
```csharp
public FailureProcessingResult PreprocessFailures(
  FailuresAccessor a )
{
  IList<FailureMessageAccessor> failures
    = a.GetFailureMessages();

  foreach( FailureMessageAccessor f in failures )
  {
    FailureDefinitionId id
      = f.GetFailureDefinitionId();

    if( id == BuiltInFailures.LinkFailures
      .DeleteLinkSymbolPrompt )
    {
      // only default option being choosen,
      // not good enough!
      //a.DeleteWarning(f);

      // cannot suppress dialogue box
      a.ResolveFailure( f );
    }
  }
  return FailureProcessingResult.Continue;
}
```

DeleteWarning is working, the only problem is that it only uses the default OK option.
Certainly there must be a way to specify another option when deleting a warning?

**Answer:** Please try to use a class derived from IFailureProcessor, and implement its ResolveFailure method appropriately.

If all else fails, a workaround would be to delete the remaining RVT model link via the API after you dismiss the warning using the default option.

**Correction:** Aleksey Moiseyev very friendlily pointed out that the ResolveFailure method shown above is still not working, i.e. the dialogue box is still showing up using the code shown.

The method should be returning the FailureProcessingResult ProceedWithCommit, not Continue. The latter allows the error dialog to appear as usual, whereas the former suppresses it.

Thank you very much, Aleksey, for pointing this out, and sorry for the erroneous original post!