---
ai_optimized: true
author: Jeremy Tammik
complexity_score: 11.2
content_type: code_example
optimization_date: '2025-12-11T11:44:15.028375'
original_url: https://thebuildingcoder.typepad.com/blog/0974_family_api_addin.html
post_number: 0974
reading_time_minutes: 10
series: family
slug: family_api_addin
source_file: 0974_family_api_addin.htm
tags:
- csharp
- doors
- elements
- family
- filtering
- parameters
- python
- references
- revit-api
- selection
- transactions
- views
title: Family API Add-in
word_count: 2001
---

### Family API Add-in

Welcome to the second part of the detailed discussion on use of the Family API in the project environment, covering scenario 2 from the following list of topics:

- [Family editor product functionality](http://thebuildingcoder.typepad.com/blog/2013/06/key-concepts-of-the-family-editor.html)

- [Key family concepts](http://thebuildingcoder.typepad.com/blog/2013/06/key-concepts-of-the-family-editor.html#2)
- [Building your first parametric Revit family](http://thebuildingcoder.typepad.com/blog/2013/06/key-concepts-of-the-family-editor.html#2b)
- [Slide deck bullets](http://thebuildingcoder.typepad.com/blog/2013/06/key-concepts-of-the-family-editor.html#3)
- [Complete slide decks](http://thebuildingcoder.typepad.com/blog/2013/06/key-concepts-of-the-family-editor.html#4)
- [Family API topics](http://thebuildingcoder.typepad.com/blog/2013/06/key-concepts-of-the-family-editor.html#5)

- [API scenario 1 – load family and place instances](http://thebuildingcoder.typepad.com/blog/2013/06/family-api-add-in-load-family-and-place-instances.html#10)

- [Checking whether a family is loaded](http://thebuildingcoder.typepad.com/blog/2013/06/family-api-add-in-load-family-and-place-instances.html#11)
- [Find a database element by type and name](http://thebuildingcoder.typepad.com/blog/2013/06/family-api-add-in-load-family-and-place-instances.html#12)
- [Loading a family](http://thebuildingcoder.typepad.com/blog/2013/06/family-api-add-in-load-family-and-place-instances.html#13)
- [Placing family instances](http://thebuildingcoder.typepad.com/blog/2013/06/family-api-add-in-load-family-and-place-instances.html#14)
- [Accessing the newly placed instances](http://thebuildingcoder.typepad.com/blog/2013/06/family-api-add-in-load-family-and-place-instances.html#15)

- [API scenario 2 – select and modify instances](#20)

- [Creating a new family type](#21)
- [Selecting instances with pre- and post-selection](#22)
- [Modifying a family instance symbol](#23)

- [API scenario 3 – instance and symbol retrieval, nested type modification](http://thebuildingcoder.typepad.com/blog/2013/07/family-api-nested-type-instance-and-symbol-retrieval.html#30)

- [Retrieve specific family symbols](http://thebuildingcoder.typepad.com/blog/2013/07/family-api-nested-type-instance-and-symbol-retrieval.html#31)
- [Retrieve specific family instances](http://thebuildingcoder.typepad.com/blog/2013/07/family-api-nested-type-instance-and-symbol-retrieval.html#32)
- [Display available door panel types](http://thebuildingcoder.typepad.com/blog/2013/07/family-api-nested-type-instance-and-symbol-retrieval.html#33)
- [Modify a nested family type](http://thebuildingcoder.typepad.com/blog/2013/07/family-api-nested-type-instance-and-symbol-retrieval.html#34)

- [External application](http://thebuildingcoder.typepad.com/blog/2013/07/family-api-nested-type-instance-and-symbol-retrieval.html#40)
- [Conclusion and download](http://thebuildingcoder.typepad.com/blog/2013/07/family-api-nested-type-instance-and-symbol-retrieval.html#50)

The product functionality was presented by Steven Campbell in his classes on the
[key family concepts](http://thebuildingcoder.typepad.com/blog/2013/06/key-concepts-of-the-family-editor.html) at the
[Revit API DevCamp in Moscow](http://www.autodesk.ru/adsk/servlet/pc/index?id=21516340&siteID=871736),
and API scenario 1 is covered by the previous post on
[loading a family and placing instances](http://thebuildingcoder.typepad.com/blog/2013/06/family-api-add-in-load-family-and-place-instances.html).

Scenario 3 and the external application remain to be discussed tomorrow.

#### API Scenario 2 – Selecting and Modifying Instances

We demonstrate the modification of selected family instances by creating a new type for the previously loaded family and assigning the resulting symbol to them.

Before creating the new family type by applying the Duplicate method to an existing one, we have to check whether it already exists.

The selection of the instances to modify supports both pre- and post- selection.

#### Creating a New Family Type

Creation of a new family type for an already loaded family is easy, because we can simply call the Duplicate method on one of its existing types.

The new type is created with the specified name and its parameter values are set accordingly.

In this case, create a new table symbol and modify its length, width and depth.
All three of these specify a distance, which is always stored internally in the Revit database using
[imperial units](http://thebuildingcoder.typepad.com/blog/2011/03/internal-imperial-units.html),
i.e. feet.

Assuming that we wish to set the three parameters using metric values given in millimetres, we need to apply a conversion factor.
Here is the helper method SetElementParameterInMm that sets the value of the named parameter on the given element to a new length specified in millimetres:

```csharp
  /// <summary>
  /// Set the value of the named parameter
  /// on the given element to a new length
  /// specified in millimetres.
  /// </summary>
  void SetElementParameterInMm(
    Element e,
    string parameter\_name,
    double lengthInMm )
  {
    e.get\_Parameter( parameter\_name )
      .Set( Util.MmToFoot( lengthInMm ) );
  }
```

Furthermore, we set the material of the new table symbol to glass.
The material is specified by setting the values of the "Top material" and "Leg material" parameters of the symbol to the element id of the corresponding Material database element.

This is implemented by the following CreateNewType method taking the existing family symbol as its input argument:

```csharp
  /// <summary>
  /// Create new table type
  /// Type Name: 1200x750x380mm
  /// Parameters to change:
  /// Width = 1200
  /// Depth = 750
  /// Height = 380
  /// Top Material = Glass
  /// Leg Material = Glass
  /// </summary>
  FamilySymbol CreateNewType( FamilySymbol oldType )
  {
    FamilySymbol sym = oldType.Duplicate(
      \_type\_name ) as FamilySymbol;

    SetElementParameterInMm( sym, "Width", 1200 );
    SetElementParameterInMm( sym, "Depth", 750 );
    SetElementParameterInMm( sym, "Height", 380 );

    Element material\_glass = Util.FindElementByName(
      sym.Document, typeof( Material ), "Glass" );

    ElementId id = material\_glass.Id;

    sym.get\_Parameter( "Top Material" ).Set( id );
    sym.get\_Parameter( "Leg Material" ).Set( id );

    return sym;
  }
```

As said, before calling this method, we need to ensure that the new symbol has not already been created.

The retrieval of a family symbol can make use of the
[FindElementByName helper method](http://thebuildingcoder.typepad.com/blog/2013/06/family-api-add-in-load-family-and-place-instances.html#12) that
we already used to check for the existence of the family itself before attempting to load it into the project.

In this case, we search for a FamilySymbol class instance with the target type name.

Again, additional filters could be applied to optimise the retrieval in a big project containing a large number of instances, e.g. by adding a category and possibly other criteria, preferably using quick filters, if possible.

The name comparison could also be speeded up by converting it to a parameter filter instead of using .NET post-processing of the results returned by the filtered element collector.

The call to the helper function and potential call to create a new type can be implemented in one single statement making use of the .NET null coalescing operator '??':

```csharp
  // Retrieve existing type or create new

  FamilySymbol symbol
    = Util.FindElementByName( doc,
      typeof( FamilySymbol ), \_type\_name )
        as FamilySymbol
    ?? CreateNewType( tables[0].Symbol );
```

#### Selecting Instances with Pre- and Post-selection

However, before we proceed to create and apply the new type, we handle the selection process of the family instances to modify.

Pre-selection is handled by retrieving the element contained in the UIDocument Selection collection.

Post-selection can be implemented using the PickObjects method, and its ease of use is vastly enhanced by providing a selection filter implementing the ISelectionFilter interface to disable picking any undesired elements.

I implemented a complete pre- and post-selection handler class named TableSelector for this.
The impact to make use of it in the external command mainline is reduced to the pure minimal:

```csharp
  TableSelector selector
    = new TableSelector( uidoc );

  IList<FamilyInstance> tables
    = selector.SelectedTables;

  if( null == tables )
  {
    return selector.Return();
  }
```

As you can see, the TableSelector interface just requires three public methods:

- The constructor, doing all the work.
- The SelectedTables property returning the result.
- The Return method, called to return in case of failure or cancellation and possibly report an error, if needed.

It defines a common filtering helper method IsTable that it uses both to implement the internal ISelectionFilter and to check the validity of pre-selected elements.

This specific
implementation requires a family instance element
of the furniture category belonging to the named
family:

```csharp
static public bool IsTable( Element e )
{
  bool rc = false;

  Category cat = e.Category;

  if( null != cat )
  {
    if( cat.Id.IntegerValue.Equals(
      (int) BuiltInCategory.OST\_Furniture ) )
    {
      FamilyInstance fi = e as FamilyInstance;

      if( null != fi )
      {
        string fname = fi.Symbol.Family.Name;

        rc = fname.Equals(
          CmdTableLoadPlace.FamilyName );
      }
    }
  }
  return rc;
}
```

By making use of this helper method, the selection filter implementation becomes completely trivial:

```python
class TableSelectionFilter : ISelectionFilter
{
  public bool AllowElement( Element e )
  {
    return IsTable( e );
  }

  public bool AllowReference( Reference r, XYZ p )
  {
    return true;
  }
}
```

The rest of the TableSelector class implementation just has to define the member data, and the three public interfaces:

```python
class TableSelector
{
  const string \_usage\_error = "Please pre-select "
    + "only table family instances to modify "
    + "before launching this command.";

  List<FamilyInstance> \_tables;
  string \_msg;
  Result \_result;

  /// <summary>
  /// Return selected tables or null
  /// </summary>
  public IList<FamilyInstance> SelectedTables
  {
    get
    {
      return \_tables;
    }
  }

  /// <summary>
  /// Instantiate and run table selector
  /// </summary>
  public TableSelector( UIDocument uidoc )
  {
    \_tables = null;
    \_msg = null;

    Document doc = uidoc.Document;

    if( null == doc )
    {
      \_msg = "Please run this command in a valid"
        + " Revit project document.";
      \_result = Result.Failed;
    }

    // Check for a pre-selected tables

    Selection sel = uidoc.Selection;

    int n = sel.Elements.Size;

    if( 0 < n )
    {
      if( 1 != n )
      {
        \_msg = \_usage\_error;
        \_result = Result.Failed;
      }

      foreach( Element e in sel.Elements )
      {
        if( !IsTable( e ) )
        {
          \_msg = \_usage\_error;
          \_result = Result.Failed;
        }

        if( null == \_tables )
        {
          \_tables = new List<FamilyInstance>( n );
        }

        \_tables.Add( e as FamilyInstance );
      }
    }

    // If no tables were pre-selected,
    // prompt for post-selection

    if( null == \_tables
      || 0 == \_tables.Count )
    {
      IList<Reference> refs = null;

      try
      {
        refs = sel.PickObjects( ObjectType.Element,
          new TableSelectionFilter(),
          "Please select tables to be modified." );
      }
      catch( Autodesk.Revit.Exceptions
        .OperationCanceledException )
      {
        \_result = Result.Cancelled;
      }

      if( null != refs && 0 < refs.Count )
      {
        \_tables = new List<FamilyInstance>(
          refs.Select<Reference, FamilyInstance>(
            r => doc.GetElement( r.ElementId )
              as FamilyInstance ) );
      }
    }

    Debug.Assert(
      null == \_tables || 0 < \_tables.Count,
      "ensure that we do not return a non-null but empty collection" );

    \_result = (null == \_tables) // || 0 == \_tables.Count
      ? Result.Cancelled
      : Result.Succeeded;
  }

  /// <summary>
  /// Return the cancellation or failure code
  /// to Revit and display a message if there is
  /// anything to say.
  /// </summary>
  public Result Return()
  {
    if( Result.Failed == \_result )
    {
      Debug.Assert( 0 < \_msg.Length,
        "expected error message" );

      Util.ErrorMsg( \_msg );
    }
    return \_result;
  }
}
```

Please note that the isolation of all the validity checking in the one single IsTable filtering helper method makes it very simple to generalise this class, which is one of my (many) next plans.

#### Modifying a Family Instance Symbol

One the new family symbol has been either retrieved or freshly created and the selection of the existing family instances to modify is successfully completed, the actual modification is trivial; it just requires iterating over the selected instances and modifying the value of their read-write Symbol property.

Here is the complete source code of the external command Execute mainline implementing this:

```python
/// <summary>
/// Create a new table type and
/// modify selected instances.
/// </summary>
[Transaction( TransactionMode.Manual )]
public class CmdTableNewTypeModify : IExternalCommand
{
  const string \_usage\_error = "Please pre-select "
    + "only table elements before running "
    + "this command.";

  /// <summary>
  /// Name of the new table family type.
  /// </summary>
  const string \_type\_name = "1200x750x380mm";

  public Result Execute(
    ExternalCommandData commandData,
    ref string message,
    ElementSet elements )
  {
    UIApplication uiapp = commandData.Application;
    UIDocument uidoc = uiapp.ActiveUIDocument;
    Application app = uiapp.Application;
    Document doc = uidoc.Document;

    TableSelector selector
      = new TableSelector( uidoc );

    IList<FamilyInstance> tables
      = selector.SelectedTables;

    if( null == tables )
    {
      return selector.Return();
    }

    using( Transaction tx = new Transaction( doc ) )
    {
      tx.Start( "Create Type and Modify Instances" );

      // Retrieve existing type or create new

      FamilySymbol symbol
        = Util.FindElementByName( doc,
          typeof( FamilySymbol ), \_type\_name )
            as FamilySymbol
        ?? CreateNewType( tables[0].Symbol );

      foreach( FamilyInstance table in tables )
      {
        Debug.Print( Util.ElementDescription(
          table ) );

        table.Symbol = symbol;
      }

      tx.Commit();
    }
    return Result.Succeeded;
  }
}
```

Please refer to the
[family API topics overview](http://thebuildingcoder.typepad.com/blog/2013/06/key-concepts-of-the-family-editor.html#5) for
a snapshot of the full Visual Studio solution implementing this..