---
post_number: "0722"
title: "Remove Imported JPG and BMP Images"
slug: "remove_image_import"
author: "Jeremy Tammik"
tags: ['csharp', 'elements', 'filtering', 'python', 'revit-api', 'transactions', 'windows']
source_file: "0722_remove_image_import.htm"
original_url: "https://thebuildingcoder.typepad.com/blog/0722_remove_image_import.html"
---

### Remove Imported JPG and BMP Images

Here is a query which helps understand how imported files are handled by Revit and how they can be removed from the model.

**Question:** I want to remove imported images like JPG and BMP files from my Revit model.

I tried to use a filtered element collector, apply its WhereElementIsNotElementType filter, select all elements whose name ends in ".jpg", and then delete them in a separate loop, but that does not seem to do it.
Some other similar attempts also failed.

How can I achieve this, please?

**Answer:** I created a new external command CmdRemoveImportedJpgs in The Building Coder samples to test this.

The approach you describe really does not work.
Next, I used my element listed and searched for all elements which name included the substring ".jpg", and that led me in the right direction:

I added one important twist to the description you provide above: I first delete all non-ElementType elements whose element name ends with ".jpg" before deleting the element types with that property.
This is the little helper predicate method I use to test the element name:
```csharp
bool ElementNameEndsWithJpg( Element e )
{
  string s = e.Name;

  return 3 < s.Length && s.EndsWith( ".jpg" );
}
```

For full coverage, you obviously need to take into account all the other filename extension permutations pointed out by Rudolf in his comment below.

Here is the complete code of my external command implementation:
```python
  UIApplication uiapp = commandData.Application;
  UIDocument uidoc = uiapp.ActiveUIDocument;
  Document doc = uidoc.Document;

  FilteredElementCollector col
    = new FilteredElementCollector( doc )
      .WhereElementIsNotElementType();

  List<ElementId> ids = new List<ElementId>();

  foreach( Element e in col )
  {
    if( ElementNameEndsWithJpg( e ) )
    {
      Debug.Print( Util.ElementDescription( e ) );
      ids.Add( e.Id );
    }
  }

  ICollection<ElementId> idsDeleted = null;
  Transaction t;

  int n = ids.Count;

  if( 0 < n )
  {
    using( t = new Transaction( doc ) )
    {
      t.Start( "Delete non-ElementType '.jpg' elements" );

      idsDeleted = doc.Delete( ids );

      t.Commit();
    }
  }

  int m = ( null == idsDeleted )
    ? 0
    : idsDeleted.Count;

  Debug.Print( string.Format(
    "Selected {0} non-ElementType element{1}, "
    + "{2} successfully deleted.",
    n, Util.PluralSuffix( n ), m ) );

  col = new FilteredElementCollector( doc )
    .WhereElementIsElementType();

  ids.Clear();

  foreach( Element e in col )
  {
    if( ElementNameEndsWithJpg( e ) )
    {
      Debug.Print( Util.ElementDescription( e ) );
      ids.Add( e.Id );
    }
  }

  n = ids.Count;

  if( 0 < n )
  {
    using( t = new Transaction( doc ) )
    {
      t.Start( "Delete element type '.jpg' elements" );

      idsDeleted = doc.Delete( ids );

      t.Commit();
    }
  }

  m = ( null == idsDeleted ) ? 0 : idsDeleted.Count;

  Debug.Print( string.Format(
    "Selected {0} element type{1}, "
    + "{2} successfully deleted.",
    n, Util.PluralSuffix( n ), m ) );

  return Result.Succeeded;
```

This seems to do the trick for me all right.

To test it, I created a new Revit model and inserted one single image named "jeremy\_philippe\_partha\_jim.jpg".

Running the CmdRemoveImportedJpgs external command on it produces the following messages in the Visual Studio debug output window:
```csharp
Raster Images <161178 jeremy\_philippe\_partha\_jim.jpg>
Selected 1 non-ElementType element, 1 successfully deleted.
ElementType <161176 jeremy\_philippe\_partha\_jim.jpg>
Raster Images <161177 jeremy\_philippe\_partha\_jim.jpg>
Selected 2 element types, 2 successfully deleted.
```

This shows that the image import generated two ElementType objects and one non-ElementType one, and all were successfully deleted.

Apparently, your initial approach does not work because it is not possible to delete the ElementType objects without first removing the non-ElementType one.

Note that they have three sequential element ids.
The raster image element with the id 161178 requires the existence of the other two.

Here is
[version 2012.0.97.0](zip/bc_12_97.zip) of
The Building Coder samples including the new command.