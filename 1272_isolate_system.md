---
post_number: "1272"
title: "Isolating Elements of a Given System"
slug: "isolate_system"
author: "Jeremy Tammik"
tags: ['csharp', 'elements', 'filtering', 'parameters', 'references', 'revit-api', 'selection', 'transactions', 'views']
source_file: "1272_isolate_system.htm"
original_url: "https://thebuildingcoder.typepad.com/blog/1272_isolate_system.html"
---

### Isolating Elements of a Given System

I am very glad to present another post contributed by Victor Chekalin, or Виктор Чекалин, who already shared many valuable insights and in-depth Revit API research in the past:

On the [Russian developer forum](http://adn-cis.org/forum/index.php?topic=1795.0), the user *goblya* asked an interesting question. He needs to hide all elements except those that belong to a specific given system.

There are some interesting moments in the solution.

The main idea is to create the filter, assign this filter to the active view and set visibility to false for this filter. The main problem is how to create a suitable filter.

User almost solved it. Here is his initial code:

```csharp
  Parameter Namesystem = elem.get\_Parameter(
    BuiltInParameter.RBS\_SYSTEM\_NAME\_PARAM );

  View view = doc.ActiveView;

  IList categories = new List();
  categories.Add( new ElementId(
    BuiltInCategory.OST\_PlaceHolderDucts ) );

  categories.Add( new ElementId(
    BuiltInCategory.OST\_DuctLinings ) );

  categories.Add( new ElementId(
    BuiltInCategory.OST\_DuctInsulations ) );

  categories.Add( new ElementId(
    BuiltInCategory.OST\_DuctTerminal ) );

  categories.Add( new ElementId(
    BuiltInCategory.OST\_MechanicalEquipment ) );

  IList rules = new List();

  rules.Add( ParameterFilterRuleFactory
    .CreateNotContainsRule( Namesystem.Id,
      Namesystem.AsString(), true ) );

  ParameterFilterElement filter = null;

  using( Transaction t = new Transaction(
    doc, "Create and Apply Filter" ) )
  {
    t.Start();
    filter = ParameterFilterElement.Create(
      doc, "2222", categories, rules );
    view.AddFilter( filter.Id );
    t.Commit();
  }

  using( Transaction t = new Transaction(
    doc, "Set Visibility Appearance" ) )
  {
    t.Start();
    view.SetFilterVisibility( filter.Id, false );
    t.Commit();
  }
```

Nevertheless, some of the elements were still not hidden, as he missed some categories.

So the challenge is to create the filter.

In the filter, you always have to set categories. In theory, it is possible to get all categories whose names start with 'OST\_Duct', but in that case it will work only with Mechanical systems.

The task requires us to hide absolutely all elements. We should not consider categories in this case at all.

However, the method ParameterFilterElement.Create requires the list of categories anyway. That means we have to pass all possible categories to the method.

To get all categories very easy:

```csharp
  var allCategories =
    doc.Settings.Categories
    .OfType<Category>()
    .Select( c => c.Id )
    .ToList();

  var filter = ParameterFilterElement.Create(
    doc,
    "All elements except the system " + system.Name ),
    allCategories,
    rules );
```

Failed.

This produces the exception 'One of the given categories is not filterable'.

OK. Not all categories can be included to the filter.

Somehow we need to get only the categories what can be applied to the filter.

After some researching of the RevitAPI.chm, I found a useful class, ParameterFilterUtilities.

The method ParameterFilterUtilities.GetAllFilterableCategories returns exactly that we need.

The second attempt:

```csharp
  var filter = ParameterFilterElement.Create(
    doc,
    string.Format(
      "\_\_\_ \_\_\_\_\_\_\_\_, \_\_\_\_\_ \_\_\_\_\_\_\_ {0}",
      system.Name ),
    ParameterFilterUtilities.GetAllFilterableCategories(),
    rules );
```

Failed again.

One of the given rules refers to a parameter that doesn't apply to this filter's categories.

It seems like we need two filters:

- All elements that does not have the parameter 'System Name'
- All elements that have the parameter 'System Name' and the value of the parameter does not contain the specific system name.

Looking again to the methods of the class ParameterFilterUtilities.

There is no method that does exactly what we need.

Nevertheless, there are some methods we can use to get there.

The method ParameterFilterUtilities.GetFilterableParametersInCommon returns the list of the parameters applicable to the given categories.

For any category we can check whether we can use the specific parameter in that category or not.

As a result, I created the following method:

```csharp
  /// <summary>
  /// Returns the list of the categories what can be
  /// used in filter by the specific parameter
  /// </summary>
  /// <param name="doc">Document on which the filter
  /// is applying</param>
  /// <param name="bip">BuiltInParameter</param>
  /// <param name="inverse">If true, the list will
  /// be inverted. I.e. you will get the list of the
  /// categories what cannot be used in filter by
  /// the specific parameter </param>
  /// <returns></returns>
  ICollection<ElementId>
    GetCategoriesApplicableForParameter(
      Document doc,
      BuiltInParameter bip,
      bool inverse = false )
  {
    // All categories available for filter.

    var allCategories = ParameterFilterUtilities
      .GetAllFilterableCategories();

    ICollection<ElementId> retResult
      = new List<ElementId>();

    foreach( ElementId categoryId in allCategories )
    {
      // Get the list of paramteres
      // compatible with the category.

      var applicableParameters
        = ParameterFilterUtilities
          .GetFilterableParametersInCommon(
            doc, new[] { categoryId } );

      // If the parameter we are interested in is
      // in the collection, add it to the result.

      if( applicableParameters.Contains(
        new ElementId( bip ) ) )
      {
        retResult.Add( categoryId );
      }
    }

    // Invert if needed.

    if( inverse )
    {
      retResult = allCategories.Where(
        x => !retResult.Contains( x ) ).ToList();
    }
    return retResult;
  }
```

Back to our task.

To get all categories that can be applied in the filter and allow to filter by 'System Name' parameter, we can use this code:

```csharp
  var categoriesWithSystem =
    GetCategoriesApplicableForParameter( doc,
      BuiltInParameter.RBS\_SYSTEM\_NAME\_PARAM );
```

And the list of categories without 'System Name' parameter.

```csharp
  var categoriesWithoutSystemNameParameter =
    GetCategoriesApplicableForParameter( doc,
      BuiltInParameter.RBS\_SYSTEM\_NAME\_PARAM, true );
```

We need to remember that an element can belong to several systems.

In this case we have to consider all of them and create several filter rules:

```csharp
  // An element can be assigned to the several systems.
  // In this case System Name parameter has a comma-
  // separated list of the systems.
  // Create several rules.

  IList<FilterRule> rules = new List<FilterRule>();

  var systems = systemName.Split( new[] { ',' },
    StringSplitOptions.RemoveEmptyEntries );

  foreach( var system in systems )
  {
    rules.Add(
      ParameterFilterRuleFactory.CreateNotContainsRule(
      new ElementId( BuiltInParameter.RBS\_SYSTEM\_NAME\_PARAM ),
      system.Trim(), true ) );
  }
```

The full code of the command:

```csharp
public Result Execute(
  ExternalCommandData commandData,
  ref string message,
  ElementSet elements )
{
  UIApplication uiapp = commandData.Application;
  UIDocument uidoc = uiapp.ActiveUIDocument;
  Application app = uiapp.Application;
  Document doc = uidoc.Document;

  Reference r;

  try
  {
    r = uidoc.Selection.PickObject(
      ObjectType.Element,
      new SystemElementFilter(),
      "Select an alement of a system" );
  }
  catch( OperationCanceledException )
  {
    return Result.Cancelled;

  }

  var elem = doc.GetElement( r.ElementId );

  var systemNameParam =
    elem.get\_Parameter(
      BuiltInParameter.RBS\_SYSTEM\_NAME\_PARAM );

  if( systemNameParam == null )
  {
    message = "How did you do that?";
    return Result.Failed;
  }

  var view = doc.ActiveView;

  var categoriesWithSystem =
    GetCategoriesApplicableForParameter( doc,
      BuiltInParameter.RBS\_SYSTEM\_NAME\_PARAM );

  var categoriesWithoutSystemNameParameter =
    GetCategoriesApplicableForParameter( doc,
      BuiltInParameter.RBS\_SYSTEM\_NAME\_PARAM, true );

  var systemName = systemNameParam.AsString();

  // An element can be assigned to the several systems.
  // In this case System Name parameter has a comma-
  // separated list of the systems.
  // Create several rules.

  IList<FilterRule> rules = new List<FilterRule>();

  var systems = systemName.Split( new[] { ',' },
    StringSplitOptions.RemoveEmptyEntries );

  foreach( var system in systems )
  {
    rules.Add(
      ParameterFilterRuleFactory.CreateNotContainsRule(
      new ElementId( BuiltInParameter.RBS\_SYSTEM\_NAME\_PARAM ),
      system.Trim(), true ) );
  }

  using( var t = new Transaction( doc ) )
  {
    t.Start( "System isolate" );

    // Hide all elements which do no have a System
    // Name parameter. Do not use this filter if
    // you want to hide only the systems.

    ParameterFilterElement filter1 =
      ParameterFilterElement.Create( doc,
        "All elements without System Name parameter",
        categoriesWithoutSystemNameParameter );

    view.AddFilter( filter1.Id );
    view.SetFilterVisibility( filter1.Id, false );

    // Hide elements which are not
    // in the selected system

    ParameterFilterElement filter2 =
      ParameterFilterElement.Create( doc,
        string.Format(
          "All elements not in the systems {0}",
          systemName ),
        categoriesWithSystem,
        rules );

    view.AddFilter( filter2.Id );
    view.SetFilterVisibility( filter2.Id, false );

    t.Commit();
  }
  return Result.Succeeded;
}
```

If you do not want to isolate architecture, and want to isolate only systems, do not apply the first filter.

Here are some results of running the command.

Before:

![Before](img/vc_isolate_system_elements_1.png)

After:

![After](img/vc_isolate_system_elements_2.png)

With the element in several systems.

Before:

![Before](img/vc_isolate_system_elements_3.png)

After:

![After](img/vc_isolate_system_elements_4.png)

The full source code is on [GitHub](https://github.com/chekalin-v/HideRevitSystems.git) and in the [archive](https://github.com/chekalin-v/HideRevitSystems/archive/master.zip).

Many thanks to Victor for this interesting discussion and useful result!