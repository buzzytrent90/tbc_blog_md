---
post_number: "1214"
title: "Modifying, Saving and Reloading Families"
slug: "mod_reload_family"
author: "Jeremy Tammik"
tags: ['csharp', 'elements', 'family', 'filtering', 'parameters', 'python', 'revit-api', 'transactions', 'views']
source_file: "1214_mod_reload_family.htm"
original_url: "https://thebuildingcoder.typepad.com/blog/1214_mod_reload_family.html"
---

### Modifying, Saving and Reloading Families

I recently grabbed one of those rare opportunities to do a little bit of coding myself, to answer a question on modifying and reloading a family.

More precisely, the task at hand is to modify the text note type font in all of the loaded families.

I'll take a look at that [below.](#3)

First, lets look at a simpler question, on how to
[save a family after editing it](http://forums.autodesk.com/t5/revit-api/saving-family-after-editing-with-familymanager/td-p/5291581),
raised by Raunoveb on the
[Revit API discussion forum](http://forums.autodesk.com/t5/revit-api/bd-p/160):

#### Saving Family after Editing with FamilyManager

**Question:** We are developing a solution that uploads Revit Family files to DMS server.

First we added a few shared parameters that have the information of DMS record to our families using the FamilyManager AddParameter method.

Everything works fine until we call our UploadOriginal method that attempts to save the modified family file:

```csharp
  string path = Path.GetTempPath();
  string name = family.Name;
  string fName = name + ".rfa";
  string fPath = path + fName;

  // Revit throws an error on this line
  // saying that Family is not editable
  // What could cause this mayhem?
  // To upload .rfa Family file I need to
  // save it as a file first and that's what
  // I try to do until mighty ERROR occurs

  Document famDoc = doc.EditFamily( family );
  famDoc.SaveAs( fPath );
  famDoc.Close( false );

  // application related code following...
```

What could be the reason of this error? How could I fix it?
Must it be overwritten like a baus ([BOSS](https://www.youtube.com/watch?v=NisCkxU544c))?

**Answer:** I recently implemented an add-in that I discuss further down to update fonts in loaded families.

For some of the loaded families, EditFamily threw that same exception saying, "Family is not editable".

I added an exception handler to skip them:

```csharp
  try
  {
    r1.FamilyDocument
      = famdoc
      = doc.EditFamily( f );
  }
  catch( Autodesk.Revit.Exceptions.ArgumentException ex )
  {
    r1.Skipped = true;
    results.Add( r1 );
    Debug.Print( "Family '{0}': {1}", f.Name, ex.Message );
    continue;
  }
```

Here is an excerpt of the add-in log; the families marked 'skipped' are the ones I mean:

![Reload family report](img/reload_family_result_msg.png)

I simply assumed that is normal.

P.S. cool video :-)

**Question:** I managed to skip those families by simply adding an `if(family.IsEditable)` block around the code.
Unfortunately for us these families we try to process must be processed so we can't just skip them.

I've had 2 ideas that could explain those errors:

FamilyManager (fm) somehow locks the family currently open in Family Document and only allows us to edit the family after it's released the family.
However even after fm.Dispose() the EditFamily method threw that exception.

(This idea could be related to first idea).
One cannot just simply write a family to file just after the shared parameters have been changed/added.
A "save" command must be somehow executed in order to "unlock" this family.
However once again I found no methods in either FamilyManager or Family class that would allows us to do that.

I have danced around this problem for 3 days and have ran out of ideas.
Any help "unlocking" these families would be greatly appreciated.

**Answer:** Thank you very much for pointing out the IsEditable predicate.
I successfully replaced my exception handler by simply checking that instead.

Regarding your locked families, have you tried ensuring that absolutely no other documents are open, only the one and only family that you are trying to modify?

**Question:** Thanks for your comment about any other open documents.
I suddenly realized that in my main program I already had FamilyDocument open.
When I used UploadOriginal() method I gave doc as an input and then tried:

```csharp
  var famDoc = doc.EditFamily( family );
  famDoc.SaveAs( path );
  famDoc.Close( false );
```

That created another Family Document instance of the same family and that's why it threw this "Family not editable" exception. I changed it to:

```csharp
  doc.SaveAs( path );

  // Can't close it since I have this family view
  // open in Revit and API doesn't have permission
  // to close visibly open documents in Revit.

  //doc.Close(false);
```

However now I've got problems with getting the Family.Name value from doc.OwnerFamily.Name.
It always returns "" and all the files saved look like this "/folder/.rfa", when they should look something like this "/folder/NightLamp.rfa".

**Answer:** Cool.

Progress.

How about using the document title instead of the family name?

**Question:** Using doc.Title did the trick. However it's weird that OwnerFamily.Name returned blank response.

Thanks for everything. And since your second answer provided most help regarding to the main question I'll mark that one as a solution.

**Answer:** Cooler still.

That was a quick solution.

Thank you for marking the solution and for the interesting discussion.

#### Replacing all Text Note Type Fonts in all Loaded Families

With that little intermezzo out of the way, we get to the real thing:

**Question:** Update text font style property.

I am trying to update the text font property for the active project and also update all the families loaded in that project, i.e. the active document.

The first part works fine, updating the text property for the active project.

I am having trouble figuring out how to update the text property for all the families loaded into the active project, though.

This is my current coding attempt:

```csharp
  Document doc = commandData.Application
    .ActiveUIDocument.Document;

  FilteredElementCollector collectorUsed
    = new FilteredElementCollector( doc );

  collectorUsed.OfClass( typeof( Family ) );

  foreach( Family f in collectorUsed )
  {
    string name = f.Name;
    Document famdoc = doc.EditFamily( f );

    FilteredElementCollector famcollectorUsed
      = new FilteredElementCollector( famdoc );

    ICollection<ElementId> textNoteTypes
      = famcollectorUsed.OfClass( typeof( TextNoteType ) )
        .ToElementIds();

    foreach( ElementId textNoteTypeId in textNoteTypes )
    {
      Element ele = doc.GetElement( textNoteTypeId );
      foreach( Parameter p in ele.Parameters )
      {
        if( p.Definition.Name == "Text Font" )
        {
          using( Transaction tranew
            = new Transaction( doc ) )
          {
            tranew.Start( "Update" );
            p.Set( "Arial Black" );
            tranew.Commit();
          }
        }
      }
    }
  }
```

**Answer:**

I discussed the topic of reloading families on The Building Coder in 2011:

- [Reloading a family](http://thebuildingcoder.typepad.com/blog/2011/06/reloading-a-family.html)
- [Reloading a family again](http://thebuildingcoder.typepad.com/blog/2011/10/reloading-a-family-again.html)

Some things have changed a little bit since then.

The main principles remain the same, however.

I looked at the help file on the
[Text Note Type Properties](http://knowledge.autodesk.com/support/revit-products/downloads/caas/CloudHelp/cloudhelp/2014/ENU/Revit/files/GUID-B3E40C07-175D-48CA-BF73-3003AF760F7B-htm.html) and
see the Text Font property that you wish to change, and that looks fine.

There is no need to loop through all the element parameters and match individual strings to find the parameter you are after, however.

You can use the GetParameters or LookupParameter method instead. You should obviously no longer use the obsolete get\_Parameter method, if you can avoid it.

By the way, you should also not use the parameter name to identify it if it is possible to use a built-in parameter enumeration value instead, since the latter is both language independent, more efficient, and guaranteed to return a unique result.

For the sake of efficiency, you might want to check whether the current font property setting already as the desired value before you open an extra new transaction and modify it.

Just as you say, though, even after you have modified the text note types in the family document and committed the transactions, the changes are still not reflected in the container project document.

This is actually clearly documented in the Revit API help file RevitAPI.chm, in the remarks on the Document.EditFamily method:

> #### Remarks
>
> This creates an independent copy of the family for editing. To apply the changes back to the family stored in the document, use the LoadFamily overload accepting IFamilyLoadOptions.
>
> This method may not be called if the document is currently modifiable (has an open transaction) or is in a read-only state. The method may not be called during dynamic updates. To test the document's current status, check the values of IsModifiable and IsReadOnly properties.

I also pointed this out when discussing some
[changes in calling the EditDocument method](http://thebuildingcoder.typepad.com/blog/2012/05/edit-family-requires-no-transaction.html) back
in the Revit 2013 timeframe.

There are some further important considerations when
[reloading a family](http://thebuildingcoder.typepad.com/blog/2011/06/reloading-a-family.html).

Actually, here is a complete list of discussions related to reloading families or mentioning the IFamilyLoadOptions interface that might also be useful:

- [The Revit Family API](http://thebuildingcoder.typepad.com/blog/2009/08/the-revit-family-api.html)
- [IFamilyLoadOptions](http://thebuildingcoder.typepad.com/blog/2010/02/ifamilyloadoptions-and-gemini.html)
- [Reloading a Family](http://thebuildingcoder.typepad.com/blog/2011/06/reloading-a-family.html)
- [Reloading a Family Again](http://thebuildingcoder.typepad.com/blog/2011/10/reloading-a-family-again.html)
- [Edit Family Requires No Transaction](http://thebuildingcoder.typepad.com/blog/2012/05/edit-family-requires-no-transaction.html)

Based on the information provided there, we have to explicitly reload all the modified families after updating their text note type font properties by calling the LoadFamily method on each, and specify an appropriate IFamilyLoadOptions interface implementation when doing so. Here is a suitable one, updated from the discussions listed above:

```python
  class JtFamilyLoadOptions : IFamilyLoadOptions
  {
    public bool OnFamilyFound(
      bool familyInUse,
      out bool overwriteParameterValues )
    {
      overwriteParameterValues = true;
      return true;
    }

    public bool OnSharedFamilyFound(
      Family sharedFamily,
      bool familyInUse,
      out FamilySource source,
      out bool overwriteParameterValues )
    {
      source = FamilySource.Family;
      overwriteParameterValues = true;
      return true;
    }
  }
```

Another issue to be aware of is that you are not allowed to perform any element deletions while iterating over the results of a filtered element collector, or Revit will throw an InvalidOperationException saying 'The iterator cannot proceed due to changes made to the Element table in Revit's database (typically, This can be the result of an Element deletion).'

That forced me to postpone the family reloading operation and move it out of the collector iteration loop, where it was originally situated.

I implemented a new add-in named SetTextFontInFamilies to demonstrate all this.

As usual, in order to understand and understand what it really does, the logging and reporting code exceeds the actual task implementation.

It keeps track of all the families processed.

Some families are skipped, because the EditFamily method throws an ArgumentException saying 'This family is not editable.'

If not skipped, it also records all the text note types processed, and how many of them actually require a modification of the font.

Here is the main result logging implementation class:

```csharp
  /// <summary>
  /// Logging helper class to keep track of the result
  /// of updating the font of all text note types in a
  /// family. The family may be skipped or not. If not,
  /// keep track of all its text note types and a flag
  /// for each indicating whether it was updated.
  /// </summary>
  class SetTextFontInFamilyResult
  {
    class TextNoteTypeResult
    {
      public string Name { get; set; }
      public bool Updated { get; set; }
    }

    /// <summary>
    /// The Family element name in the project database.
    /// </summary>
    public string FamilyName { get; set; }

    /// <summary>
    /// The family document used to reload the family.
    /// </summary>
    public Document FamilyDocument { get; set; }

    /// <summary>
    /// Was this family skipped, e.g. this family is not editable.
    /// </summary>
    public bool Skipped { get; set; }

    /// <summary>
    /// List of text note type names and updated flags.
    /// </summary>
    List<TextNoteTypeResult> TextNoteTypeResults;

    public SetTextFontInFamilyResult( Family f )
    {
      FamilyName = f.Name;
      TextNoteTypeResults = null;
    }

    public void AddTextNoteType(
      TextNoteType tnt,
      bool updated )
    {
      if( null == TextNoteTypeResults )
      {
        TextNoteTypeResults
          = new List<TextNoteTypeResult>();
      }
      TextNoteTypeResult r = new TextNoteTypeResult();
      r.Name = tnt.Name;
      r.Updated = updated;
      TextNoteTypeResults.Add( r );
    }

    int NumberOfUpdatedTextNoteTypes
    {
      get
      {
        return null == TextNoteTypeResults
          ? 0
          : TextNoteTypeResults
            .Count<TextNoteTypeResult>(
              r => r.Updated );
      }
    }

    public bool NeedsReload
    {
      get
      {
        return 0 < NumberOfUpdatedTextNoteTypes;
      }
    }

    public override string ToString()
    {
      // FamilyDocument.Title

      string s = FamilyName + ": ";

      if( Skipped )
      {
        s += "skipped";
      }
      else
      {
        int nTotal = TextNoteTypeResults.Count;
        int nUpdated = NumberOfUpdatedTextNoteTypes;

        s += string.Format(
          "{0} text note types processed, "
          + "{1} updated", nTotal, nUpdated );
      }
      return s;
    }
  }
```

I actually ended up using it for more than just logging the results, once I discovered that we need to terminate the first iteration over the filtered element collector before we can apply the modifications.
Therefore, by tracking the font modifications made, this class also keeps track of which families need reloading at all.

The main Execute method making use of this, determining and iterating over all the loaded families, modifying all their text note type fonts and reloading them afterwards, ends up looking like this:

```python
  public Result Execute(
    ExternalCommandData commandData,
    ref string message,
    ElementSet elements )
  {
    UIApplication uiapp = commandData.Application;
    UIDocument uidoc = uiapp.ActiveUIDocument;
    Application app = uiapp.Application;
    Document doc = uidoc.Document;

    FilteredElementCollector families
      = new FilteredElementCollector( doc )
        .OfClass( typeof( Family ) );

    List<SetTextFontInFamilyResult> results
      = new List<SetTextFontInFamilyResult>();

    Document famdoc;
    SetTextFontInFamilyResult r1;
    bool updatedTextNoteStyle;

    foreach( Family f in families )
    {
      r1 = new SetTextFontInFamilyResult( f );

      bool updatedFamily = false;

      // Using exception handler.

      //try
      //{
      //  r1.FamilyDocument
      //    = famdoc
      //    = doc.EditFamily( f );
      //}
      //catch( Autodesk.Revit.Exceptions.ArgumentException ex )
      //{
      //  r1.Skipped = true;
      //  results.Add( r1 );
      //  Debug.Print( "Family '{0}': {1}", f.Name, ex.Message );
      //  continue;
      //}

      // Better: test IsEditable predicate.

      if( f.IsEditable )
      {
        r1.FamilyDocument
          = famdoc
          = doc.EditFamily( f );
      }
      else
      {
        r1.Skipped = true;
        results.Add( r1 );
        Debug.Print( "Family '{0}' is not editable", f.Name );
        continue;
      }

      FilteredElementCollector textNoteTypes
        = new FilteredElementCollector( famdoc )
          .OfClass( typeof( TextNoteType ) );

      foreach( TextNoteType tnt in textNoteTypes )
      {
        updatedTextNoteStyle = false;

        // It is normally better to use the built-in
        // parameter enumeration value rather than
        // the parameter definition display name.
        // The latter is language dependent, possibly
        // returns multiple hits, and uses a less
        // efficient string comparison.

        //Parameter p2 = tnt.get\_Parameter(
        //  \_parameter\_bip );

        IList<Parameter> ps = tnt.GetParameters(
          \_parameter\_name );

        Debug.Assert( 1 == ps.Count,
          "expected only one 'Text Font' parameter" );

        foreach( Parameter p in ps )
        {
          if( \_font\_name != p.AsString() )
          {
            using( Transaction tx
              = new Transaction( doc ) )
            {
              tx.Start( "Update Text Font" );
              p.Set( \_font\_name );
              tx.Commit();

              updatedFamily
                = updatedTextNoteStyle
                = true;
            }
          }
        }
        r1.AddTextNoteType( tnt, updatedTextNoteStyle );
      }
      results.Add( r1 );

      // This causes the iteration over the filtered
      // element collector to throw an
      // InvalidOperationException: The iterator cannot
      // proceed due to changes made to the Element table
      // in Revit's database (typically, This can be the
      // result of an Element deletion).

      //if( updatedFamily )
      //{
      //  f2 = famdoc.LoadFamily(
      //    doc, new JtFamilyLoadOptions() );
      //}
    }

    // Reload modified families after terminating
    // the filtered element collector iteration.

    IFamilyLoadOptions opt
      = new JtFamilyLoadOptions();

    Family f2;

    foreach( SetTextFontInFamilyResult r in results )
    {
      if( r.NeedsReload )
      {
        f2 = r.FamilyDocument.LoadFamily( doc, opt );
      }
    }

    TaskDialog d = new TaskDialog(
      "Set Text Note Font" );

    d.MainInstruction = string.Format(
      "{0} families processed.", results.Count );

    List<string> family\_results
      = results.ConvertAll<string>(
        r => r.ToString() );

    family\_results.Sort();

    d.MainContent = string.Join( "\r\n",
      family\_results );

    d.Show();

    return Result.Succeeded;
  }
```

Here is a report of the result, displayed by this command in a Revit task dialogue:

![Reload family report](img/reload_family_result_msg.png)

This report omits the list of text note types processed within each family, although we actually do keep track of them internally as well in the TextNoteTypeResult and SetTextFontInFamilyResult classes.

Note that I go to the very slight extra effort of sorting the results alphabetically for the convenience of the human reader.

Also note that the task dialogue very kindly adds a scroll bar automatically.

Finally, note that running this command even in a minimal new empty Revit project takes rather a long time to complete.

The complete implementation and Visual Studio solution including the add-in manifest is provided in the
[SetTextFontInFamilies GitHub repository](https://github.com/jeremytammik/SetTextFontInFamilies).