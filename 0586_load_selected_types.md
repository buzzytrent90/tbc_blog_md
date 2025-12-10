---
post_number: "0586"
title: "Loading Only Selected Family Types"
slug: "load_selected_types"
author: "Jeremy Tammik"
tags: ['csharp', 'elements', 'family', 'revit-api', 'transactions', 'windows']
source_file: "0586_load_selected_types.htm"
original_url: "https://thebuildingcoder.typepad.com/blog/0586_load_selected_types.html"
---

### Loading Only Selected Family Types

As you probably know, the Revit API provides two methods to load family types, also known as symbols: LoadFamilySymbol, which loads an individual type, and LoadFamily, which loads the entire family and all the types it defines. Use of the latter method can be quite time consuming and wasteful, especially if you only require a few of the types included.

In the past we also discussed a couple of other issues on handling family types, such as
[creating a new family type](http://thebuildingcoder.typepad.com/blog/2008/11/creating-a-new-family-symbol.html) using the Duplicate method
and
[unloading an unused type](http://thebuildingcoder.typepad.com/blog/2009/06/unload-family-type.html).

Erez van Leeuwen of
[Nordined](http://www.npqsolutions.com/cadnl) voiced
the interesting request of how to avoid the overhead of calling LoadFamily and loading all its types by presenting a list of the types to the user and asking her which ones are actually required, to which Saikat Bhattacharya came up with the following solution:

**Question:** Revit provides multiple ways of loading a family.
The two 'Load Families' options displayed in the Revit ribbon are

- Home > Component > Load Family- Insert > Load Family

The first one loads a family and places an instance of it, and the second one only loads the family.
If the chosen family has a TXT file associated with it containing different types, Revit shows a popup window where you can select the types you want to load.

Using the API to load a family, I can choose between the LoadFamily and LoadFamilySymbol methods.
Is there any other function I can use to load a family and have Revit pop up the dialog to choose which types I want loaded?

If this does not exist, can you tell me if there is a way to read which types exist within a family?

**Answer:** There Revit API does not provide any method which will pop up a dialogue listing the types contained in a family.

You could load the entire family using the LoadFamily method and then use the Family.Symbols method to list all the types defined in it. The Revit API help file RevitAPI.chm does in fact include a code snippet showing how to iterate through all the symbols in a loaded family:
```csharp
public void GetInfoForSymbols( Family family )
{
  StringBuilder message = new StringBuilder(
    "Selected element's family name is : "
    + family.Name );

  if( family.Symbols.IsEmpty )
  {
    message.AppendLine(
      "Contains no family symbols." );
  }
  else
  {
    message.AppendLine(
      "The family symbols contained"
      + " in this family are : " );

    // Get family symbols contained in this family

    foreach( FamilySymbol sym in family.Symbols )
    {
      // Get family symbol name

      message.AppendLine( "\nName: " + sym.Name );

      foreach( Material material in sym.Materials )
      {
        message.AppendLine(
          "\tMaterial: " + material.Name );
      }
    }
  }
  TaskDialog.Show( "Revit", message.ToString() );
}
```

The problem is, of course, that this loads the entire family and all its symbols up front. Doing this for all the families needed will cause the Revit project to grow and become bigger very fast.

Access to and iteration over all the family symbols without permanently loading the entire family into the project file can be achieved using the power of transactions in the Revit API.

You can load a family within a transaction, read through its symbols, and then roll back the transaction to undo the loading of the family.
During this process, the names of the symbols contained in the loaded family can be extracted and saved.
Subsequently, a new transaction can be used to call the LoadFamilySymbol method to load a specific symbol based on the names retrieved in the previous step.
The following code snippet illustrates this approach and is quite simple and self-explanatory:
```csharp
  string filename = @"C:\Documents and Settings"
    + "\All Users\Application Data\Autodesk"
    + "\RAC 2011\Metric Library\Columns"
    + "\M\_Round Column.rfa";

  UIApplication app = commandData.Application;
  Document doc = app.ActiveUIDocument.Document;

  Transaction trans = new Transaction(
    doc, "FakeLoading" );

  trans.Start();

  Family family = null;
  string symbName = String.Empty;
  int counter = 0;
  if( doc.LoadFamily( filename, out family ) )
  {
    foreach( FamilySymbol symb in family.Symbols )
    {
      TaskDialog.Show( "Symbol names", symb.Name );
      if( counter == 0 )
      {
        symbName = symb.Name;
      }
      counter++;
    }
  }
  trans.RollBack();

  Transaction transNew = new Transaction(
    doc, "RealLoading" );

  transNew.Start();

  if( doc.LoadFamilySymbol( filename, symbName ) )
  {
    TaskDialog.Show( "Status",
      "We managed to load only one desired symbol!" );
  }
  transNew.Commit();
```

**Question:** Of course, loading the whole family, rolling everything back and reloading the required types might considerably slow down the complete loading operation.

If the whole family is already being loaded, it might be faster to only unload or delete the unwanted types.

Is there a way to unload specific types? Within Revit I can just delete the unwanted types by pressing delete or I can purge them out.
Can I do this via the API?

**Answer:** As mentioned above, to
[unload specific family types](http://thebuildingcoder.typepad.com/blog/2009/06/unload-family-type.html),
you can simply delete them from the Revit document object.

If you want to unload more than one type, you can collect all the types that you wish to unload in an ElementSet or create an ICollection of their element ids and use one of the following overloads of the Document.Delete method:

- Delete(ElementSet)- Delete(ICollection(Of ElementId))

While editing this, another thought crossed my mind:

The approach above is based on reading the family into the project in order to determine what types it contains.
There may be another possible approach, of course, not loading the family into the project but opening the family file as a separate background document via the API and analysing that using its family manager object instead.
You can iterate the types that way as well.
I don't know whether there is also any need to handle a possibly associated TXT file, and how that would affect the list of types returned by the family manager, but it might be worth a try.