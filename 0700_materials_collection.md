---
post_number: "0700"
title: "Materials Collection and Filtering"
slug: "materials_collection"
author: "Jeremy Tammik"
tags: ['csharp', 'elements', 'filtering', 'parameters', 'python', 'references', 'revit-api', 'transactions']
source_file: "0700_materials_collection.htm"
original_url: "https://thebuildingcoder.typepad.com/blog/0700_materials_collection.html"
---

### Materials Collection and Filtering

We already mentioned that null entries may be returned when
[retrieving materials](http://thebuildingcoder.typepad.com/blog/2011/08/retrieving-materials.html) from
the document settings Materials collection.
This raised a couple of more points, e.g. more on the
[null entries](#2) and
[using LINQ and filtered element collectors](#3) to
retrieve materials.
In the same context, we can also look at
[retrieving all materials used by an element](#4) and
[removing the deprecated material related API calls](#5) in
The Building Coder samples.

#### Materials Collection May Contain Null Entries

Quite a while ago, Rudolf Honke observed that the Document.Settings.Materials collection may contain empty entries.

This came up in a case where he wanted to list all the Materials sorted by name.
The materials returned from the collection may have null values, in which case trying to access their name will obviously throw an exception:

![Null entry in Materials collection](img/materials_null.png)

In this case, you can simply skip the invalid material entries before sorting them, because otherwise the null values will make the Sort method crash.

Here is an example of how this can be handled to create a sorted generic .NET list of materials:
```csharp
  /// <summary>
  /// Return a list of the document's
  /// non-null materials sorted by name.
  /// </summary>
  List<Material> GetSortedMaterials( Document doc )
  {
    Materials doc\_materials = doc.Settings.Materials;
    int n = doc\_materials.Size;

    List<Material> materials\_sorted
      = new List<Material>( n );

    foreach( Material m in doc\_materials )
    {
      if( m != null )
      {
        materials\_sorted.Add( m );
      }
    }
    materials\_sorted.Sort(
      delegate( Material m1, Material m2 )
      {
        return m1.Name.CompareTo( m2.Name );
      }
    );
    return materials\_sorted;
  }
```

As said, though, you can also use the filtered element collector to retrieve materials, e.g. using
```csharp
  new FilteredElementCollector( doc )
    .OfClass( typeof( Material ) );
```

#### Material Filtering Using LINQ and Filtered Element Collectors

Victor Chekalin reported another issue in his
[comment](http://thebuildingcoder.typepad.com/blog/2011/08/retrieving-materials.html?cid=6a00e553e1689788330162fde14500970d#comment-6a00e553e1689788330162fde14500970d) on
[retrieving materials](http://thebuildingcoder.typepad.com/blog/2011/08/retrieving-materials.html):

**Question:** I've got a NullReferenceException when I used document.Settings.Materials.get\_Item("Concrete");

This error occurred in a big real project.
I discovered that the Document.Materials collection contains null items.
How is it possible?
Is it a Revit internal database error or something else?

I solved my problem using Linq:
```csharp
  var material = document
    .Settings
    .Materials
    .OfType<Material>()
    .FirstOrDefault( m => m.Name.Equals( "Concrete" ) );
```

**Answer:** The materials collection in the Document.Settings.Materials property may indeed contain empty entries.

You need to check for null values and skip these invalid materials before trying to access them.

I guess that in your code snippet, the OfType filter eliminates the null entries.

**Response:** Yes, of course I skip null entries when I iterate materials in loop or using Ling query.

But Materials.get\_Item(string) it is a standard Revit method and it doesn't work when collection contains null entries.
So this needs to be fixed.

I used this method many times in my project because I thought it would work correctly and it must work in spite of the collection containing nulls.
But now, unfortunately for me, I must rewrite my code and change this method to my own :-(

One more problem that I'm interesting in: why does the Materials.get\_Item method require an open transaction?
I'm just reading some data.
For example, when I read Element parameters, no transaction is required.

**Answer:** As you may be aware, we have been if the process of replacing the material management in Revit by a new cross-product Autodesk materials library for several releases now.
This will enhance interoperability between the products and provide many other benefits as well.
For now, the simplest and most effective solution is to ignore the Materials collection and use a filtered element collector instead as described above.

**Response:** Yes, you are right.
The FilteredElementCollector works without errors.

Here is a suggestion for extension methods to retrieve the collection of all materials and an individual material by name:
```python
  public static class DocumentExtensions
  {
    public static IEnumerable<Material> GetMaterials(
      this Document doc )
    {
      FilteredElementCollector collector
        = new FilteredElementCollector( doc );

      return collector
        .OfClass( typeof( Material ) )
        .OfType<Material>();
    }

    public static Material GetMaterialByName(
      this Materials materials,
      string materialName )
    {
      return materials
        .OfType<Material>()
        .FirstOrDefault(
          m => m.Name.Equals( materialName ) );
    }
  }
```

They can be used like this:
```csharp
  var materialsInDocument = doc.GetMaterials();

  var concreteMaterial = doc.Settings.Materials
    .GetMaterialByName( materialName );
```

However, I can now see some new interesting effects :-)

The number of materials retrieved via the collector differs from the Settings.Materials count.
In my test project, the collector retrieves 894 materials instead 853 materials in Settings.Materials.

Do you know why?

#### Retrieving Element and Material Relationships

To wrap this up, here is another question from the
[comment](http://thebuildingcoder.typepad.com/blog/2011/08/retrieving-materials.html#comment-6a00e553e16897883301675ec00258970b) by
[Dan Tartaglia](http://www.facebook.com/dan.tartaglia1) on
the same post:

**Question:** Is there an easy way to know if a material is being used by an object in the active file?

**Answer:** I am not aware of any other method than iterating over all elements and checking what they use, which is presumably exactly what you are **not** looking for.

Victor replied and added an example implementation:

As Jeremy says, the only way is to iterate all materials in all elements.

I use the following code to get all elements used by material:
```csharp
  var elementsWithMaterials =
    ( from el in elements
      from Material m in el.Materials
      select new ElementMaterial( el, m ) )
    .ToList();
```

Here I get element and materials in each element collection:
```csharp
  var groupedElementsInMaterial
    = elementsWithMaterials
      .GroupBy( x => x.Material, new ElementComparer() )
      .OrderBy( x => x.Key.Name );
```

Here, I group elements by material.

This gives the following nice tri-lingual result:

![Elements grouped by material](img/materials_grouped.png)

#### Cleaned Up Building Coder Samples

While we are at it, here is
[version 2012.0.96.3](zip/bc_12_96_3.zip) of
The Building Coder samples with the deprecated material related methods in CmdGetMaterials.cs replaced by new code which no longer produces any warnings in Revit the 2012 API.
The obsolete lines are marked with comments saying '// 2011', and their replacements are marked '// 2012'.

There are still compilation warnings due to use of one deprecated method FindReferencesByDirection, which I plan to address soon as well.