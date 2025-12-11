---
ai_optimized: true
author: Jeremy Tammik
complexity_score: 7.0
content_type: code_example
optimization_date: '2025-12-11T11:44:14.473389'
original_url: https://thebuildingcoder.typepad.com/blog/0728_remove_dwf_link.html
post_number: 0728
reading_time_minutes: 6
series: general
slug: remove_dwf_link
source_file: 0728_remove_dwf_link.htm
tags:
- csharp
- doors
- elements
- family
- python
- references
- revit-api
- sheets
- transactions
- views
- walls
- windows
title: Remove DWF Links
word_count: 1256
---

### Remove DWF Links

I spent this weekend in Chamonix with my brother Nick.
We did one day of rock climbing in the valley at Les Gaillants and Servoz doing
[Surbac a' Bras](http://www.highmountainguides.com/index.php/eng/Gallery/Chamonix-Rock-Climbing/Chamonix-Rock-Climbing-Gallery/Surbac-a-Bras-6a-Servoz), 6a+, and one day of skiing with Chris down the
[Vallee Blanche](http://fr.wikipedia.org/wiki/Vall%C3%A9e_Blanche),
[Mer de Glace](http://fr.wikipedia.org/wiki/Mer_de_Glace) and
[Montenvers](http://fr.wikipedia.org/wiki/Montenvers) from the
[Telepherique](http://fr.wikipedia.org/wiki/T%C3%A9l%C3%A9ph%C3%A9rique_de_l%27aiguille_du_Midi) de
[l'Aiguille du Midi](http://fr.wikipedia.org/wiki/Aiguille_du_Midi).

Our route from the telepherique actually joins the Vallee Blanche via the steep looking descent at the extreme right of this picture, with a sharp ridge crossing it diagonally, the upper half in sunshine, and the lower in shadow:

![Nick and Jeremy in the Vallee Blanche at Chamonix](file:////j/photo/jeremy/2012/2012-03-04_vallee_blanche/WP_000578_nick_jeremy.jpg)

I had a fall and a hundred meter slide coming down that steep incline, which was interesting and a bit painful.

Returning to the Revit API, I recently explored how to
[remove imported JPG and BMP images](http://thebuildingcoder.typepad.com/blog/2012/02/remove-imported-jpg-and-bmp-images.html).
Soon afterwards, another query on removing file references in the model showed up.
In my naivety, I assumed that the solution would be similar.
Far failed, as we shall shortly see:

**Question:** I want to remove all DWF links from a RVT file.
I can remove RVT and DWG links but cannot figure out how to do the same for DWF and did not find any way on the web, help file or API examples.

**Answer:** I looked at your sample model which includes a couple of DWF mark-up objects:

![DWF mark-up elements](img/remove_dwf_link.png)

I used the
[element lister](http://thebuildingcoder.typepad.com/blog/2010/05/pre-post-and-pick-select.html) on
it to create an overview of all its elements in a text file.
The result shows an obvious way to identify at least some of the DWF import instances by using their category name, which includes the substring ".dwfx" (copy to an editor to see the truncated lines in full):

```
Id=161300; Class=ImportInstance; Category=Project1.dwfx[Sheet: A101 - Unnamed]; Name=Markup Object 3 1;
Id=161301; Class=ImportInstance; Category=Project1.dwfx[Sheet: A101 - Unnamed]; Name=Markup Object 2 1;
Id=161302; Class=ImportInstance; Category=Project1.dwfx[Sheet: A101 - Unnamed]; Name=Markup Object 1 1;
```

Searching for other objects with a dwf substring in their category name, I discovered the following element types:

```
Id=161219; Class=ElementType; Category=Project1.dwfx[Sheet: A101 - Unnamed]; Name=Project1.dwfx[Sheet: A101 - Unnamed];
Id=161278; Class=ElementType; Category=Project1.dwfx[Sheet: A101 - Unnamed]; Name=Markup Object 3 1;
Id=161279; Class=ElementType; Category=Project1.dwfx[Sheet: A101 - Unnamed]; Name=Markup Object 2 1;
Id=161280; Class=ElementType; Category=Project1.dwfx[Sheet: A101 - Unnamed]; Name=Markup Object 1 1;
```

Apparently, there is one main element type for the file reference itself, then one element type and one import instance for each of its occurrences in the model.

I tried to use this information and an approach similar to the one described for the
[removal of image files](http://thebuildingcoder.typepad.com/blog/2012/02/remove-imported-jpg-and-bmp-images.html) to
delete them from the model.

Analogously to the ElementNameEndsWithJpg predicate checking the element name, I implemented ElementCategoryContainsDwf which retrieves and checks the category name for the DWF substring.
It returns true if the given element category name contains the substring ".dwf":
```csharp
bool ElementCategoryContainsDwf( Element e )
{
  return ( null != e.Category )
    && e.Category.Name.ToLower()
      .Contains( ".dwf" );
}
```

I then implemented the algorithm which first retrieves and deletes the non-ElementType objects fulfilling this criterion, followed by the element type ones, but the call to doc.Delete with their ids simply returns null and does nothing.

I noted that the DWF mark-up elements are pinned, so I added code to unpin them prior to the deletion attempt:
```csharp
int Unpin( List<ElementId> ids, Document doc )
{
  int count = 0;

  foreach( ElementId id in ids )
  {
    Element e = doc.get\_Element( id );
    if( e.Pinned )
    {
      e.Pinned = false;
      ++count;
    }
  }
  return count;
}
```

Unfortunately, that still did not help.

The new command CmdRemoveDwfLinks in The Building Coder samples that I implemented to demonstrate the working solution provided below includes the code for this failed attempt, in case you are interested, but I will omit it here for the sake of brevity.
It may be of use if you ever run into the need to unpin a set of elements, for example.

Just as I was about to give up and ask for help, I discovered a previous case that provides a hint:

"Trying to delete the markup element will not work using the Document.Delete method, because you cannot delete that kind of element directly through the user interface either, like you would a wall or door.
Instead, you have to launch the Manage > Manage Links dialogue and remove the markup there.
In the API, this can be achieved using the ExternalFileUtils class."

I implemented a new method RemoveDwfLinkUsingExternalFileUtils and a call to execute it to the external command CmdRemoveDwfLinks to test this.
It removes all DWF links from the model and returns the total number of deleted elements:
```python
int RemoveDwfLinkUsingExternalFileUtils(
  Document doc )
{
  List<ElementId> idsToDelete
    = new List<ElementId>();

  ICollection<ElementId> ids = ExternalFileUtils
    .GetAllExternalFileReferences( doc );

  foreach( ElementId id in ids )
  {
    Element e = doc.get\_Element( id );

    Debug.Print( Util.ElementDescription( e ) );

    ExternalFileReference xr = ExternalFileUtils
      .GetExternalFileReference( doc, id );

    ExternalFileReferenceType xrType
      = xr.ExternalFileReferenceType;

    if( xrType == ExternalFileReferenceType.DWFMarkup )
    {
      ModelPath xrPath = xr.GetPath();

      string path = ModelPathUtils
        .ConvertModelPathToUserVisiblePath( xrPath );

      if( path.EndsWith( ".dwf" )
        || path.EndsWith( ".dwfx" ) )
      {
        idsToDelete.Add( id );
      }
    }
  }

  int n = idsToDelete.Count;

  ICollection<ElementId> idsDeleted = null;

  if( 0 < n )
  {
    using( Transaction t = new Transaction( doc ) )
    {
      t.Start( "Delete DWFx Links" );

      idsDeleted = doc.Delete( idsToDelete );

      t.Commit();
    }
  }

  int m = ( null == idsDeleted )
    ? 0
    : idsDeleted.Count;

  Debug.Print( string.Format(
    "Selected {0} DWF external file reference{1}, "
    + "{2} element{3} successfully deleted.",
    n, Util.PluralSuffix( n ), m, Util.PluralSuffix( m ) ) );

  return m;
}
```

Here is the complete code of the external command mainline Execute method calling both the initial non-functional RemoveDwfLinkUsingDelete and the new working RemoveDwfLinkUsingExternalFileUtils methods:
```csharp
public Result Execute(
  ExternalCommandData commandData,
  ref string message,
  ElementSet elements )
{
  UIApplication uiapp = commandData.Application;
  UIDocument uidoc = uiapp.ActiveUIDocument;
  Document doc = uidoc.Document;

  if( doc.IsFamilyDocument )
  {
    Util.ErrorMsg(
      "This command requires an active document." );

    return Result.Failed;
  }

  int nDeleted = RemoveDwfLinkUsingDelete( doc );

  int nDeleted2 = RemoveDwfLinkUsingExternalFileUtils( doc );

  return Result.Succeeded;
}
```

This seems to do the trick for me all right.

The new code in RemoveDwfLinkUsingExternalFileUtils is significantly shorter and simpler than the failed initial attempt using the Delete method directly, besides having the advantage of functioning :-)

It also turns out that far more elements than just the seven listed above that I found by searching the element and category names for the DWF substring are deleted by the removal of the DWF file reference.
In fact, a total of 25 elements are affected.

Running the CmdRemoveDwfLinks external command on it produces the following messages in the Visual Studio debug output window:

```
Element <117704 Standard>
Project1.dwfx[Sheet: A101 - Unnamed] <161219 Project1.dwfx[Sheet: A101 - Unnamed]>
Selected 1 DWF external file reference, 25 elements successfully deleted.
```

Here is
[version 2012.0.98.0](zip/bc_12_98.zip) of
The Building Coder samples including the new command.