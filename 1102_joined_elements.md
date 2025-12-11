---
ai_optimized: true
author: Jeremy Tammik
complexity_score: 3.4
content_type: qa
optimization_date: '2025-12-11T11:44:15.308556'
original_url: https://thebuildingcoder.typepad.com/blog/1102_joined_elements.html
post_number: '1102'
reading_time_minutes: 3
series: elements
slug: joined_elements
source_file: 1102_joined_elements.htm
tags:
- csharp
- elements
- family
- geometry
- references
- revit-api
- selection
- walls
- windows
title: Getting Two Different Kinds of Joined Elements
word_count: 553
---

### Getting Two Different Kinds of Joined Elements

Revit loves creating and maintaining relationships, e.g. by joining elements and their geometry.

Here are a couple of element joining topics that we touched on in the past:

- [AutoJoinElements](http://thebuildingcoder.typepad.com/blog/2010/05/autojoinelements.html)
- [Joined Beam Geometry Access](http://thebuildingcoder.typepad.com/blog/2011/01/joined-beam-geometry-access.html)
- [Wall Joins and Geometry](http://thebuildingcoder.typepad.com/blog/2011/08/wall-joins-and-geometry.html)
- [Move Duct Join Add-In with Video and GitHub Support](http://thebuildingcoder.typepad.com/blog/2013/10/move-duct-join-add-in-with-video-and-github-support.html)

The user interface also includes functionality related to this and explicitly labelled Join Geometry:

![Join Geometry in the Revit user interface](img/JoinedElements.jpeg)

The same functionality uses slightly different nomenclature in the API, though, which might prove a bit confusing.

Here is a recent developer query to prove the point, with a nice clear answer by my colleague Akira Kudo:

**Question:** I am trying to use the JoinGeometryUtils.GetJoinedElements method with little success.

When passed in an existing element that has another element joined to it, the method still returns an empty collection.

However, if I right click one of the elements in the user interface and then 'Select Joined Elements', both elements are selected.

**Answer:** The GetJoinedElements method only returns elements that share a `GeomJoin`.

A GeomJoin is a pair-wise cut of geometry from one element to another.

To try this out, you can use the attached sample code and Revit model in
[JoinedElements1.zip](zip/JoinedElements1.zip),
implementing the following external command:

```csharp
  public Result Execute(
    ExternalCommandData commandData,
    ref string message,
    ElementSet elements )
  {
    UIApplication uiapp = commandData.Application;
    UIDocument uidoc = uiapp.ActiveUIDocument;
    Document doc = uidoc.Document;

    Reference ref1 = uidoc.Selection.PickObject(
      ObjectType.Element, "pick an element" );

    Element elem = doc.GetElement( ref1 );

    ICollection<ElementId> the\_list\_of\_the\_joined
      = JoinGeometryUtils.GetJoinedElements(
        doc, elem );

    System.Windows.Forms.MessageBox.Show(
      the\_list\_of\_the\_joined.Count.ToString() );

    uidoc.Selection.Elements.Clear();

    foreach( ElementId id in the\_list\_of\_the\_joined )
    {
      uidoc.Selection.Elements.Add(
        doc.GetElement( id ) );
    }
    return Result.Succeeded;
  }
```

As said, this is completely different from 'Select Joined Elements' in the user interface.

That UI functionality is available through the API via the LocationCurve.get\_ElementsAtJoin method.
It determines elements joined to the current one at the end of its location curve and also enables changing their joining behaviour and order.

You can try this out as well using the attached sample code and Revit model in
[JoinedElements2.zip](zip/JoinedElements2.zip),
implementing this different external command:

Please also find the sample project in the JoinedElements2.zip.

```csharp
  public Result Execute(
    ExternalCommandData commandData,
    ref string message,
    ElementSet elements )
  {
    UIApplication uiapp = commandData.Application;
    UIDocument uidoc = uiapp.ActiveUIDocument;
    Document doc = uidoc.Document;

    Reference ref1 = uidoc.Selection.PickObject(
      ObjectType.Element, "pick an element" );

    Element elem = doc.GetElement( ref1 );
    FamilyInstance fi = elem as FamilyInstance;
    LocationCurve lc = fi.Location as LocationCurve;

    uidoc.Selection.Elements.Clear();
    ElementArray startarray = lc
      .get\_ElementsAtJoin( 0 ); // start

    foreach( Element elem1 in startarray )
    {
      uidoc.Selection.Elements.Add( elem1 );
    }

    ElementArray endarray = lc
      .get\_ElementsAtJoin( 1 ); // end

    foreach( Element elem2 in endarray )
    {
      uidoc.Selection.Elements.Add( elem2 );
    }
    return Result.Succeeded;
  }
```

You should also check out the code provided by the Revit SDK sample ProximityDetection\_WallJoinControl in the GeometryAPI subfolder.