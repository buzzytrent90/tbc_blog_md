---
ai_optimized: true
author: Jeremy Tammik
complexity_score: 5.1
content_type: code_example
optimization_date: '2025-12-11T11:44:15.351178'
original_url: https://thebuildingcoder.typepad.com/blog/1125_select_sheets.html
post_number: '1125'
reading_time_minutes: 7
series: views
slug: select_sheets
source_file: 1125_select_sheets.htm
tags:
- csharp
- doors
- elements
- filtering
- levels
- python
- revit-api
- rooms
- selection
- sheets
- views
- walls
- windows
title: Selecting Sheets instead of Views for the Simplified BIM Editor
word_count: 1469
---

### Selecting Sheets instead of Views for the Simplified BIM Editor

Today I look at interactively picking sheets to export, instead of views.

First, however, I must mention the
[list of April Fool's posts](http://adndevblog.typepad.com/autocad/2014/04/feeling-foolish.html) compiled
by
[Stephen Preston](http://adndevblog.typepad.com/autocad/stephen-preston.html),
including my own visionary idea to
[recycle waste as insulation and curtain wall material](http://thebuildingcoder.typepad.com/blog/2014/04/automating-large-scale-waste-recycling-and-sustainability.html) around
the building producing it.
Personally, I like Stephen's own post on the
[Autodesk Toilet Design Suite](http://adndevblog.typepad.com/autocad/2014/03/autodesk-announces-microsuites-1.html) product much better :-)

Back to picking sheets...

This is the third instalment of implementing the new RoomEditorApp features required for my
[generic 2D simplified BIM editor](http://thebuildingcoder.typepad.com/blog/2014/03/back-from-desert-and-two-happy-events.html#3) Tech
Summit presentation, including a change of plan and some additional complexity, as we shall see below.

Inspired by input from interested developers, I opted to base the simplified BIM cloud database and mobile device visualisation on sheets instead of floor plans.

Steps 1, 2 and 3 in the
[previous tentative workflow](http://thebuildingcoder.typepad.com/blog/2014/03/using-generic-collections-with-filters-and-forms.html) refer
to plan views, and actually floor plans, to be specific.

I switch those to select sheets instead.

Additional complexity is thus introduced, since each sheet can contain and display multiple views, and each view will have its own location and scale within the sheet.

It will be interesting to see how to accurately represent the multiple nested transformations required to get from the Revit coordinate system to the views, sheets, cloud database, SVG editor final destination and all the way back again.

#### Tentative Workflow

Meanwhile, here is the updated plan:

1. Launch the RoomEditorApp export sheets command.
2. Display a list of view sheets in a popup window.
3. Select sheets to be exported and click OK.
4. Determine all the floor plan views displayed in the selected sheets.
5. Determine all the categories used in those views.
6. Display a list of those categories in a popup window.
7. Select categories to be exported and click OK.
8. Determine location and scaling of the floor plan views within the selected sheets.
9. Store relevant graphical and non-graphical information in a cloud database.
10. Navigate and display simplified model on mobile device.
11. Edit graphical and non-graphical information.
12. Update Revit BIM either interactively or real-time.

Here and now, I'll discuss the new steps 1-4.

The steps 5, 6 and 7 correspond to the steps 4 and 5 in the previous tentative workflow, were addressed
[selecting visible categories from a set of views](http://thebuildingcoder.typepad.com/blog/2014/03/selecting-visible-categories-from-a-set-of-views.html),
and can easily be adapted to the new workflow.

Here are the new methods and classes required:

- [ElementEqualityComparer](#3) – an equality comparer ensuring that elements with the same element id compare equal.
- [FrmSelectSheets](#4) – interactive sheet view selection form.
- [IsSameOrSubclassOf](#5) – check whether class is subclass of OR equal to the class itself.
- [CmdUploadSheets](#6) – drive the sheet and category selection and list the results.

#### ElementEqualityComparer

After selecting the sheets, I want to determine which floor plan views they contain.

I stuff all the views as keys into a dictionary.

To ensure that the views compare properly with each other, I need to supply a comparison operator that compares their element ids, not the objects themselves.

This is obviously very similar to the
[CategoryEqualityComparer](http://thebuildingcoder.typepad.com/blog/2014/03/selecting-visible-categories-from-a-set-of-views.html#3) introduced
in the last instalment.
The similarity lies in the fact that categories also have an Id property, but are not derived from the Revit Element class, so that property represents a category id, not an element id.

#### FrmSelectSheets

I edited the previous
[FrmSelectViews](http://thebuildingcoder.typepad.com/blog/2014/03/using-generic-collections-with-filters-and-forms.html#2) to
select view sheets instead of floor plan views,
e.g., replacing ViewPlan by ViewSheet, ViewType.FloorPlan by ViewType.DrawingSheet, etc.

The selection result is now obviously a list of sheets, from which the floor plan views need to be extracted.

#### IsSameOrSubclassOf

I initially tried to use the .NET IsSubclassOf method to check for views of type ViewPlan in the sheets, only to notice once again that this method only returns true for true non-equal subclasses.

I therefore defined a new utility method IsSameOrSubclassOf to do what I really want, as suggested by Lasse V. Karlsen in
[stackoverflow.com](http://stackoverflow.com/questions/2742276/in-c-how-do-i-check-if-a-type-is-a-subtype-or-the-type-of-an-object):

```python
  /// <summary>
  /// Return true if the type b is either a
  /// subclass of OR equal to the base class itself.
  /// IsSubclassOf returns false if the two types
  /// are the same. It only returns true for true
  /// non-equal subclasses.
  /// </summary>
  public static bool IsSameOrSubclassOf(
    Type a,
    Type b )
  {
    return a.IsSubclassOf( b ) || a == b;
  }
```

#### CmdUploadSheets

The previous
[CmdUploadViews](http://thebuildingcoder.typepad.com/blog/2014/03/using-generic-collections-with-filters-and-forms.html#3) is
also renamed, and this is obviously the place where the interesting changes need to be applied.

This is still the skeleton of the final command, currently just driving the sheet and category selection and listing the results.

The most interesting new part is the determination of the views displayed within the sheets.

For details on the rest, please refer to the initial
[CmdUploadViews description](http://thebuildingcoder.typepad.com/blog/2014/03/using-generic-collections-with-filters-and-forms.html#3).

Here is the entire implementation of the external command Execute method:

```csharp
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

  // Interactive sheet selection.

  FrmSelectSheets form = new FrmSelectSheets( doc );

  if( DialogResult.OK == form.ShowDialog(
    revit\_window ) )
  {
    List<ViewSheet> sheets
      = form.GetSelectedSheets();

    int n = sheets.Count;

    string caption = string.Format(
      "{0} Sheet{1} Selected",
      n, Util.PluralSuffix( n ) );

    string msg = string.Join( ", ",
      sheets.Select<Element, string>(
        e => e.Name ) ) + ".";

    // Determine all views displayed
    // in the selected sheets.

    Dictionary<View, int> views
      = new Dictionary<View, int>(
        new ElementEqualityComparer() );

    int nFloorPlans = 0;

    foreach( ViewSheet sheet in sheets )
    {
      foreach( View v in sheet.Views )
      {
        if( !views.ContainsKey( v ) )
        {
          if( Util.IsSameOrSubclassOf(
              v.GetType(), typeof( ViewPlan ) )
            && v.CanBePrinted
            && ViewType.FloorPlan == v.ViewType )
          {
            ++nFloorPlans;
          }
          views.Add( v, 0 );
        }
        ++views[v];
      }
    }

    msg += ( 1 == n )
      ? "\nIt contains"
      : "\nThey contain";

    n = views.Count;

    msg += string.Format(
      " {0} view{1} including {2} floor plan{3}: ",
      n, Util.PluralSuffix( n ), nFloorPlans,
      Util.PluralSuffix( nFloorPlans ) );

    msg += string.Join( ", ",
      views.Keys.Select<Element, string>(
        e => e.Name ) ) + ".";

    Util.InfoMsg2( caption, msg );

    // Determine all categories occurring
    // in the views displayed by the sheets.

    List<Category> categories
      = new List<Category>(
        new CategoryCollector( views.Keys ).Keys );

    // Sort categories alphabetically by name
    // to display them in selection form.

    categories.Sort(
      delegate( Category c1, Category c2 )
      {
        return string.Compare( c1.Name, c2.Name );
      } );

    // Interactive category selection.

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

      msg = string.Join( ", ",
        categories.Select<Category, string>(
          e => e.Name ) ) + ".";

      Util.InfoMsg2( caption, msg );
    }
  }
  return Result.Succeeded;
```

I ran the command in a super simple model with the following views and sheets:

![Project browser with views and sheets](img/sheet_select_1.png)

The three sheets are offered for selection in the check box list:

![Selecting sheets](img/sheet_select_2.png)

Simply hitting OK selects all three.
They are listed together with the views and count of floor plans they contain:

```
  3 Sheets Selected: Sheet view of Level 0 and 1,
  3D, Level 0 Duplicate.

  They contain 4 views including 3 floor plans:
  Level 0, Level 1, {3D}, Dependent on Level 0.
```

![Selected sheets and the views they contain](img/sheet_select_3.png)

The three sheets are offered for selection in the check box list:

![Selecting categories](img/sheet_select_4.png)

Simply hitting OK selects all five, and they are listed:

```
  Selected 5 categories from 4 views displaying
  1692 elements, 821 with HasMaterialQuantities=true

  5 Categories Selected: Curtain Panels, Doors,
  Furniture, Structural Columns, Walls.
```

![Selected categories](img/sheet_select_5.png)

#### Download

For the complete source code, Visual Studio solution and add-in manifest, please refer to the
[RoomEditorApp GitHub repository](https://github.com/jeremytammik/RoomEditorApp).

The version discussed above is stored as
[release 2014.0.2.3](https://github.com/jeremytammik/RoomEditorApp/releases/tag/2014.0.2.3).