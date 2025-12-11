---
ai_optimized: true
author: Jeremy Tammik
complexity_score: 7.0
content_type: code_example
optimization_date: '2025-12-11T11:44:15.370904'
original_url: https://thebuildingcoder.typepad.com/blog/1135_categ_type_inst_param.html
post_number: '1135'
reading_time_minutes: 6
series: elements
slug: categ_type_inst_param
source_file: 1135_categ_type_inst_param.htm
tags:
- csharp
- elements
- parameters
- python
- revit-api
- selection
- sheets
- walls
- windows
title: Category Support for Shared Type and Instance Parameters
word_count: 1252
---

### Category Support for Shared Type and Instance Parameters

Welcome to my last post on the Revit 2014 API, and one final new external command and update of The Building Coder sample collection before we migrate to Revit 2015.

Revit 2015 was
[released last week, the Revit SDK was posted and updated](http://thebuildingcoder.typepad.com/blog/2014/04/revit-2015-released.html),
[RevitLookup for Revit 2015 is available on GitHub](http://thebuildingcoder.typepad.com/blog/2014/04/revitlookup-for-revit-2015.html) and the
[DevDays Online presentation](http://thebuildingcoder.typepad.com/blog/2014/04/revit-2015-api-news-devdays-online-recording.html) on
the new API functionality is available.

Before migrating The Building Coder samples from Revit 2014 to Revit 2015, let's add one last external command to it, supporting us in taking a detailed look at how to determine whether a category provides support for instance versus type parameters.

This topic arose in the Revit API discussion forum, where Todd Jacobs of
[Gannett Fleming, Inc.](http://www.gfnet.com/) started a thread to
[determine if a category supports type parameter binding](http://forums.autodesk.com/t5/Revit-API/Determine-if-Category-supports-Type-parameter-binding/m-p/4918068):

**Question:** I can get a list of categories that support bound parameters like this.

```csharp
  SortedList<string, Category> CatList
    = new SortedList<string, Category>();

  Categories cats = doc.Settings.Categories;

  foreach( Category cat in cats )
  {
    if( cat.AllowsBoundParameters )
    {
      CatList.Add( cat.Name, cat );
    }
  }
```

But how can I tell from this list which categories allow Type bound parameters vs. Instance bound parameters?

I would like to produce two lists, similar to the UI for adding project parameters when switching from Type to Instance; the Categories list updates with the available categories for the given selection, not interested in sub-categories at the moment.

As an example:

The Sheets category shows up only when created shared Instance parameters:

![Categories supporting instance parameters](img/category_instance_param_list.png)

It is not displayed at all when selecting Type parameter properties:

![Categories supporting type parameters](img/category_type_param_list.png)

**Answer:** Do the suggestions on
[category analysis with and without Python](http://thebuildingcoder.typepad.com/blog/2014/03/category-analysis-with-and-without-python.html) help?

**Response:** No, not really, unfortunately.

How does Revit do it internally? Is it a static list?

Here is where I want to be:

```csharp
  // Create a Category Set containing only
  // categories that allow Type bound parameters

  Categories cats = doc.Settings.Categories;

  Autodesk.Revit.DB.CategorySet catSetTypeOnly
    = doc.Application.Create.NewCategorySet();

  foreach( Category cat in cats )
  {
    // This property exists.

    if( cat.AllowsBoundParameters )
    {
      // This property is needed.

      if( cat.AllowsTypeBoundParameters )
      {
        catSetTypeOnly.Insert( cat );
      }
    }
  }
```

Then I could produce the same list as Revit for these, like this:

![Categories supporting type and instance parameters](img/category_type_instance_param_list.png)

**Answer:** I am sorry to say the Revit API does not currently provide this functionality, so I submitted the wish list item CF-1079 [API wish: access to CategoryInfo hasSymbols -- 09406242] on your behalf for it.

**Response:** For our immediate needs we developed a set of hardcoded enum's for this task as a workaround.

Please be advised that Equality checks will fail without casting, and Type Safe concerns are not addressed.

Please see attached
[CodeExample.txt](zip/category_type_instance_param_code_example.txt).

**Answer:** Congratulations on putting together these two useful lists, and thank you very much for sharing them.

I simplified your sample code, and pondered your choice of enum.

I wonder whether it is more efficient performance-wise to use an enumeration of a dictionary for looking up the built-in categories to find out whether they support type or instance parameters, respectively.

Basically, this can be seen as two dictionaries mapping a built-in category to a Boolean yes-no value.

Or just mapping to a yes value, and no answer stands for no.

I looked at a stackoverflow discussion on
[enum and performance](http://stackoverflow.com/questions/3256713/enum-and-performance).

In the end, I lean towards the dictionary solution.

I therefore reused your two lists to convert them to dictionaries and define two Boolean predicate methods:

```csharp
  BuiltInCategory bic = ...;

  bool bType = BicSupportsTypeParameters( bic );
  bool bInstance = BicSupportsInstanceParameters( bic );
```

I implemented a new external command CmdCategorySupportsTypeParameter in The Building Coder samples to test them.

I use two simple arrays to store the collections of built-in categories supporting type and instance parameters, respectively:

```csharp
  #region Built-in categories supporting type parameters
  static public BuiltInCategory[]
    \_bicAllowsBoundParametersAsType
      = new BuiltInCategory[]
      {
        ///<summary>Analytical Links</summary>
        BuiltInCategory.OST\_LinksAnalytical,
        ///<summary>Structural Connections</summary>
        BuiltInCategory.OST\_StructConnections,
        ///<summary>Structural Fabric Areas</summary>
        BuiltInCategory.OST\_FabricAreas,
. . .
        ///<summary>Walls</summary>
        BuiltInCategory.OST\_Walls
      };
  #endregion // Built-in categories supporting type parameters
  #region Built-in categories supporting instance parameters
  static public BuiltInCategory[]
    \_bicAllowsBoundParametersAsInstance
      = new BuiltInCategory[]
      {
        ///<summary>Analytical Links</summary>
        BuiltInCategory.OST\_LinksAnalytical,
        ///<summary>Analytical Nodes</summary>
        BuiltInCategory.OST\_AnalyticalNodes,
        ///<summary>Analytical Foundation Slabs</summary>
        BuiltInCategory.OST\_FoundationSlabAnalytical,
. . .
        ///<summary>Walls</summary>
        BuiltInCategory.OST\_Walls
      };
  #endregion // Built-in categories supporting instance parameters
```

They are used to define dictionaries for faster lookup:

```csharp
  static readonly Dictionary<BuiltInCategory, BuiltInCategory>
    \_bicSupportsTypeParameters
      = \_bicAllowsBoundParametersAsType
        .ToDictionary<BuiltInCategory, BuiltInCategory>(
          c => c );

  static readonly Dictionary<BuiltInCategory, BuiltInCategory>
    \_bicSupportsInstanceParameters
      = \_bicAllowsBoundParametersAsInstance
        .ToDictionary<BuiltInCategory, BuiltInCategory>(
          c => c );
```

These in turn are used to define two efficient Boolean predicate lookup functions:

```csharp
  /// <summary>
  /// Return true if the given built-in
  /// category supports type parameters.
  /// </summary>
  static bool BicSupportsTypeParameters(
    BuiltInCategory bic )
  {
    return \_bicSupportsTypeParameters.ContainsKey(
      bic );
  }

  /// <summary>
  /// Return true if the given built-in
  /// category supports instance parameters.
  /// </summary>
  static bool BicSupportsInstanceParameters(
    BuiltInCategory bic )
  {
    return \_bicSupportsInstanceParameters.ContainsKey(
      bic );
  }
```

Here is the external command Execute method implementation to exercise them:

```python
  static string SupportsOrNotString( bool b )
  {
    return b
      ? "supports"
      : "does not support";
  }

  public Result Execute(
    ExternalCommandData revit,
    ref string message,
    ElementSet elements )
  {
    UIApplication uiapp = revit.Application;
    UIDocument uidoc = uiapp.ActiveUIDocument;
    Document doc = uidoc.Document;

    int nCategories = 0;
    int nSupportType = 0;
    int nSupportInstance = 0;
    bool bType, bInstance;

    foreach( BuiltInCategory bic in
      Enum.GetValues( typeof( BuiltInCategory ) ) )
    {
      bType = BicSupportsTypeParameters( bic );
      bInstance = BicSupportsInstanceParameters( bic );

      ++nCategories;
      nSupportType += bType ? 1 : 0;
      nSupportInstance += bInstance ? 1 : 0;

      Debug.Print( "{0} {1} instance and {2} type parameters",
        bic,
        SupportsOrNotString( bInstance ),
        SupportsOrNotString( bType ) );
    }

    string caption = "Categories supporting type "
      + "and instance parameters";

    string msg = string.Format(
      "Tested {0} built-in categories "
      + "in total, {1} supporting instance and {2} "
      + "supporting type parameters.", nCategories,
      nSupportInstance, nSupportType );

    Debug.Print( "\n" + caption + ":\n" + msg );

    TaskDialog.Show( caption, msg );

    return Result.Succeeded;
  }
```

It tests their return value for each and every built-in category enumeration value and prints a report like this to the Visual Studio debug output window:

```
  OST_StackedWalls_Obsolete_IdInWrongRange does not
    support instance and does not support type parameters
  OST_MassTags_Obsolete_IdInWrongRange does not
    support instance and does not support type parameters
  OST_MassSurface_Obsolete_IdInWrongRange
    does not support instance and does not support type parameters
  . . .
  OST_LinksAnalytical supports instance and
    supports type parameters
  . . .
  OST_AnalyticalNodes supports instance and
    does not support type parameters
  . . .
  OST_MatchAll does not support instance and
    does not support type parameters
  INVALID does not support instance and
    does not support type parameters

  Categories supporting type and instance parameters:
  Tested 919 built-in categories in total,
  128 supporting instance and 91 supporting type parameters.
```

I do not see any categories at all that support type parameters and not instance parameters.

Here is the task dialogue summarising the results:

![Report on categories supporting type and instance parameters](img/category_type_instance_param_report.png)

I published the full source code in
[The Building Coder samples GitHub repository](https://github.com/jeremytammik/the_building_coder_samples).

The version discussed above is
[release 2014.0.109.0](https://github.com/jeremytammik/the_building_coder_samples/releases/tag/2014.0.109.0).

Many thanks to Todd for providing these two important lists of categories.