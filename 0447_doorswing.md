---
ai_optimized: true
author: Jeremy Tammik
complexity_score: 2.5
content_type: qa
optimization_date: '2025-12-11T11:44:13.960989'
original_url: https://thebuildingcoder.typepad.com/blog/0447_doorswing.html
post_number: '0447'
reading_time_minutes: 3
series: general
slug: doorswing
source_file: 0447_doorswing.htm
tags:
- csharp
- doors
- geometry
- revit-api
- transactions
title: DoorSwing Fix
word_count: 642
---

### DoorSwing Fix

I am spending this week in the UK.
We are having a long-awaited EMEA DevTech team meeting today.
We finally succeeded in creating a possibility for all ADN engineers working in the European region to get together face to face.
I met some of my colleagues for the first time, for instance Marat, who is based in Moscow, and Kristine from Toronto (not really part of Europe).
The rest of the week I will be attending an internal training course, so expect a bit less blogging activity than usual.
I do hope to be able to present at least something anyway, though, beginning with the following little item that cropped up a week or two ago:

Eric Wigren of [CAD Specialisten](http://www.cadspecialisten.se) brought a little problem with the DoorSwing SDK sample to my attention.

Since the fix is simple, I will publish my version of it here, in case anyone needs it urgently, like Eric did.
The public SDK will also be fixed, of course, the next time an update becomes available.

It started out with Eric's raising the following issue:

**Question:** I am trying to use the Revit 2011 SDK DoorSwing sample.
If I flip a door in the user interface without clicking 'Update Door Properties' and then save the project, I get the following error message:

"Attempt to modify the model outside of transaction."

Do you have a solution for this?

I need the DoorSwing add-in because in Sweden when ordering doors for a new building, the factory needs to know for every door whether its leaf is on the left or right.
This swing direction feature is available in AutoCAD Architecture, but not in Revit.

**Answer:** I looked at this sample and discovered two unrelated problems with it:

- The external application is missing from the add-in manifest file.- The external application call-back event handlers lack the required transactions.

Put differently, with a little more detail:

- The add-in manifest file lists the three commands, but not the external application implemented by this sample.- The external application implements event handlers for the DocumentSaving and DocumentSavingAs, and these do not set up the transactions that they require.

The latter issue is causing the problem you describe.

Adding the external application to the add-in manifest can be achieved by including the following lines to DoorSwing.addin immediately before the final line with the closing RevitAddIns tag (copy to a text editor to see the truncated lines):
```csharp
  <AddIn Type="Application">
    <Name>DoorSwing</Name>
    <Assembly>C:\...\DoorSwing.dll</Assembly>
    <ClientId>ad10ff26-34a5-4186-8142-3619264e0833</ClientId>
    <FullClassName>Revit.SDK.Samples.DoorSwing.CS.ExternalApplication</FullClassName>
  </AddIn>
```

To add the transaction required to the event handlers, I implemented a common helper function 'f' called by both of them in the module ExternalApplication.cs.
The resulting code looks like this:
```csharp
/// <summary>
/// This event is fired whenever a document
/// is saved. Update door's information
/// according to door's current geometry.
/// </summary>
/// <param name="sender">The source of the event.</param>
/// <param name="args">An DocumentSavingEventArgs
/// that contains the DocumentSaving event data.</param>
private void DocumentSavingHandler(
  Object sender,
  DocumentSavingEventArgs args )
{
  f( args.Document );
}

/// <summary>
/// This event is fired whenever a document
/// is saved as. Update door's information
/// according to door's current geometry.
/// </summary>
/// <param name="sender">The source of the event.</param>
/// <param name="args">An DocumentSavingAsEventArgs
/// that contains the DocumentSavingAs event data.</param>
private void DocumentSavingAsHandler(
  Object sender,
  DocumentSavingAsEventArgs args )
{
  f( args.Document );
}

void f( Document doc )
{
  string message = "";

  Transaction t = new Transaction( doc,
    "Door Swing" );

  t.Start();

  try
  {
    DoorSwingData.UpdateDoorsInfo(
      doc, false, false, ref message );

    t.Commit();
  }
  catch( Exception ex )
  {
    t.RollBack();

    message = ex.Message;
  }
  if( 0 < message.Length )
  {
    MessageBox.Show( message, "Door Swing" );
  }
}
```

Here is
[DoorSwingUpdateJeremy.zip](zip/DoorSwingUpdateJeremy.zip) containing the two modified files DoorSwing.addin and ExternalApplication.cs.