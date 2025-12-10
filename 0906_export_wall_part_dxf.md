---
post_number: "0906"
title: "Export Wall Parts Individually to DXF"
slug: "export_wall_part_dxf"
author: "Jeremy Tammik"
tags: ['csharp', 'elements', 'family', 'filtering', 'geometry', 'python', 'references', 'revit-api', 'schedules', 'selection', 'sheets', 'transactions', 'views', 'walls']
source_file: "0906_export_wall_part_dxf.htm"
original_url: "https://thebuildingcoder.typepad.com/blog/0906_export_wall_part_dxf.html"
---

﻿

### Export Wall Parts Individually to DXF

I am back from my vacation.
It was a wonderful break, and I feel ready and happy to get back to grips with everyday life and work again.

A query from a colleague caught up with me already on the way back, on exporting a whole bunch of selected wall panel parts to individual DXF files.
That gave me something nice and interesting to fiddle with during the nightly train ride up from Napoli to Milano and led to the following issues:

- [Exporting individual compound wall gyp wallboard parts](#2)
- [Handling and dismissing a warning message](#3)
- [Adding support for both pre- and post- part selection](#4)
- [Handling temporary transactions and regeneration](#5)

Before getting to that, though, here are a few final notes on my last vacation day in Napoli.

#### Last Day in Napoli

I had some wonderful pastry in the
[Gran Bar Riviera](http://www.tripadvisor.com/Restaurant_Review-g187785-d1129216-Reviews-Gran_Bar_Riviera-Naples_Province_of_Naples_Campania.html) pasticceria
on the Riviera di Chiaia.
Incidentally, an old palazzo partially collapsed there last week, just a few hundred meters away from where I was staying.

![Collapsed palazzo in Riviera di Chiaia](file:///j/photo/jeremy/2013/2013-03-08_napoli/p1000534_collapsed_house_cropped.jpg)

![Shoemaker Gabriele](file:///j/photo/jeremy/2013/2013-03-08_napoli/p1000586_gabriele_shoemaker_cropped.jpg)

![Gabriele and Jeremy](file:///j/photo/jeremy/2013/2013-03-08_napoli/p1000587_gabriele_jeremy.jpg)

Further, I took a pair of old shoes to Gabriele, an extremely sweet and happy 86 years old shoemaker in the Spanish quarter.
He repaired them and they look better than new now.
He also invited us to coffee and told me his life story.
If you ever need a pair of shoes fixed, be sure to look him up in Vico Lungo del Gelso, 108, I-80134 Napoli :-)

Two final parting pictures capturing some of the decrepit charm of Naples...

![Impressions from the Spanish quarter](file:///j/photo/jeremy/2013/2013-03-08_napoli/p1000580.jpg)

![Impressions from the Spanish quarter](file:///j/photo/jeremy/2013/2013-03-08_napoli/p1000592.jpg)

Anyway, now I am back at work now again, and we return to the Revit API and my nocturnal dabbling on the train.

#### Exporting Individual Compound Wall Gyp Wallboard Parts

This query came in from my colleague William Spier, MEP & Design to Fabrication SME at Autodesk
([YouTube](http://www.youtube.com/AutodeskMEPTechs),
[Revit Family Jewels](http://familyjewels.typepad.com)).
He says:

**Question:** I would like to customise two pretty easy things.

The scenario is a compound wall on which I divide the gyp wallboard into standard (4’x8’) sheet size parts.
I need to export each of those sheets/parts to DXF format, but even though I have divided them into parts, Revit still groups them and exports them as one single DXF file – not as separate ones, like I need.
The only way around this is to isolate each part one at a time and export each one individually, which is WAY too laborious.

So I need to customize the export process so that:

1. Each part exports as a separate part, even though I globally selected them.
2. Each part exports named by its own unique code/identifier – whatever coding keeps each one distinct in the parts schedule.

Here is a snapshot from a simple sample project:

![Sample wall parts](img/sample_wall_parts.png)

**Answer:** A first search for related API methods turn up the following potentially useful items:

- Document.Export( string, string, ICollection<ElementId>, DXFExportOptions )- View.IsolateElementTemporary( ElementId )- Part.ParentDividedElementId property (obsolete)- Part.GetSourceElementOriginalCategoryIds method (obsolete)- Part.GetSourceElementIds method

Skipping the obsolete ones, it turns out to be really easy to implement a first working version that I packaged in an external command named ExportWallboard.

I first check that a valid document is provided and a 3D view is active that we can use for isolating and exporting each part:
```csharp
  if( null == doc )
  {
    ErrorMsg( "Please run this command in a valid"
      + " Revit project document." );
    return Result.Failed;
  }

  View view = doc.ActiveView;

  if( null == view || !( view is View3D ) )
  {
    ErrorMsg( "Please run this command in a valid"
      + " 3D view." );
    return Result.Failed;
  }

  // Define the list of views to export,
  // including only the current 3D view

  List<ElementId> viewIds = new List<ElementId>( 1 );
  viewIds.Add( view.Id );
```

Basically, all that is required is to follow the steps suggested above:

- Isolate each part
- Define the DXF filename
- Call the export method

```csharp
  Element e = doc.GetElement( id );

  Debug.Assert( e is Part,
    "expected parts only" );

  Part part = e as Part;

  ICollection<LinkElementId> lids
    = part.GetSourceElementIds();

  Debug.Assert( 1 == lids.Count,
    "unexpected multiple part source elements." );

  LinkElementId lid = lids.First<LinkElementId>();
  ElementId hostId = lid.HostElementId;
  ElementId linkedId = lid.LinkedElementId;
  ElementId parentId = hostId;
  ElementId partId = e.Id;

  filename = string.Format( "{0}\_{1}",
    parentId, partId );

  view.IsolateElementTemporary( partId );

  doc.Export( \_folder, filename, viewIds, opt );
```

The IsolateElementTemporary method requires a transaction, so even though the whole operation is theoretically read-only, we still need to specify manual transaction mode for this command.
We encapsulate the isolate and export method in a transaction that is later rolled back, so the model ends up unchanged after all.

### Handling and Dismissing a Warning Message

However, a complication arises:

Calling the Export method with an isolated element in the current view displays a task dialogue warning message:
![Exporting a view with isolated element warning](img/export_with_isolate_warning.png)

I repeatedly discussed how to automatically handle messages like this in the past, e.g. to
[detach a workset](http://thebuildingcoder.typepad.com/blog/2012/10/detach-workset-and-taskdialog-command-link-order.html).

To be notified of this message, we subscribe to the DialogShowing event:

```csharp
  uiapp.DialogBoxShowing
    += new EventHandler<DialogBoxShowingEventArgs>(
      OnDialogBoxShowing );
```

Do not forget to unsubscribe afterwards.
In this case, we can do so at the end of the command.

To ensure that the unsubscription is performed whatever happens, regardless of any potential errors, I encapsulate the whole operation in a try statement and unsubscribe in its 'finally' clause.
```csharp
  try
  {
    // Register event handler for
    // "TaskDialog\_Really\_Print\_Or\_Export\_Temp\_View\_Modes"
    // dialogue

    uiapp.DialogBoxShowing
      += new EventHandler<DialogBoxShowingEventArgs>(
        OnDialogBoxShowing );

    DXFExportOptions opt = new DXFExportOptions();

    string filename;

    using( Transaction tx = new Transaction( doc ) )
    {
      tx.Start( "Transaction Name" );

      foreach( ElementId id in ids )
      {
        Element e = doc.GetElement( id );

        // . . .

        view.IsolateElementTemporary( partId );

        doc.Export( \_folder, filename, viewIds, opt );
      }

      // We do not commit the transaction, because
      // we do not want any modifications saved.
      // The transaction is only created and started
      // because it is required by the
      // IsolateElementTemporary method.
      // Since the transaction is not committed,
      // the changes are automatically discarded.

      //tx.Commit();
    }
  }
  finally
  {
    uiapp.DialogBoxShowing
      -= new EventHandler<DialogBoxShowingEventArgs>(
        OnDialogBoxShowing );
  }
  return Result.Succeeded;
```

Initially, I did not yet know exactly which dialogue id to use to identify this specific message.
I therefore first implemented a dummy DialogBoxShowing event handler, triggered the event, and determined the dialogue id to use by re-running the command and looking at the event handler argument in the debugger.

As it turns out, the required dialogue id in our case is "TaskDialog\_Really\_Print\_Or\_Export\_Temp\_View\_Modes".
We wish to retain the temporary isolate mode and export, i.e. select the second command link option.
Therefore, the final dialogue box showing event handler implementation becomes:
```csharp
void OnDialogBoxShowing(
  object sender,
  DialogBoxShowingEventArgs e )
{
  TaskDialogShowingEventArgs e2
    = e as TaskDialogShowingEventArgs;

  if( null != e2 && e2.DialogId.Equals(
    "TaskDialog\_Really\_Print\_Or\_Export\_Temp\_View\_Modes" ) )
  {
    e.OverrideResult(
      (int)TaskDialogResult.CommandLink2 );
  }
}
```

### Adding Support for both Pre- and Post- Part Selection

In the initial implementation, I just went ahead and used the pre-selected set of parts defined by the user before launching the external command, accessible via the uidoc.Selection.Elements collection.

However, it is much more user friendly to also support post-selection.
For that case, it is also useful to implement a selection filter to simplify easy mass selection of the parts.

In order to handle the elements identically regardless of whether they were pre- or post-selected, I convert the selection set to a list of element ids in both cases.

For the pre-selection, I iterate over the pre-selected elements and test each one as follows:
```csharp
  // Iterate over all pre-selected parts

  List<ElementId> ids = null;

  Selection sel = uidoc.Selection;

  if( 0 < sel.Elements.Size )
  {
    foreach( Element e in sel.Elements )
    {
      if( !( e is Part ) )
      {
        ErrorMsg( "Please pre-select only gyp wallboard"
          + " parts before running this command." );
        return Result.Failed;
      }

      Part part = e as Part;

      ICollection<LinkElementId> lids
        = part.GetSourceElementIds();

      if( 1 != lids.Count )
      {
        ErrorMsg( "Gyp wallboard parts has multiple"
          + " source elements." );
        return Result.Failed;
      }

      LinkElementId lid = lids.First<LinkElementId>();
      ElementId hostId = lid.HostElementId;
      ElementId linkedId = lid.LinkedElementId;
      ElementId parentId = hostId;
      ElementId partId = e.Id;

      // Determine parent category

      Element parent = doc.GetElement( parentId );
      Category cat = parent.Category;

      ICollection<ElementId> cids
        = part.GetSourceElementOriginalCategoryIds();

      if( 1 != cids.Count )
      {
        ErrorMsg( "Gyp wallboard parts has multiple"
          + " source element categories." );
        return Result.Failed;
      }

      ElementId cid = cids.First<ElementId>();

      //cat = doc.GetElement( id ) as Category;

      // Expected parent category is OST\_Walls

      BuiltInCategory bic
        = (BuiltInCategory)cid.IntegerValue;

      if( BuiltInCategory.OST\_Walls != bic )
      {
        ErrorMsg( "Pleqase pre-select only "
          + " gyp wallboard parts." );
        return Result.Failed;
      }

      if( null == ids )
      {
        ids = new List<ElementId>( 1 );
      }

      ids.Add( partId );
    }

    if( null == ids )
    {
      ErrorMsg( "Please pre-select only gyp wallboard"
        + " parts before running this command." );
      return Result.Failed;
    }
  }
```

The category of the source elements can be determined using the Part.GetSourceElementOriginalCategoryIds method, which unsurprisingly turns out to be OST\_Walls.

I use that to check the part source element category like this in the final selection filter implementation:
```python
class WallPartSelectionFilter : ISelectionFilter
{
  public bool AllowElement( Element e )
  {
    bool rc = false;

    Part part = e as Part;

    if( null != part )
    {
      ICollection<ElementId> cids
        = part.GetSourceElementOriginalCategoryIds();

      if( 1 == cids.Count )
      {
        ElementId cid = cids.First<ElementId>();

        BuiltInCategory bic
          = (BuiltInCategory)cid.IntegerValue;

        rc = ( BuiltInCategory.OST\_Walls == bic );
      }
    }
    return rc;
  }

  public bool AllowReference( Reference r, XYZ p )
  {
    return true;
  }
}
```

For the post-selection, the selection filter ensures that no inappropriate parts can be selected, so no additional scan is required to test the references returned by the PickObjects method.

I can use a generic LINQ method to convert the collection of resulting references to the list of element ids:
```csharp
  // If no parts were pre-selected,
  // prompt for post-selection

  if( null == ids )
  {
    IList<Reference> refs = null;

    try
    {
      refs = sel.PickObjects( ObjectType.Element,
        new WallPartSelectionFilter(),
        "Please select wall parts." );
    }
    catch( Autodesk.Revit.Exceptions
      .OperationCanceledException )
    {
      return Result.Cancelled;
    }
    ids = new List<ElementId>(
      refs.Select<Reference, ElementId>(
        r => r.ElementId ) );
  }
```

Here is
[ExportWallboard03.zip](zip/ExportWallboard03.zip) including the complete source code, Visual Studio solution and add-in manifest of the current state of this external command.

There are obviously still some implementation details to iron out.

### Handling Temporary Transactions and Regeneration

Luckily, I took a closer look at the generated DXF output files before letting this command loose on humanity.

To my horror, I discovered that all files generated except the first one contain no geometry.

As it turns out, there is an issue with the regeneration and view settings in my original implementation.

After quite a bit of experimentation, I found out that I can successfully generate the individual files if I add the following steps:

- Add code to switch off the first temporary isolation before applying the next one.
- Commit the transaction to temporaily isolate each part before calling Export.
- This requires encapsulating all the temporary transactions in a group and rolling back the entire group instead of the individual transactions, as described for the
  [temporary transaction trick touchup](http://thebuildingcoder.typepad.com/blog/2012/11/temporary-transaction-trick-touchup.html).
- Separate the code to switch off the last temporary isolation and applying the new one into separate transactions.

The resulting loop exporting all the parts ends up looking like this:

```csharp
  using( TransactionGroup txg = new TransactionGroup( doc ) )
  {
    txg.Start( "Export Wall Parts" );

    foreach( ElementId id in ids )
    {
      Element e = doc.GetElement( id );

      Debug.Assert( e is Part,
        "expected parts only" );

      Part part = e as Part;

      ICollection<LinkElementId> lids
        = part.GetSourceElementIds();

      Debug.Assert( 1 == lids.Count,
        "unexpected multiple part source elements." );

      LinkElementId lid = lids.First<LinkElementId>();
      ElementId hostId = lid.HostElementId;
      ElementId linkedId = lid.LinkedElementId;
      ElementId parentId = hostId;
      ElementId partId = e.Id;

      filename = string.Format( "{0}\_{1}",
        parentId, partId );

      if( view.IsTemporaryHideIsolateActive() )
      {
        using( Transaction tx = new Transaction( doc ) )
        {
          tx.Start( "Disable Temporary Isolate" );

          view.DisableTemporaryViewMode(
            TemporaryViewMode.TemporaryHideIsolate );

          tx.Commit();
        }

        Debug.Assert( !view.IsTemporaryHideIsolateActive(),
          "expected to turn off temporary hide/isolate" );
      }

      using( Transaction tx = new Transaction( doc ) )
      {
        tx.Start( "Export Wall Part "
          + partId.ToString() );

        view.IsolateElementTemporary( partId ); // requires transaction

        //List<ElementId> unhideIds = new List<ElementId>( 1 );
        //unhideIds.Add( partId );
        //view.UnhideElements( unhideIds );

        //doc.Regenerate(); // this is insufficient

        tx.Commit();
      }

      doc.Export( \_folder, filename, viewIds, opt );

      // We do not commit the transaction group,
      // because no modifications should be saved.
      // The transaction group is only created and
      // started to encapsulate the transactions
      // required by the IsolateElementTemporary
      // method. Since the transaction group is not
      // committed, the changes are automatically
      // discarded.

      //txg.Commit();
    }
  }
```

Here is
[ExportWallboard04.zip](zip/ExportWallboard04.zip) including the source code, Visual Studio solution and add-in manifest of the updated version.

I certainly expect more implementation details to crop up that need ironing out.

For instance, I could imagine adding some code to delete the PCP files that are generated together with the DXF output.
Or is there any reason to keep them?

I can also imagine that the part identification needs improving.
Currently, the command simply uses the part and its source element ids.
Maybe the part unique id would be better, or the parent element id needs checking, or is unnecessary snd can be completely removed.

Anyway, for a first stab, this implementation now looks pretty good to me.