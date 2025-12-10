---
post_number: "1775"
title: "Getsimilartypes"
slug: "getsimilartypes"
author: "Jeremy Tammik"
tags: ['doors', 'elements', 'family', 'filtering', 'parameters', 'revit-api', 'sheets', 'views', 'walls']
source_file: "1775_getsimilartypes.md"
original_url: "https://thebuildingcoder.typepad.com/blog/1775_getsimilartypes.html"
---

### GetSimilarTypes, SnappingService and Title Labels
So many interesting discussions and inspiring solutions in
the [Revit API discussion forum](http://forums.autodesk.com/t5/revit-api-forum/bd-p/160)!
Here are two of them, plus one non-forum beginner case:
- [`GetSimilarTypes` filters for curtain wall door symbols](#2)
- [`SnappingService` – what does it actually do?](#3)
- [Get title block label parameters](#4)
#### GetSimilarTypes Filters for Curtain Wall Door Symbols
Yet once again,
Frank [@Fair59](https://forums.autodesk.com/t5/user/viewprofilepage/user-id/2083518) Aarssen
comes to the rescue, answering
the [Revit API discussion forum](http://forums.autodesk.com/t5/revit-api-forum/bd-p/160) thread
on [`BuiltInCategory` of doors and curtain wall doors](https://forums.autodesk.com/t5/revit-api-forum/builtincategory-of-doors-and-curtain-wall-doors/m-p/9002988):
\*\*Question:\*\* I want to build a plugin that changes the curtain wall panel types.
When I select a curtain wall door, how can I filter for only curtain wall panel door symbols?
Is there any specific value for curtain panel door?
Snooping a curtain wall panel door in RevitLookup:
![Snoop door in curtain wall](img/door_category_1.png)
A door in a standard wall looks similar:
![Snoop door in standard wall](img/door_category_2.png)
Since the category is the same for all doors, that cannot be used to tell them apart.
Another possible criterion might be to query the door for its host using
its [`Host` property](https://www.revitapidocs.com/2020/69f30141-bd3b-8bdd-7a63-6353d4d495f9.htm).
Using the host element properties, one ought to be able to differentiate curtain walls from others.
However, I am looking at family symbols that have not yet been placed, not instances, so the host is not defined yet.
The `Family` class provides an [IsCurtainPanelFamily property](https://www.revitapidocs.com/2020/da0becae-cb65-fffd-1e97-b4aab5533004.htm),
so I also tried checking

```
  FamilyInstance.Symbol.Family.IsCurtainPanelFamily
```

Unfortunately, `IsCurtainPanelFamily` always returns false, so that does not work either.
\*\*Answer:\*\* From the `FamilySymbol`, you can find similar types using `GetSimilarTypes`.
This returns all curtain wall panels that can be placed in a curtain wall.
Filter that list for the door category:
```csharp
IEnumerable CW_doors
= new FilteredElementCollector(
doc, symbol.GetSimilarTypes() )
.OfCategory( BuiltInCategory.OST_Doors )
.Cast();
```
Many thanks to Frank for solving this!
I added his suggestion
to [The Building Coder samples](https://github.com/jeremytammik/the_building_coder_samples) in
a new method `GetDoorSymbolsForCurtainWall`
in [release 2020.0.147.7](https://github.com/jeremytammik/the_building_coder_samples/compare/2020.0.147.7...2020.0.147.7).
#### SnappingService – What Does it Actually Do?
From
the [Revit API discussion forum](http://forums.autodesk.com/t5/revit-api-forum/bd-p/160) thread
on [`SnappingService` – What it actually does?](https://forums.autodesk.com/t5/revit-api-forum/snappingservice-what-it-actually-does/m-p/8986801)
\*\*Question:\*\* Does anyone have any insight or experience? The API documentation doesn't say much :-(
\*\*Answer:\*\* According to the development team, `SnappingService` provides snapping points and lines and is used for point clouds.
They also expressed an intent to add some documentation for it, and maybe for a couple of other undocumented services as well.
#### Get Title Block Label Parameters
Let's wrap up for today with a beginner's question:
\*\*Question:\*\* I am trying to access these title block label parameters marked in yellow through the Revit API:
![Title block label parameters](img/title_block_label_parameters.png)
I could not find a proper way to access those values in the Revit API documentation.
I'd like to know if this actually possible to achieve or not.
\*\*Answer:\*\* Yes, this is easy to achieve.
In fact, you can probably find out for yourself how to access this data by
installing [RevitLookup](https://github.com/jeremytammik/RevitLookup) and
using that to explore the title block properties.
Are you aware of RevitLookup?
It is a really invaluable tool for Revit add-in development and understanding the structure and contents of the Revit database.
You may also find it out easily for yourself by searching the Internet for an existing solution, e.g., by searching for something
like [revit api title block label parameters](https://duckduckgo.com/?q=revit+api+title+block+label+parameters).
The Building Coder took a look at
the [title block of a sheet](https://thebuildingcoder.typepad.com/blog/2009/11/title-block-of-sheet.html) and
its parameters a long time ago, without providing a method to achieve what you are asking for.
Since I think it is time to take another look at this and it may be useful for others in future as well, I put together the following function that may or may not achieve what you are after:
```csharp
///
/// Read the title block parameters to retrieve the
/// label parameters Sheet Number, Author and Client
/// Name
/// summary>
static void ReadTitleBlockLabelParameters(
Document doc )
{
FilteredElementCollector title_block_instances
= new FilteredElementCollector( doc )
.OfCategory( BuiltInCategory.OST_TitleBlocks )
.OfClass( typeof( FamilyInstance ) );
Parameter p;
Debug.Print( "Title block instances:" );
foreach( FamilyInstance tb in title_block_instances )
{
ElementId typeId = tb.GetTypeId();
Element type = doc.GetElement( typeId );
p = tb.get_Parameter(
BuiltInParameter.SHEET_NUMBER );
Debug.Assert( null != p,
"expected valid sheet number" );
string s_sheet_number = p.AsString();
p = tb.get_Parameter(
BuiltInParameter.PROJECT_AUTHOR );
Debug.Assert( null != p,
"expected valid project author" );
string s_project_author = p.AsValueString();
p = tb.get_Parameter(
BuiltInParameter.CLIENT_NAME );
Debug.Assert( null != p,
"expected valid client name" );
string s_client_name = p.AsValueString();
Debug.Print(
"Title block {0} <{1}> of type {2} <{3}>: "
+ "{4} project author {5} for client {6}",
tb.Name, tb.Id.IntegerValue,
type.Name, typeId.IntegerValue,
s_sheet_number, s_project_author,
s_client_name );
}
}
```
I also added this code
to [The Building Coder samples](https://github.com/jeremytammik/the_building_coder_samples)
in [release 2020.0.147.6](https://github.com/jeremytammik/the_building_coder_samples/compare/2020.0.147.5...2020.0.147.6).
I hope this helps.
Please confirm whether this method does what you are after.
If not, please use RevitLookup to find out exactly what you need and fix it accordingly.