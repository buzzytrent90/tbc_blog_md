---
ai_optimized: true
author: Jeremy Tammik
complexity_score: 4.7
content_type: code_example
optimization_date: '2025-12-11T11:44:13.903258'
original_url: https://thebuildingcoder.typepad.com/blog/0410_change_element_type.html
post_number: '0410'
reading_time_minutes: 4
series: transactions
slug: change_element_type
source_file: 0410_change_element_type.htm
tags:
- csharp
- elements
- family
- filtering
- parameters
- python
- revit-api
- transactions
- walls
- windows
title: Change Element Type
word_count: 736
---

### Change Element Type

Here is a simple question that we have touched upon in passing a couple of times, but never previously elevated to the honour of an own post.
It also provides an opportunity to highlight one of the more interesting basic Revit API introduction labs that I always wanted to discuss in some detail:

**Question:** I can retrieve all the family instances in the document.
I would now like to programmatically change their family symbols to match certain given requirements.
Could you please explain how I can change the family symbol of a given instance?

A similar question was also raised in a
[comment](http://thebuildingcoder.typepad.com/blog/2008/11/creating-a-new-family-symbol.html?cid=6a00e553e1689788330128760d9099970c#comment-6a00e553e1689788330128760d9099970c) by
[Anthony Forlong](http://www.cbp.co.nz):

**Question:** I've been trying to adapt your wall code to enable a user to change a family symbol to another type (of the same family).
I can duplicate the selected symbol and change parameters etc.; however I am unable to find a way to replace the selected symbol with the new one.

**Answer:** The answer is to simply assign the new type to the FamilyInstance Symbol property.

Here are two examples of doing this that we presented here in the past:

- [Duplicating a type and assigning the new type to a family instance](http://thebuildingcoder.typepad.com/blog/2009/04/revit-api-cases-1.html#3):

  ```csharp
  inst.Symbol = dupSym;
  ```- [Changing the symbol of a curtain panel family instance to a glazed type](http://thebuildingcoder.typepad.com/blog/2009/07/revit-form-creation-api.html):

    ```csharp
    fi.Symbol = pFamilySymbol.glazed;
    ```

By the way, the symbol of a family instance is also referred to as the element type, and represented in the Revit API by the ElementType class.

For a more complex and complete example, the Revit API introduction external command Lab3\_4\_ChangeSelectedInstanceType demonstrates how to change the symbol of a selected instance.
It shows a couple of other interesting techniques as well on the way.

Lab3\_4\_ChangeSelectedInstanceType is a form-based utility to change the type or symbol of a selected standard family instance and implements the following steps:

- Retrieve a single preselected family instance or prompt the user to select one interactively.- Determine its category.- Retrieve all families that can be used to represent that category.- For each of the applicable families, retrieve all their symbols and insert them into a mapping from family name to a list of the symbols it contains.- Display a dialogue box prompting the user to select one of applicable symbols from one the families.- Change the family instance type to the selected symbol.

The important line of code that addresses your question is at the end of the Execute method:
```csharp
  inst.Symbol = form.cmbType.SelectedItem
    as FamilySymbol;
```

For completeness sake, here is the entire code of the Execute method implementing the steps listed above:
```python
[Transaction( TransactionMode.Automatic )]
[Regeneration( RegenerationOption.Manual )]
public class Lab3\_4\_ChangeSelectedInstanceType
  : IExternalCommand
{
  public Result Execute(
    ExternalCommandData commandData,
    ref string message,
    ElementSet elements )
  {
    UIApplication app = commandData.Application;
    UIDocument uidoc = app.ActiveUIDocument;
    Document doc = uidoc.Document;

    FamilyInstance inst =
      LabUtils.GetSingleSelectedElementOrPrompt(
        uidoc, typeof( FamilyInstance ) )
        as FamilyInstance;

    if( null == inst )
    {
      LabUtils.ErrorMsg(
        "Selected element is not a "
        + "standard family instance." );

      return Result.Cancelled;
    }

    // determine selected instance category:
    Category instCat = inst.Category;

    Dictionary<string, List<FamilySymbol>>
      mapFamilyToSymbols
        = new Dictionary<string, List<FamilySymbol>>();

    {
      WaitCursor waitCursor = new WaitCursor();

      // find all corresponding families and types:

      FilteredElementCollector families
        = new FilteredElementCollector( doc );

      families.OfClass( typeof( Family ) );

      foreach( Family f in families )
      {
        bool categoryMatches = false;

        // we cannot trust f.Category or
        // f.FamilyCategory, so grab the category
        // from first family symbol instead:

        foreach( FamilySymbol sym in f.Symbols )
        {
          categoryMatches = sym.Category.Id.Equals(
            instCat.Id );

          break;
        }

        if( categoryMatches )
        {
          List<FamilySymbol> symbols
            = new List<FamilySymbol>();

          foreach( FamilySymbol sym in f.Symbols )
          {
            symbols.Add( sym );
          }

          mapFamilyToSymbols.Add( f.Name, symbols );
        }
      }
    }

    // display the form allowing the user to select
    // a family and a type, and assign this type
    // to the instance.

    Lab3\_4\_Form form
      = new Lab3\_4\_Form( mapFamilyToSymbols );

    if( System.Windows.Forms.DialogResult.OK
      == form.ShowDialog() )
    {
      inst.Symbol = form.cmbType.SelectedItem
        as FamilySymbol;

      LabUtils.InfoMsg(
        "Successfully changed family : type to "
        + form.cmbFamily.Text + " : "
        + form.cmbType.Text );
    }
    return Result.Succeeded;
  }
}
```

Here is a snapshot of my current version of the Revit API introduction labs including the Lab3\_4\_ChangeSelectedInstanceType external command,
[rac\_labs\_2010-07-16.zip](zip/rac_labs_2010-07-16.zip).