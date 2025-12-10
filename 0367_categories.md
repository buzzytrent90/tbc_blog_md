---
post_number: "0367"
title: "Categories"
slug: "categories"
author: "Jeremy Tammik"
tags: ['csharp', 'elements', 'family', 'levels', 'parameters', 'references', 'revit-api', 'rooms', 'selection']
source_file: "0367_categories.htm"
original_url: "https://thebuildingcoder.typepad.com/blog/0367_categories.html"
---

### Categories

We have looked at many aspects of categories in the past, such as
[category comparison](http://thebuildingcoder.typepad.com/blog/2009/06/category-comparison-and-model-element-selection-revisited.html),
[family categories](http://thebuildingcoder.typepad.com/blog/2009/07/get-and-set-family-category-and-parameters.html),
[categories and parameter bindings](http://thebuildingcoder.typepad.com/blog/2009/09/adding-a-category-to-a-parameter-binding.html), and
[system versus user defined categories](http://thebuildingcoder.typepad.com/blog/2009/10/system-versus-user-family-category.html).

Here is very basic question that just came in and may be of general interest:

**Question:** Is it possible to obtain a complete list of Revit family categories?

**Answer:** Yes, sure it is.
You can create it yourself.
There are two sets of categories in a Revit project document: the ones defined in the document itself, and the pre-defined built-in ones.

The top-level document ones are available from the document settings categories property, and the built-in ones from the BuiltInCategory enumeration.

I implemented a new Building Coder sample command CmdCategories to list the contents of these two collections.

This is the code to read the document categories.
The Categories object is a map that contains all the top-level Category objects within the Document:
```csharp
  UIApplication app = commandData.Application;
  Document doc = app.ActiveUIDocument.Document;
  Categories categories = doc.Settings.Categories;

  int n = categories.Size;

  Debug.Print( "{0} categories and their parents:", n );

  foreach( Category c in categories )
  {
    Category p = c.Parent;

    Debug.Print( "  {0} ({1}), parent {2}",
      c.Name, c.Id.IntegerValue,
      (null == p ? "<none>" : p.Name) );
  }
```

This lists 219 categories in the sample project I tested it in, starting like this:

```
219 categories and their parents:
  Plumbing Fixture Tags (-2005010), parent <none>
  Switch System (-2008101), parent <none>
  Structural Framing Tags (-2005015), parent <none>
  Room Tags (-2000480), parent <none>
  Raster Images (-2000560), parent <none>
  Security Devices (-2008079), parent <none>
  Structural Beam System Tags (-2005130), parent <none>
```

Since all of these are top-level categories, the Parent property is always null.
You can still ask the map for a sub-category by name or id and it should return it to you.
You can also extract the sub-categories of each parent category you iterate.

Here is the code iterating over the built-in categories listed in the BuiltInCategory enumeration:
```csharp
  Array bics = Enum.GetValues(
    typeof( BuiltInCategory ) );

  n = bics.Length;

  Debug.Print( "{0} built-in categories and the "
    + "corresponding document ones:", n );

  Category cat;
  string s;

  foreach( BuiltInCategory bic in bics )
  {
    try
    {
      cat = categories.get\_Item( bic );

      s = (null == cat)
        ? "<none>"
        : string.Format( "--> {0} ({1}",
          cat.Name, cat.Id.IntegerValue );
    }
    catch( Exception ex )
    {
      s = ex.GetType().Name + " " + ex.Message;
    }
    Debug.Print( "  {0} {1}", bic.ToString(), s );
  }
```

This lists 747 built-in categories and their corresponding document ones, if found.

I had to add an exception handler around the call to the Categories collection Item property taking a BuiltInCategory argument, since I discovered that it throws exceptions when called with certain built-in categories:

- A InvalidOperationException with an empty message on the built-in categories
  - OST\_FurnitureHiddenLines- OST\_HostTemplate InvalidOperationException- OST\_MassFaceSplitter InvalidOperationException- OST\_MassCutter InvalidOperationException- OST\_ZoningEnvelope InvalidOperationException- A NullReferenceException with a message saying "Object reference not set to an instance of an object" on
    - OST\_InstanceDrivenLineStyle- OST\_CurtainGridsSystem- OST\_IOSNotSilhouette- OST\_InvisibleLines

This is due to the fact that a few categories can't be stored in the map correctly because their names are the same as other categories, and the name is used as a key.
A workaround might be: obtain the category from a new Categories object (from Settings.Categories), get the category directly by BuiltInCategory from that object.

Here is a
[text file](zip/367_categories.txt) including
the output of running this command in my simple sample project.

Here is
[version 2011.0.67.0](zip/bc_11_67.zip)
of The Building Coder sample source code and Visual Studio solution including the new command.

The Revit 2010 version of the Revit API introduction labs also included a command Lab2\_5\_Categories which performed some further analysis on the document and built-in categories and their relationships with each other.
The last version we published was in the discussion on
[selecting model elements](http://thebuildingcoder.typepad.com/blog/2009/11/select-model-elements-2.html).