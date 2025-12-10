---
post_number: "0935"
title: "Migrating a Built-in Category and Other Things"
slug: "migrate_category"
author: "Jeremy Tammik"
tags: ['csharp', 'elements', 'family', 'filtering', 'parameters', 'references', 'revit-api', 'rooms', 'views', 'windows']
source_file: "0935_migrate_category.htm"
original_url: "https://thebuildingcoder.typepad.com/blog/0935_migrate_category.html"
---

### Migrating a Built-in Category and Other Things

Here is a category related issue that appeared when migrating from the Revit 2013 to the Revit 2014 API, plus a several other typical simple little Revit API questions from the last day or two:

- [Migrating the room separation lines built-in category](#2)
- [Storage of RGB colour values for materials](#3)
- [Updating a DXF link](#4)
- [Retrieving truss family elements](#5)
- [Retrieving stair stringer data](#6)
- [Storing custom data](#7)

#### Migrating the Room Separation Lines Built-in Category

**Question:** I get the following error while migrating my C# code from Revit 2013 to Revit 2014:

- 'Autodesk.Revit.DB.BuiltInCategory' does not contain a definition for 'OST\_AreaSeparationLines'

It is caused by the following line of code:

```csharp
  ICollection<Element> collection
    = collector.OfCategory(
        BuiltInCategory.OST\_AreaSeparationLines )
      .ToElements();
```

Do you have any solution to suggest?

**Answer:** The easiest way to fix this kind of issue is to look at an element that has this category using RevitLookup in Revit 2013, and then load the same model and see what category it has in Revit 2014.

**Response:** That worked!

It seems to be called OST\_RoomSeparationLines now.

Problem solved.

#### Storage of RGB Colour Values for Materials

**Question:** I looked at a material element 'Color' parameter value today.
This parameter values comes back as an integer.

It appears to me to have the red and blue values reversed.

When looking at the 'Site – Earth' material in the rac\_advanced\_sample\_project included with Revit, the Color **property** on the material class returns material.Color.Red=97, material.Color.Green=75 and material.Color.Blue=62, which is also what the Revit GUI shows in 'Manage Materials'.
I checked on an online colour generator web site and it looks the same as in the Revit GUI, a dirty brown colour.

However, the integer number that comes back from the 'Color' **parameter** on the material element is 4082529, which when converted to RGB values using online converters, comes back as Red=62, Green=75 and Blue=97, which is a bluish colour.
The Red and Blue values appear to be reversed.

I would have expected the integer 6376254 to be returned for this colour instead.

Can you explain this, please?

**Answer:** Yes, easily.

This is actually working as expected.

Revit uses the Windows convention of storing colours as an
[RGB COLORREF value](http://msdn.microsoft.com/en-us/library/windows/desktop/dd183449%28v=vs.85%29.aspx) in
a DWORD (32 bit integer) that specifies "The low-order byte contains a value for the relative intensity of red; the second byte contains a value for green; and the third byte contains a value for blue".

In other words it is (A)BGR and not (A)RGB.

This corresponds to the value of the integer returned from a Color parameter.

#### Updating a DXF Link

**Question:** I have created a DXF link DXF using the following code:

```csharp
  DWGImportOptions opt = new DWGImportOptions();
  ElementId linkId;

  if( doc.Link( "C:\\tmp\\test.dxf",
    opt, doc.ActiveView, out linkId ) )
  {
    Element e = doc.GetElement( linkId );
    ImportInstance i = e as ImportInstance;
  }
```

Later, the DXF is modified by an external application.

Is it possible to programmatically force Revit to reload the link?

I think I could also delete the ImportInstance and create a new one, but I don't how to get the proper DWGImportOptions and DXF file path from the existing ImportInstance.

**Answer:** Yes, this can easily be achieved.
You can use the RevitLinkType.Unload method to unload it first, and then call RevitLinkType.Reload to reload the updated DXF file.

The imported DXF file has the type of CADLinkType.
Just as you say, you could also delete the link and import it again.

You can find the imported DXF ImportInstance's original path by calling the ExternalFileReference.GetPath method, and the
ExternalFileUtils.GetExternalRefernce method will return the ImportInstance's ExternalFileReference object.

#### Retrieving Truss Family Elements

**Question:** How can I retrieve a truss element family instance?

**Answer:** This is a very basic question, answered many times over in the
[Revit API getting started material](http://thebuildingcoder.typepad.com/blog/2013/04/getting-started-with-the-revit-api.html).

I recommend that you take a look at that material, work through the 'My First Revit Plugin' and DevTV tutorials, install the Revit SDK, RevitLookup, RvtSamples, and then answer that question for yourself.

I'll just give you one little additional hint to help get you started:

Every access to a Revit database element is achieved using a filtered element collector.

To retrieve all truss family instances, you could use something like this:

```csharp
  FilteredElementCollector collector
    = new FilteredElementCollector( doc )
      .OfClass( typeof( FamilyInstance ) )
      .OfCategory( BuiltInCategory.OST\_Truss );

  foreach( Element e in collector )
  {
    // e is a truss family instance
  }
```

#### Retrieval of Stair Stringer Data

**Question:** I want to access the values of the stair stringer height and thickness properties on a stair element.
How can I achieve this, please?

I tried using the built-in parameters STAIRS\_ATTR\_STRINGER\_THICKNESS and STAIRS\_ATTR\_STRINGER\_HEIGTH, but the value being returned is null. I am also unable to get the family symbol of the stair element.

**Answer:** The two parameters that you are trying to access are stored on the stair type, not the stair itself.

The stair type is represented by the ElementType class, not FamilySymbol.

You can access the stair type and the two desired paramaters like this:
```csharp
  Document doc = stair.Document;

  ElementId id = stair.GetTypeId();

  ElementType type = doc.get\_Element( id )
    as ElementType;

  Parameter pThick = type.get\_Parameter(
    BuiltInParameter.STAIRS\_ATTR\_STRINGER\_THICKNESS );

  Parameter pHeight = type.get\_Parameter(
    BuiltInParameter.STAIRS\_ATTR\_STRINGER\_HEIGHT );

  double thick = pThick.AsDouble();
  double height = pHeight.AsDouble();
```

#### Storing Custom Data

**Response:** I need more than just the path.
I would like to retrieve other DWGImportOptions parameters as well.
Is it possible to get them somehow?

**Answer:** I don't think there is any way to retrieve all the ImportInstance's DWGImportOptions.

Since the DXF file was imported by your own code, you can easily work around that gap by storing the DWGImportOptions property values of interest in the ImportInstance object.

You can use
[extensible storage](http://thebuildingcoder.typepad.com/blog/2011/04/extensible-storage.html)
([wiki](http://wikihelp.autodesk.com/Revit/enu/2014/Help/3665-Developers/0135-Advanced135/0136-Storing_136/0141-Extensib141),
[category](http://thebuildingcoder.typepad.com/blog/storage))
to store custom data on Revit elements.

When you use this, you can later read back the settings from the extensible storage data.