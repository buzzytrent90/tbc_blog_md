---
ai_optimized: true
author: Jeremy Tammik
complexity_score: 2.4
content_type: qa
optimization_date: '2025-12-11T11:44:13.997771'
original_url: https://thebuildingcoder.typepad.com/blog/0467_refresh_referencing_sheet.html
post_number: '0467'
reading_time_minutes: 3
series: views
slug: refresh_referencing_sheet
source_file: 0467_refresh_referencing_sheet.htm
tags:
- csharp
- parameters
- revit-api
- sheets
- transactions
- views
title: Refresh Referencing Sheet Parameter Display
word_count: 566
---

### Refresh Referencing Sheet Parameter Display

Here is question and solution by Piotr Zurek on the problem of the stale data displayed by the "Referencing Sheet" parameter value in a section head:

**Question:** Is it possible to refresh the "Referencing Sheet" parameter value displayed in a section head programmatically?

I have a section viewport inserted in a sheet.
The parameter in question shows the no. of the first sheet with a view that shows the section head for that section.
Now, this section head is hidden and the parameter should change to the name of the next sheet that contains a view with this section's head.

I can't seem to be able to do it through the API.

Manually it can be done by activating the section view, modifying it slightly and then deactivating.
Another way is to change the discipline of that view.

In a few words, my plug-in does the following:

It requires that the user selects a single section viewport in a sheet.
Then it gets the name of that sheet and then it searches for all plan views (ViewPlan) that include a section head for this section.
It also checks if this section head is visible and possibly in which sheet this plan view was inserted.
Then the user can define in which views the section head should be visible (using check boxes next to the view's name).

Usually the user will pick just one view in a certain sheet to force the "referencing sheet" parameter to be changed to that sheet.

After that it would be nice if the parameter was updated automatically, but it doesn't seem to be.

The problem seems to be known to Revit people at Autodesk, as you can see from
[the last paragraph of this blog post](http://revitclinic.typepad.com/my_weblog/2010/03/sort-order-referencing-sheet-parameters-dependent-views.html).

You made the following suggestions to resolve this:

1. Regenerate the document using the Document.Regenerate method.- Start and commit a transaction using Transaction.Start and Commit.- Select the section view programmatically and apply some minimal change to it, e.g. moving it by a zero length vector with the Document.Move method.- Save the document using the Document.Save method.

I tried them all out, but they do not help:

1. Document.Regenerate – I tried that as the first solution. Didn't do anything.- All the changes to the model are enclosed between Transaction.Start and Transaction.Commit.- Tried that, but it didn't work.- I actually tried saving and reopening the model and it still doesn't work.

**Answer:** I managed to find a workaround that works for me.
In order to force the update, I change the discipline of my view back and forth:
```csharp
void UpdateReferencingSheet(
  ViewSection selectedViewport )
{
  BuiltInParameter bip
    = BuiltInParameter.VIEW\_DISCIPLINE;

  Parameter discipline
    = selectedViewport.get\_Parameter( bip );

  int disciplineNo = discipline.AsInteger();

  Document doc = selectedViewport.Document;

  Transaction transaction = new Transaction( doc );

  if( TransactionStatus.Started
    == transaction.Start( "Updating the model" ) )
  {
    switch( disciplineNo )
    {
      case 1:
        discipline.Set( 2 );
        break;

      default:
        discipline.Set( 1 );
        break;
    }
  }
  discipline.Set( disciplineNo );
  transaction.Commit();
}
```

Many thanks to Piotr for this solution!

I implemented a new Building Coder sample command CmdUpdateReferencingSheet to test this method.

Here is
[version 2011.0.77.0](zip/bc_11_77.zip)
of The Building Coder samples including the complete source code and Visual Studio solution with the new command.