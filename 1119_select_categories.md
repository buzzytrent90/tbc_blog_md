---
ai_optimized: true
author: Jeremy Tammik
complexity_score: 5.6
content_type: code_example
optimization_date: '2025-12-11T11:44:15.339719'
original_url: https://thebuildingcoder.typepad.com/blog/1119_select_categories.html
post_number: '1119'
reading_time_minutes: 9
series: general
slug: select_categories
source_file: 1119_select_categories.htm
tags:
- csharp
- doors
- elements
- filtering
- geometry
- levels
- python
- revit-api
- rooms
- selection
- transactions
- views
- walls
- windows
title: Selecting Visible Categories from a Set of Views
word_count: 1730
---

### Selecting Visible Categories from a Set of Views

Today I look at interactively picking specific categories to export to the simplified 2D BIM editor cloud database from a list of all categories retrieved from a collection of elements displayed in a given set of plan views.

This is the second instalment of implementing the new RoomEditorApp features required for my
[generic 2D simplified BIM editor](http://thebuildingcoder.typepad.com/blog/2014/03/back-from-desert-and-two-happy-events.html#3) Tech
Summit presentation.

Specifically, I now address steps 4 and 5 in the
[tentative workflow](http://thebuildingcoder.typepad.com/blog/2014/03/using-generic-collections-with-filters-and-forms.html):

1. Launch the RoomEditorApp export plan views command.
2. Display a list of plan views in a popup window.
3. Select views to be exported and click OK.
4. Display a list of categories in a popup window.
5. Select categories to be exported and click OK.
6. Store relevant graphical and non-graphical information in a cloud database.
7. Navigate and display simplified model on mobile device.
8. Edit graphical and non-graphical information.
9. Update Revit BIM either interactively or real-time.

The list of categories to choose from should obviously not include all categories present in the Revit project, only those relevant to the elements in the selected views.

Therefore, it makes sense to implement a category collector helper class to determine the categories of interest based on the views selected in step 3.

For each view, we determine the elements of potential interest displayed in it.
From those, we collect all the distinct categories.
In this case, 'distinct categories' is defined as 'categories having different element ids'.

This distinction is achieved by implementing a category equality comparer class that considers two categories the same if their element id is equal.
This comparer class is passed in to the generic dictionary class instance used to collect the categories.
Without the element id comparer, it would contain a large number of duplicate entries.

The dictionary uses the category instances as keys.
This enables us to easily pass them in as selectable items to a checked list box in the .NET category selection form for the interactive user selection step 5.

Consequently, I implement one helper and two new main classes to achieve these steps and update the external command to drive them:

- [CategoryEqualityComparer](#3) – Implement an equality comparer to ensure that categories with the same category id compare equal.
- [CategoryCollector](#4) – Collect categories from elements displayed in a given set of views.
- [FrmSelectCategories](#5) – Interactive category selection form.
- [CmdUploadViews](#6) – Drive the view and category selection.

#### CategoryEqualityComparer

An equality comparer implements the IEqualityComparer interface, requiring the two methods Equals and GetHashCode.

They are straightforward to implement based on the category element id:

```python
  /// <summary>
  /// Categories with the same element id equate to
  /// the same category. Without this, many, many,
  /// many duplicates.
  /// </summary>
  class CategoryEqualityComparer
    : IEqualityComparer<Category>
  {
    public bool Equals( Category x, Category y )
    {
      return x.Id.IntegerValue.Equals(
        y.Id.IntegerValue );
    }

    public int GetHashCode( Category obj )
    {
      return obj.Id.IntegerValue.GetHashCode();
    }
  }
```

#### CategoryCollector

The CategoryCollector collects the distinct categories from all the elements of interest displayed in a given set of views.

The elements of interest are those that we might potentially wish to represent in the simplifier BIM editor cloud database and viewer.

There are an infinite number of ways to define what that might mean, so you will almost certainly have to adapt the detailed decision to your specific needs.

This is related to the question of how to select all model elements or visible 3D elements from a project, to which we already discussed quite a wide range of answers:

- [Valid category and non-empty geometry](http://thebuildingcoder.typepad.com/blog/2009/05/selecting-model-elements.html)
- [Ditto, and eliminate certain categories](http://thebuildingcoder.typepad.com/blog/2009/11/select-model-elements-2.html)
- [Valid category and neither category nor element is hidden](http://thebuildingcoder.typepad.com/blog/2009/11/visible-elements.html)
- [Based on the Category.HasMaterialQuantities property](http://thebuildingcoder.typepad.com/blog/2010/10/selecting-model-elements.html)
- [Based on the elements visible in a default 3D view](http://thebuildingcoder.typepad.com/blog/2010/10/model-elements-revisited.html)
- [Using a custom exporter](http://thebuildingcoder.typepad.com/blog/2013/08/determining-absolutely-all-visible-elements.html)

The last three of these are all very effective.

In this case, I chose to combine two of them, using a filtered element collector based on the view plus checking the Category.HasMaterialQuantities property.

The category collector implements a couple of additional bookkeeping variables to count the number of views, total elements and elements with Category.HasMaterialQuantities set for reporting purposes.

The entire view iteration, element selection, category collection and reporting takes place in the constructor:

```python
  /// <summary>
  /// Collect all categories of all visible
  /// elements in a given set of views.
  /// </summary>
  class CategoryCollector : Dictionary<Category, int>
  {
    /// <summary>
    /// Number of view selected.
    /// </summary>
    int \_nViews;

    /// <summary>
    /// Number of elements in all selected views
    /// including repetitions.
    /// </summary>
    int \_nElements;

    /// <summary>
    /// Number of elements whose category have
    /// material quantities in all selected views
    /// including repetitions.
    /// </summary>
    int \_nElementsWithCategorMaterialQuantities;

    public CategoryCollector( IList<ViewPlan> views )
      : base( new CategoryEqualityComparer() )
    {
      \_nViews = views.Count;
      \_nElements = 0;
      \_nElementsWithCategorMaterialQuantities = 0;

      if( 0 < \_nViews )
      {
        Document doc = views[0].Document;

        FilteredElementCollector a;

        foreach( View v in views )
        {
          a = new FilteredElementCollector( doc, v.Id )
            .WhereElementIsViewIndependent();

          foreach( Element e in a )
          {
            ++\_nElements;

            Category cat = e.Category;

            if( null != cat
              && cat.HasMaterialQuantities )
            {
              ++\_nElementsWithCategorMaterialQuantities;

              if( !ContainsKey( cat ) )
              {
                Add( cat, 0 );
              }
              ++this[cat];
            }
          }
        }
      }
      Debug.Print( "Selected {0} categor{1} from "
        + "{2} view{3} displaying {4} element{5}, "
        + "{6} with HasMaterialQuantities=true",
        Count, Util.PluralSuffixY( Count ),
        \_nViews, Util.PluralSuffix( \_nViews ),
        \_nElements, Util.PluralSuffix( \_nElements ),
        \_nElementsWithCategorMaterialQuantities );
    }
  }
```

I executed this selection process in a very simple model with just three views, Level 0, Level 1 and Site, and pared down the number of categories selected to just five in the end: Curtain Panels, Doors, Furniture, Structural Columns, Walls.

Here are the results for some of the other alternatives I explored before choosing the final filtering approach:

- 14 categories – all elements listed in the given set of views: Cameras, Curtain Panels, Curtain Wall Grids, Curtain Wall Mullions, Doors, Elevations, Furniture, Project Base Point, Room Tags, Rooms, Structural Columns, Survey Point, Views, Walls.
- 13 categories – not view specific, eliminating Room Tags: Cameras, Curtain Panels, Curtain Wall Grids, Curtain Wall Mullions, Doors, Elevations, Furniture, Project Base Point, Rooms, Structural Columns, Survey Point, Views, Walls.
- 12 categories – non-empty bounding box, eliminating Project Base Point and Survey Point: Cameras, Curtain Panels, Curtain Wall Grids, Curtain Wall Mullions, Doors, Elevations, Furniture, Room Tags, Rooms, Structural Columns, Views, Walls.
- 5 categories – Category.HasMaterialQuantities, eliminating all of the above and more: Curtain Panels, Doors, Furniture, Structural Columns, Walls.

#### FrmSelectCategories

Once the categories to present to the user have been pared down to the ones of potential interest, we can display them in a checked list box in a .NET form for the user to make the final interactive selection.

The category selection form FrmSelectCategories is similar to the view selection form
[FrmSelectViews](http://thebuildingcoder.typepad.com/blog/2014/03/using-generic-collections-with-filters-and-forms.html#2) that
we discussed last week, except that we pass the pre-defined list of categories from the category collector discussed above straight into the constructor:

```python
  /// <summary>
  /// Interactive category selection form.
  /// </summary>
  public partial class FrmSelectCategories : Form
  {
    IList<Category> \_categories;

    /// <summary>
    /// Initialise the category selector
    /// with the given list of categories.
    /// </summary>
    /// <param name="categories"></param>
    public FrmSelectCategories(
      IList<Category> categories )
    {
      InitializeComponent();

      \_categories = categories;
    }

    /// <summary>
    /// Initialise the category selector with
    /// the list of categories passed in to
    /// the constructor and check them all.
    /// </summary>
    private void FrmSelectCategories\_Load(
      object sender,
      EventArgs e )
    {
      checkedListBox1.DataSource = \_categories;
      checkedListBox1.DisplayMember = "Name";

      // Set all entries to be initially checked.

      int n = checkedListBox1.Items.Count;

      for( int i = 0; i < n; ++i )
      {
        checkedListBox1.SetItemChecked( i, true );
      }
    }

    /// <summary>
    /// Access the selected categories after the
    /// form has been successfully completed.
    /// </summary>
    public List<Category> GetSelectedCategories()
    {
      return checkedListBox1.CheckedItems
        .Cast<Category>().ToList<Category>();
    }
  }
```

#### CmdUploadViews

The categories retrieved by the category collector are sorted alphabetically before populating the interactive selection form checked list box.

Just like the first form, the second one is properly parented by attaching it to the main Revit application window handle.

Here is the entire updated command implementation:

```csharp
#region Namespaces
using System;
using System.Linq;
using System.Collections.Generic;
using Autodesk.Revit.ApplicationServices;
using Autodesk.Revit.Attributes;
using Autodesk.Revit.DB;
using Autodesk.Revit.UI;
using ComponentManager = Autodesk.Windows.ComponentManager;
using IWin32Window = System.Windows.Forms.IWin32Window;
using DialogResult = System.Windows.Forms.DialogResult;
using System.Diagnostics;
#endregion

namespace RoomEditorApp
{
  [Transaction( TransactionMode.ReadOnly )]
  public class CmdUploadViews : IExternalCommand
  {
    public Result Execute(
      ExternalCommandData commandData,
      ref string message,
      ElementSet elements )
    {
      IWin32Window revit\_window
        = new JtWindowHandle(
          ComponentManager.ApplicationWindow );

      UIApplication uiapp = commandData.Application;
      UIDocument uidoc = uiapp.ActiveUIDocument;
      Application app = uiapp.Application;
      Document doc = uidoc.Document;

      if( null == doc )
      {
        Util.ErrorMsg( "Please run this command in a valid"
          + " Revit project document." );
        return Result.Failed;
      }

      FrmSelectViews form = new FrmSelectViews( doc );

      if( DialogResult.OK == form.ShowDialog(
        revit\_window ) )
      {
        List<ViewPlan> views = form.GetSelectedViews();

        int n = views.Count;

        string caption = string.Format(
          "{0} Plan View{1} Selected",
          n, Util.PluralSuffix( n ) );

        string list = string.Join( ", ",
          views.Select<Element, string>(
            e => e.Name ) );

        Util.InfoMsg2( caption, list );

        List<Category> categories
          = new List<Category>(
            new CategoryCollector( views ).Keys );

        // Sort categories alphabetically by name
        // to display them in selection form.

        categories.Sort(
          delegate( Category c1, Category c2 )
          {
            return string.Compare( c1.Name, c2.Name );
          } );

        FrmSelectCategories form2
          = new FrmSelectCategories( categories );

        if( DialogResult.OK == form2.ShowDialog(
          revit\_window ) )
        {
          categories = form2.GetSelectedCategories();

          n = categories.Count;

          caption = string.Format(
            "{0} Categor{1} Selected",
            n, Util.PluralSuffixY( n ) );

          list = string.Join( ", ",
            categories.Select<Category, string>(
              e => e.Name ) );

          Util.InfoMsg2( caption, list );
        }
      }
      return Result.Succeeded;
    }
  }
}
```

Executing the external command in my simple test model displays the following forms and reports:

Plan view selection form:

![Plan view selection form](img/category_select_1_views.png)

Plan view selection report:

![Plan view selection report](img/category_select_2_view_report.png)

Category selection form:

![Category selection form](img/category_select_3_categories.png)

Category selection report:

![Category selection report](img/category_select_4_category_report.png)

#### Download

For the complete source code, Visual Studio solution and add-in manifest, please refer to the
[RoomEditorApp GitHub repository](https://github.com/jeremytammik/RoomEditorApp).

The version discussed above is stored as
[release 2014.0.2.2](https://github.com/jeremytammik/RoomEditorApp/releases/tag/2014.0.2.2).