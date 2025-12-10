---
post_number: "1132"
title: "Transaction Group and Regeneration for InstanceVoidCutUtils"
slug: "instance_void_cut"
author: "Jeremy Tammik"
tags: ['csharp', 'elements', 'family', 'filtering', 'levels', 'parameters', 'references', 'revit-api', 'selection', 'sheets', 'transactions']
source_file: "1132_instance_void_cut.htm"
original_url: "https://thebuildingcoder.typepad.com/blog/1132_instance_void_cut.html"
---

### Transaction Group and Regeneration for InstanceVoidCutUtils

We looked at a nice example of using
[Boolean operations and InstanceVoidCutUtils](http://thebuildingcoder.typepad.com/blog/2011/06/boolean-operations-and-instancevoidcututils.html) back
in the year 2011, cutting out part of a beam using a void cutter family.

I tried to make use of this again in a
[recent Revit API training](http://thebuildingcoder.typepad.com/blog/2014/02/different-revit-api-aspects-and-features.html) and was
somewhat shocked to discover that it did not work as expected.

I checked all the prerequisites were fulfilled.

#### Void Cutter Family

First of all, the void cutter family must have its 'Cut with Voids When Loaded' parameter checked:

![Cut with void property](img/instance_void_cut_cut_with_voids.png)

If this is not the case, the void is not an 'Unattached' void, and it will throw an exception saying "The element is not a family instance with an unattached void that can cut. Parameter name: cuttingInstance".

The cutting symbol has three angle parameters associated with it, which can be used to rotate it around various axes.

#### CutBeamWithVoid Method

The sample code implements a method CutBeamWithVoid taking two arguments: the beam family instance and the void cutter family symbol. It places three cutter instances along the top of the beam, modifies their angle parameters and applies the cut.

Initially, the three cutting instances are placed along the top face of the beam.

Their angle parameter values are changed from the default zero to 45 degrees, causing two of them to partially cut the beam.

The call to the InstanceVoidCutUtils.AddInstanceVoidCut method still threw a message saying "The element is not a family instance with an unattached void that can cut. Parameter name: cuttingInstance".

When the call to InstanceVoidCutUtils.AddInstanceVoidCut was commented out, you could see that the cutter family instances were correctly positioned:

![Cutting instances placed with no cut applied](img/instance_void_cut_no_cut.png)

Selecting the elements manually and combining them using the join and cut tools in the UI Modify tab successfully completed the operation:

![Cutting instances placed with cut applied](img/instance_void_cut_applied.png)

The manual operation should be completely analogous to InstanceVoidCutUtils.

What was wrong?

#### Need for Regeneration

After struggling for a while with that, I gave up and asked my colleagues.

Scott Conover answered: could the solution be as simple as Regenerating after the creation of the instance? Your sample code does not do that.

Normally, before any part of a newly created element is read or used, Revit requires a regeneration to take place.

Note that for that reason, if you have a significant number to place and join in a loop, you may want to structure the code that it places them all, then regenerates, and then creates all of cuts/joins.

Jim Jia added: Scott is right – I verified doc.Regenerate can solve the issue.

Besides, I also noticed if the cutter family was already loaded and some cutter instances were ever created, the call to doc.Regenerate is not required.

Shockingly enough (for me personally), that really is the simple solution to this problem.

We have talked about this numerous times in the past, and yet I missed it myself this time around, again:

- [Manual regeneration option danger](http://thebuildingcoder.typepad.com/blog/2010/04/manual-regeneration-mode-danger.html)
- [Regeneration option best practices](http://thebuildingcoder.typepad.com/blog/2010/04/regeneration-option-best-practices.html)
- [To regenerate or not to regenerate...](http://thebuildingcoder.typepad.com/blog/2010/06/to-regenerate-or-not-to-regenerate.html)
- [Refresh referencing sheet parameter display](http://thebuildingcoder.typepad.com/blog/2010/11/refresh-referencing-sheet-parameter-display.html)
- [Setting text width requires regen](http://thebuildingcoder.typepad.com/blog/2010/11/setting-text-width-requires-regen.html)
- [Extra transaction or regeneration required](http://thebuildingcoder.typepad.com/blog/2012/12/extra-transaction-or-regeneration-required.html)
- [Regenerate between plane and dimension creation](http://adndevblog.typepad.com/aec/2013/01/it-is-easy-to-miss-this-regenerating-the-model.html)
- [Regenerate to avoid accessing stale data](http://thebuildingcoder.typepad.com/blog/2013/11/erasing-extensible-storage-with-linked-files.html#3)

The topic of regeneration is also related to the
[temporary transaction trick](http://thebuildingcoder.typepad.com/blog/2012/10/the-temporary-transaction-trick-for-gross-slab-data.html),
the associated suggestion to
[encapsulate multiple transactions in a transaction group](http://thebuildingcoder.typepad.com/blog/2012/11/temporary-transaction-trick-touchup.html),
commit the individual transactions and then roll bock the entire group instead.

In Revit 2010, in this particular case, the regeneration was apparently not required.

Whenever you run into an issue like this, you must always consider the
[need to regenerate](http://thebuildingcoder.typepad.com/blog/about-the-author.html#5.33).

#### InstanceVoidCutTest Add-in

I implemented a Revit add-in defining an external command to demonstrate this in the
[InstanceVoidCutTest GitHub repository](https://github.com/jeremytammik/InstanceVoidCutTest).

The initial code without the cut working is captured in
[release 2014.0.0.0](https://github.com/jeremytammik/InstanceVoidCutTest/releases/tag/2014.0.0.0).

In
[release 2014.0.0.2](https://github.com/jeremytammik/InstanceVoidCutTest/releases/tag/2014.0.0.2),
I separated the operations placing the cutting instances and applying the cut into two separate transactions.
Both operations were performed in a single loop iterating over the three instances, so I separated them into two loops, each contained in its own transaction.

At that point, all works well.

#### Transactions, Transaction Groups and Simple Regeneration

Three separate transactions are used to load the cutting family, create the cutting instances and apply the actual cuts:

![Three separate transactions](img/instance_void_cut_three_transactions.png)

If you prefer not to display the individual transaction generated by one single command in the undo stack visible in the user interface, you can collect them all together into a transaction group.

I therefor added a transaction group and assimilated the transactions into that by calling its Assimilate method.

Note that the call to Assimilate includes the commit, so trying to call Commit afterwards in addition to Assimilate will throw an exception saying "The Transaction group has not been started (its status is not 'Started')".

Here is the undo stack displaying the single transaction group:

![Transaction group](img/instance_void_cut_transaction_group.png)

This version is captured in the GitHub repository as
[release 2014.0.0.3](https://github.com/jeremytammik/InstanceVoidCutTest/releases/tag/2014.0.0.3).

Looking more closely at my colleagues' recommendations above, I see that they are actually just pointing out the need to regenerate, which is a less expensive operation than committing a transaction and starting a new one.

I therefore updated the implementation in
[release 2014.0.0.4](https://github.com/jeremytammik/InstanceVoidCutTest/releases/tag/2014.0.0.4) to
use one single transaction to both create instances and apply the cut, and added a call to regenerate in between.

#### InstanceVoidCutTest External Command Implementation

Here is the final implementation:

```csharp
#region Namespaces
using System;
using System.Collections.Generic;
using System.Diagnostics;
using System.Linq;
using Autodesk.Revit.ApplicationServices;
using Autodesk.Revit.Attributes;
using Autodesk.Revit.DB;
using Autodesk.Revit.UI;
using Autodesk.Revit.UI.Selection;
using Autodesk.Revit.DB.Structure;
using System.IO;
#endregion

namespace InstanceVoidCutTest
{
  [Transaction( TransactionMode.Manual )]
  public class Command : IExternalCommand
  {
    const string FamilyName = "Cutter";

    const string FamilyPath
      = "C:/a/vs/InstanceVoidCutTest/Cutter.rfa";

    public static void ErrorMsg( string msg )
    {
      Debug.WriteLine( msg );

      TaskDialog.Show( "InstanceVoidCutUtils Test",
        msg );
    }

    /// <summary>
    /// Return a string for a real number
    /// formatted to two decimal places.
    /// </summary>
    public static string RealString( double a )
    {
      return a.ToString( "0.##" );
    }

    /// <summary>
    /// Return a string for an XYZ point
    /// or vector with its coordinates
    /// formatted to two decimal places.
    /// </summary>
    public static string PointString( XYZ p )
    {
      return string.Format( "({0},{1},{2})",
        RealString( p.X ),
        RealString( p.Y ),
        RealString( p.Z ) );
    }

    public class BeamSelectionFilter : ISelectionFilter
    {
      public bool AllowElement( Element e )
      {
        return e is FamilyInstance
          && null != e.Category
          && e.Category.Id.IntegerValue.Equals( (int)
            BuiltInCategory.OST\_StructuralFraming );
      }

      public bool AllowReference( Reference r, XYZ p )
      {
        return true;
      }
    }

    /// <summary>
    /// Retrieve cutting symbol,
    /// loading family if needed.
    /// </summary>
    static FamilySymbol RetrieveOrLoadCuttingSymbol(
      Document doc )
    {
      FilteredElementCollector a
          = new FilteredElementCollector( doc )
            .OfClass( typeof( Family ) );

      Family family = a.FirstOrDefault<Element>(
        e => e.Name.Equals( FamilyName ) )
          as Family;

      if( null == family )
      {
        // It is not present, so check for
        // the file to load it from:

        if( !File.Exists( FamilyPath ) )
        {
          ErrorMsg( string.Format(
            "Please ensure that the void cutter "
            + "family file '{0}' is present.",
            FamilyPath ) );

          return null;
        }

        // Load family from file:

        using( Transaction tx = new Transaction(
          doc ) )
        {
          tx.Start( "Load Family" );
          doc.LoadFamily( FamilyPath, out family );
          tx.Commit();
        }
      }

      FamilySymbol cuttingSymbol = null;

      foreach( FamilySymbol s in family.Symbols )
      {
        cuttingSymbol = s;
        break;
      }

      return cuttingSymbol;
    }

    /// <summary>
    /// Cut a beam with three instances of a void
    /// cutting family. Its family parameter "Cut
    /// with Voids When Loaded" must be set to true.
    /// </summary>
    static void CutBeamWithVoid(
      FamilyInstance beam,
      FamilySymbol cuttingSymbol )
    {
      Document doc = beam.Document;

      Level level = doc.GetElement( beam.LevelId )
        as Level;

      LocationCurve lc = beam.Location
        as LocationCurve;

      Curve beamCurve = lc.Curve;

      Debug.Print( "Beam location from {0} to {1}.",
        PointString( beamCurve.GetEndPoint( 0 ) ),
        PointString( beamCurve.GetEndPoint( 1 ) ) );

      XYZ p;
      int n = 3;
      string parameter\_name;
      ElementId[] ids = new ElementId[n];

      using( Transaction tx = new Transaction( doc ) )
      {
        tx.Start( "Create Cutting Instances and Apply Cut" );

        for( int i = 1; i <= n; ++i )
        {
          // Position on beam for this cutting instance

          p = beamCurve.Evaluate( i \* 0.25, true );

          Debug.Print(
            "Family instance insertion at {0}.",
            PointString( p ) );

          FamilyInstance cuttingInstance = doc.Create
            .NewFamilyInstance( p, cuttingSymbol,
              level, StructuralType.NonStructural );

          parameter\_name = "A" + i.ToString();

          cuttingInstance
            .get\_Parameter( parameter\_name )
            .Set( 0.5 \* Math.PI );

          ids[i - 1] = cuttingInstance.Id;
        }

        doc.Regenerate();

        for( int i = 0; i < n; ++i )
        {
          InstanceVoidCutUtils.AddInstanceVoidCut(
            doc, beam, doc.GetElement( ids[i] ) );
        }
        tx.Commit();
      }
    }

    public Result Execute(
      ExternalCommandData commandData,
      ref string message,
      ElementSet elements )
    {
      UIApplication uiapp = commandData.Application;
      UIDocument uidoc = uiapp.ActiveUIDocument;
      Document doc = uidoc.Document;

      using( TransactionGroup g
        = new TransactionGroup( doc ) )
      {
        g.Start( "Cut Beam with Voids" );

        // Retrieve or load cutting symbol

        FamilySymbol cuttingSymbol
          = RetrieveOrLoadCuttingSymbol( doc );

        // Select beam to cut

        Selection sel = uidoc.Selection;

        FamilyInstance beam = null;

        try
        {
          Reference r = sel.PickObject(
            ObjectType.Element,
            new BeamSelectionFilter(),
            "Pick beam to cut" );

          beam = doc.GetElement( r.ElementId )
            as FamilyInstance;
        }
        catch( Autodesk.Revit.Exceptions
          .OperationCanceledException )
        {
          return Result.Cancelled;
        }

        // Place cutting instances and apply cuts

        CutBeamWithVoid( beam, cuttingSymbol );

        g.Assimilate();
      }
      return Result.Succeeded;
    }
  }
}
```

In future (Jeremy), please do remember the

need to regenerate

**Addendum:** Arnošt Löbel points out that the title of this post may be a bit misleading.

There is no regeneration for 'Transaction Groups'.
Regenerations of a model are possible only within a Transaction.

Jeremy replies that the original title was 'Transaction Groups and Regeneration Need for InstanceVoidCutUtils', but that is too long and causes a horrible line break, so I just removed all the words that could possibly be left out.

Sorry for that :-)