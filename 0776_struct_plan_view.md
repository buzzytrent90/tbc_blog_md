---
post_number: "0776"
title: "Create Structural Plan View"
slug: "struct_plan_view"
author: "Jeremy Tammik"
tags: ['csharp', 'elements', 'family', 'filtering', 'levels', 'python', 'revit-api', 'schedules', 'transactions', 'views']
source_file: "0776_struct_plan_view.htm"
original_url: "https://thebuildingcoder.typepad.com/blog/0776_struct_plan_view.html"
---

### Create Structural Plan View

Today is the last day of the last week before the
[AEC DevCamp](http://www.cvent.com/events/devcamp-2012/event-summary-56817a3b57614f8eb59ea05fcd59bc32.aspx)
in Waltham.
Here are the complete
[session list and descriptions](https://custom.cvent.com/FDBB345248B94F40BFFFCEF2FBE054E4/files/645f182b028d480281ebdda12bae6576.pdf).
I have to submit my material today.
Before doing so, one last post on a pure API topic.

I mentioned that Revit 2013 introduced a new
[view API](http://thebuildingcoder.typepad.com/blog/2012/03/revit-2013-and-its-api.html#2) and
now includes the possibility to create
[schedules](http://thebuildingcoder.typepad.com/blog/2012/04/developer-center-and-sdk-update.html#22) and
[section views](http://thebuildingcoder.typepad.com/blog/2012/05/change-section-view-type-and-hide-cut-line.html).

Here is a question that came up regarding the new view creation and Onebox and leads to a couple of interesting migration points:

**Question:** I have a plug-in developed for Revit Structure which is currently working on the versions 2009 to 2012.

Part of the plug-in involves creation of levels and a corresponding floor plan for each.

I now ran into the following problem in Revit 2013 Onebox: after creating all Levels with their respective floor plans, they are displayed in the wrong subnode of the Views tree in the Project Browser.
The default Level 'Level 1' is shown under the 'Structural Plans' node, and the newly created floor plans are located under 'Floor Plans' instead.

In spite of the unexpected result in the GUI, all levels are actually created and all floor plans correctly assigned.

I also noticed that the NewViewPlan method to create floor plans is marked obsolete in the 2013 API, so I tried using the new ViewPlan.Create method instead.
The result is the same using both methods.

Here is the code using both of these methods:
```python
/// <summary>
/// Create FloorPlan for Level using NewViewPlan
/// </summary>
public Level CreateLevel2012(
  Document doc,
  double elevation,
  string name )
{
  Autodesk.Revit.Creation.Document createdoc
    = doc.Create;

  Level level = createdoc.NewLevel( elevation );
  level.Name = name;
  ElementId nid = level.Id;

  createdoc.NewViewPlan( name, level,
    ViewPlanType.FloorPlan );

  return level;
}

/// <summary>
/// Create FloorPlan for Level using ViewPlan.Create
/// </summary>
public Level CreateLevel2013(
  Document doc,
  double elevation,
  string name )
{
  Level level = doc.Create.NewLevel( elevation );
  level.Name = name;
  ElementId nid = level.Id;

  IEnumerable<ViewFamilyType> viewFamilyTypes
    = from elem in new FilteredElementCollector( doc )
        .OfClass( typeof( ViewFamilyType ) )
      let type = elem as ViewFamilyType
      where type.ViewFamily == ViewFamily.FloorPlan
      select type;

  ViewPlan floorPlan = ViewPlan.Create( doc,
    viewFamilyTypes.First().Id, nid );

  return level;
}
```

**Answer:** I created a sample application to test the structural plan creation in Revit 2013 Onebox.
It initially displayed the newly created floor plans under the 'Floor Plan' node, just as you describe.
I wondered whether the behaviour might depend on whether you run this in a structural or architectural project, how the template file is set up, and what discipline is assigned to the view, but that does not seem to make any difference, as I soon found out.

I started up a new structural project in Revit Onebox and ran the command there, and note that the new level also appears in the Floor Plans node.

I next used RevitLookup to examine the default views listed in the Structural Plans node in more detail, using Add-Ins > Revit Lookup > Snoop DB... > ViewPlan > Level 1 70032.

One thing I immediately note is that the ViewFamilyType of the default Level 1 in the structural project is Structural Plan 53969, and not 'Floor Plan' like the one you are referencing in your code:

![Structural plan view family type](img/view_structural_plan_object_type.png)

I therefore modified the creation code to use ViewFamily.StructuralPlan instead of ViewFamily.FloorPlan.
There is no need to set the view discipline, since that happens anyway automatically.

With that in place, the newly created views now show up in the right place in the project browser:

![Structural plan in project browser](img/view_structural_plan.png)

Here is the complete code that I ended up with:
```csharp
  UIApplication uiapp = commandData.Application;
  UIDocument uidoc = uiapp.ActiveUIDocument;
  Document doc = uidoc.Document;

  using( Transaction tx = new Transaction( doc ) )
  {
    tx.Start( "Create Floor Plan" );

    Autodesk.Revit.Creation.Document createdoc
      = doc.Create;

    string name = "Findus";

    // Create Level

    double elevation = 10;
    Level level = createdoc.NewLevel( elevation );
    level.Name = name;
    ElementId nid = level.Id;

    bool use\_obsolete\_code = false;

    if( use\_obsolete\_code )
    {
      // Create FloorPlan for the
      // Level using NewViewPlan

      createdoc.NewViewPlan( name, level,
        ViewPlanType.FloorPlan );
    }
    else
    {
      // Create FloorPlan for the
      // Level using ViewPlan.Create

      ViewFamilyType vft
        = new FilteredElementCollector( doc )
          .OfClass( typeof( ViewFamilyType ) )
          .Cast<ViewFamilyType>()
          .FirstOrDefault<ViewFamilyType>( x =>
            ViewFamily.StructuralPlan == x.ViewFamily );

      ViewPlan floorPlan = ViewPlan.Create(
        doc, vft.Id, nid );

      //floorPlan.Discipline
      //  = ViewDiscipline.Structural;
    }
    tx.Commit();
  }
  return Result.Succeeded;
```

I hope you appreciate that you have a lot more control over the view creation using the new method rather than the old one.
There is a heaps more functionality in the new Revit 2013 view API that we have not touched on yet...

Here is
[CreateStructuralPlan.zip](zip/CreateStructuralPlan.zip)
including the entire Visual Studio solution, source code and add-in manifest.