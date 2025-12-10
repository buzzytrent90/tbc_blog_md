---
post_number: "1243"
title: "Determining Intersecting Elements and Continued Futureproofing"
slug: "future_proof"
author: "Jeremy Tammik"
tags: ['csharp', 'elements', 'family', 'filtering', 'geometry', 'parameters', 'python', 'references', 'revit-api', 'rooms', 'selection', 'sheets', 'transactions', 'views', 'walls', 'windows']
source_file: "1243_future_proof.htm"
original_url: "https://thebuildingcoder.typepad.com/blog/1243_future_proof.html"
---

### Determining Intersecting Elements and Continued Futureproofing

I continued cleaning up the obsolete API usage in The Building Coder samples, and took a look at a new blog post by Joe Ye:

- [Eliminating more obsolete API usage](#2)
- [Determining all family instances intersecting an element](#3)
- [Updated element selection utility methods](#4)
- [Intersection results and download](#5)
- [Explanation: joined geometry removes intersection](#6)

#### Eliminating More Obsolete API Usage

I continue in my battle against the obsolete API usage warnings after the
[initial migration to Revit 2015](http://thebuildingcoder.typepad.com/blog/2014/04/compiling-the-revit-2015-sdk-and-migrating-bc-samples.html#5) and
subsequently finally
[getting started migrating deprecated API](http://thebuildingcoder.typepad.com/blog/2014/11/migrating-deprecated-api-and-2d-boolean-operations.html#3).

I now removed the following ten warnings, all of them with the number CS0618, leaving
[35 remaining](zip/bc_migr_2015_e.txt) to
be fixed:

- CmdCollectorPerformance.cs: Autodesk.Revit.DB.Element.get\_Parameter(string) is obsolete, as more than one parameter can have the same name on a given element.
- CmdCoordsOfViewOnSheet.cs: Autodesk.Revit.DB.ViewSheet.Views is obsolete. Use GetAllPlacedViews instead.
- CmdRollingOffset.cs: Autodesk.Revit.Creation.Document.NewPipe is obsolete. Please use Pipe.Create instead.
- CmdNewDuctSystem.cs: Autodesk.Revit.UI.Selection.Selection.Elements is obsolete. Use GetElementIds instead.
- CmdTransformedCoords.cs: Autodesk.Revit.UI.Selection.Selection.Elements is obsolete. Use GetElementIds instead.
- CmdUnrotateNorth.cs: Autodesk.Revit.UI.Selection.Selection.Elements is obsolete. Use GetElementIds instead.
- CmdWindowHandle.cs: Autodesk.Revit.UI.Selection.SelElementSet is obsolete. Use GetElementIds instead.
- CmdWindowHandle.cs: Autodesk.Revit.UI.Selection.Selection.Elements is obsolete. Use GetElementIds instead.
- CmdWindowHandle.cs: Autodesk.Revit.UI.Selection.SelElementSet is obsolete. Use GetElementIds instead.
- CmdWindowHandle.cs: Autodesk.Revit.UI.Selection.Selection.Elements is obsolete. Use GetElementIds instead.

As always, the most up to date version of The Building Coder samples is provided in
[its GitHub repository](https://github.com/jeremytammik/the_building_coder_samples),
and the version described here is
[release 2015.0.116.4](https://github.com/jeremytammik/the_building_coder_samples/releases/tag/2015.0.116.4).

#### Determining all Family Instances Intersecting an Element

My colleague
[Joe Ye](http://adndevblog.typepad.com/aec/joe-ye.html) left
Autodesk and is now independently active with [gisbim.com](http://www.glsbim.com) or *Mount of Olives*.

He is still actively supporting and blogging about Revit programming, though, partially as a consultant for Autodesk, e.g. in this explanation on how to
[how to retrieve all columns intersecting a wall](http://blog.csdn.net/joexiongjin/article/details/41090861).

Of course, this can be solved addressing the more general issue of retrieving all family instances of a specific type intersecting a given BIM element.

**Question:** How can I find all columns intersecting a given wall?

**Answer:** The Revit API does not provide direct access to query the relationship between the pillars and wall.

However, you can easily obtain it indirectly through the API using an intersecting element filter, e.g. one of the two classes ElementIntersectsElementFilter or ElementIntersectsSolidFilter.
The latter requires a solid, the former takes a BIM element and uses its existing solid geometry.
For more information on these, please refer to the Revit API help file RevitAPI.chm.

Here is Joe's sample code demonstrating this:

```csharp
  // Find intersections between family instances and a selected element

  Reference Reference = uidoc.Selection.PickObject(
    ObjectType.Element, "Select element that will "
    + "be checked for intersection with all family "
    + "instances" );

  Element e = doc.GetElement( reference );

  GeometryElement geomElement = e.get\_Geometry(
    new Options() );

  Solid solid = null;
  foreach( GeometryObject geomObj in geomElement )
  {
    solid = geomObj as Solid;
    if( solid = !null ) break;
  }

  FilteredElementCollector collector
    = new FilteredElementCollector( doc )
      .OfClass( typeof( FamilyInstance ) )
      .WherePasses( new ElementIntersectsSolidFilter(
        solid ) );

  TaskDialog.Show( "Revit", collector.Count() +
    "Family instances intersect with selected element ("
    + element.Category.Name + "ID:" + element.Id + ")" );
```

I could not resist cleaning this up a bit and adding some LINQ and other generic and functional twiddles to achieve the following enhancements:

- Foolproof the solid selection by skipping null and empty solids.
- Improve the result message and add element information.
- List the element ids of all the intersecting instances.

Here is the resulting implementation:

```csharp
/// <summary>
/// Retrieve all family instances intersecting a
/// given BIM element, e.g. all columns
/// intersecting a wall.
/// </summary>
void GetInstancesIntersectingElement( Element e )
{
  Document doc = e.Document;

  Solid solid = e.get\_Geometry( new Options() )
    .OfType<Solid>()
    .Where<Solid>( s => null != s && !s.Edges.IsEmpty )
    .FirstOrDefault();

  FilteredElementCollector intersectingInstances
    = new FilteredElementCollector( doc )
      .OfClass( typeof( FamilyInstance ) )
      .WherePasses( new ElementIntersectsSolidFilter( solid ) );

  int n = intersectingInstances.Count<Element>();

  string result = string.Format(
    "{0} family instance{1} intersect{2} the "
    + "selected element {3}{4}",
    n, Util.PluralSuffix( n ),
    ( 1 == n ? "s" : "" ),
    Util.ElementDescription( e ),
    Util.DotOrColon( n ) );

  string id\_list = 0 == n
    ? string.Empty
    : string.Join( ", ",
        intersectingInstances
          .Select<Element, string>(
            x => x.Id.IntegerValue.ToString() ) )
      + ".";

  Util.InfoMsg2( result, id\_list );
}
```

After some hesitation, I even took the plunge and decided to actually test this before pushing to GitHub.

I modified the CmdCollectorPerformance external command temporarily to test this method, changing its transaction mode from automatic to read-only and adding these lines of code:

```csharp
  Element wall = Util.SelectSingleElementOfType(
    uidoc, typeof( Wall ), "a wall", true );

  GetInstancesIntersectingElement( wall );
```

That led to some fixes in the utility class selection methods, a few more obsolete API usage removals, plus the discovery that the HasRequestedType method probably never previously worked as intended.

#### Updated Element Selection Utility Methods

Here is a long overdue update to The Building Coder samples element selection utility methods:

```python
  public static Element SelectSingleElement(
    UIDocument uidoc,
    string description )
  {
    if( ViewType.Internal == uidoc.ActiveView.ViewType )
    {
      TaskDialog.Show( "Error",
        "Cannot pick element in this view: "
        + uidoc.ActiveView.Name );

      return null;
    }

    try
    {
      Reference r = uidoc.Selection.PickObject(
        ObjectType.Element,
        "Please select " + description );

      return uidoc.Document.GetElement( r ); // 2012
    }
    catch( Autodesk.Revit.Exceptions.OperationCanceledException )
    {
      return null;
    }
  }

  public static Element GetSingleSelectedElement(
    UIDocument uidoc )
  {
    ICollection<ElementId> ids
      = uidoc.Selection.GetElementIds();

    Element e = null;

    if( 1 == ids.Count )
    {
      foreach( ElementId id in ids )
      {
        e = uidoc.Document.GetElement( id );
      }
    }
    return e;
  }

  static bool HasRequestedType(
    Element e,
    Type t,
    bool acceptDerivedClass )
  {
    bool rc = null != e;

    if( rc )
    {
      Type t2 = e.GetType();

      rc = t2.Equals( t );

      if( !rc && acceptDerivedClass )
      {
        rc = t2.IsSubclassOf( t );
      }
    }
    return rc;
  }

  public static Element SelectSingleElementOfType(
    UIDocument uidoc,
    Type t,
    string description,
    bool acceptDerivedClass )
  {
    Element e = GetSingleSelectedElement( uidoc );

    if( !HasRequestedType( e, t, acceptDerivedClass ) )
    {
      e = Util.SelectSingleElement(
        uidoc, description );
    }
    return HasRequestedType( e, t, acceptDerivedClass )
      ? e
      : null;
  }

  /// <summary>
  /// Retrieve all pre-selected elements of the specified type,
  /// if any elements at all have been pre-selected. If not,
  /// retrieve all elements of specified type in the database.
  /// </summary>
  /// <param name="a">Return value container</param>
  /// <param name="uidoc">Active document</param>
  /// <param name="t">Specific type</param>
  /// <returns>True if some elements were retrieved</returns>
  public static bool GetSelectedElementsOrAll(
    List<Element> a,
    UIDocument uidoc,
    Type t )
  {
    Document doc = uidoc.Document;

    ICollection<ElementId> ids
      = uidoc.Selection.GetElementIds();

    if( 0 < ids.Count )
    {
      a.AddRange( ids
        .Select<ElementId,Element>(
          id => doc.GetElement( id ) )
        .Where<Element>(
          e => t.IsInstanceOfType( e ) ) );
    }
    else
    {
      a.AddRange( new FilteredElementCollector( doc )
        .OfClass( t ) );
    }
    return 0 < a.Count;
  }
```

With the new selection code in place, the project now produces
[32 obsolete API warning messages](zip/bc_migr_2015_e.txt).

#### Intersection Results and Download

Back to the intersection test.

The Revit SDK sample FindReferencesByDirection/FindColumns includes a suitable model for the intersection test, FindColumns-Basic.rvt:

![FindColumns-Basic.rvt sample model](img/columns_intersecting_wall_sample.png)

In the original implementation of this sample, described in
[AVF displays intersections](http://thebuildingcoder.typepad.com/blog/2011/12/using-avf-to-display-intersections-and-highlight-rooms.html) and
[FindColumns Using Geometry Creation and Booleans](http://thebuildingcoder.typepad.com/blog/2011/12/using-avf-to-display-intersections-and-highlight-rooms.html#2),
the resulting intersections are graphically highlighted using the Analysis Visualisation Framework for easy interactive comparison.

![Wall-column intersections highlighted using AVF](img/wall_column_intersections_avf.png)

When picking the straight wall, the following result is reported, more or less as expected:

![Columns intersecting straight wall](img/columns_intersecting_wall_1.png)

Actually, looking at it more closely, we would probably expect more than just two intersections.

The curved wall is even worse and reports zero intersecting instances.

I added code to compare the result of the solid intersection filter with the element one:

```csharp
  Solid solid = e.get\_Geometry( new Options() )
    .OfType<Solid>()
    .Where<Solid>( s => null != s && !s.Edges.IsEmpty )
    .FirstOrDefault();

  FilteredElementCollector intersectingInstances
    = new FilteredElementCollector( doc )
      .OfClass( typeof( FamilyInstance ) )
      .WherePasses( new ElementIntersectsSolidFilter(
        solid ) );

  int n1 = intersectingInstances.Count<Element>();

  intersectingInstances
    = new FilteredElementCollector( doc )
      .OfClass( typeof( FamilyInstance ) )
      .WherePasses( new ElementIntersectsElementFilter(
        e ) );

  int n = intersectingInstances.Count<Element>();

  Debug.Assert( n.Equals( n1 ),
    "expected solid intersection to equal element intersection" );
```

The results are identical for both filters.

I'll keep you posted when I find a solution for this problem.

As always, the updated version of The Building Coder samples is available from
[its GitHub repository](https://github.com/jeremytammik/the_building_coder_samples),
and the version including the GetInstancesIntersectingElement method and updated element selection is
[release 2015.0.116.5](https://github.com/jeremytammik/the_building_coder_samples/releases/tag/2015.0.116.5).

#### Explanation: Joined Geometry Removes Intersection

I received an immediate and satisfying answer to my question above from Scott Conover, who explains:

These filters are working as designed.

The concrete columns intersecting the wall are joined to the wall. Thus their geometry gets automatically adjusted to not trigger an item in the interference checking tool that is the basis for the ElementIntersectsElementFilter implementation, and the solids are not considered overlapping.

If you place or move a steel column instance to be interfering with the wall instead of the concrete ones, it will work as you expect.

I covered this in my geometry class at Autodesk University 2012, cf. the following quote from the
[handout document](http://thebuildingcoder.typepad.com/files/cp4011_conover.pdf):

> The element filters:
>
> - ElementIntersectsElementFilter
> - ElementIntersectsSolidFilter
>
> pass elements whose actual 3D geometry intersects the 3D geometry of the target object.
>
> With ElementIntersectsElementFilter, the target object is another element. The intersection is determined with the same logic used by Revit to determine if an interference exists during generation of an Interference Report. (This means that some combinations of elements will never pass this filter, such as concrete members which are automatically joined at their intersections). Also, elements which have no solid geometry, such as Rebar, will not pass this filter.
>
> With ElementIntersectsSolidFilter, the target object is any solid. This solid could have been obtained from an existing element, created from scratch using the routines in GeometryCreationUtilities, or the result of a secondary operation such as a Boolean operation. Similar to the ElementIntersectsElementFilter, this filter will not pass elements which lack solid geometry.
>
> Both filters can be inverted to match elements outside the target object volume.

Thank you, Scott, for the clarification!

**Addendum:** Joe Ye adds:

Yes, Mount of Olives is the right translation, and the meaning of Gan Lan Shan (橄榄山) is indeed the mount of olives.

I use the term GLS Soft as the unique common company name for both Chinese and English.

I am hiring employees. It is hard to find full time Revit developers. It seems I have to educate some green hands to become full time employees.

Regarding finding the walls around a column, here is an alternative approach:

As you say, the ElementIntersectsElementFilter does not find the joined column when both the wall and the column are concrete and they have therefore been automatically combined.

In this case, it is recommended to use the ElementIntersectsSolidFilter instead, passing in the column's original geometry solid.

See you soon in Las Vegas!