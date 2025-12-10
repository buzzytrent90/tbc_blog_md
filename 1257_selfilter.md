---
post_number: "1257"
title: "SelFilter, a Powerful Generic Selection Filter Utility"
slug: "selfilter"
author: "Jeremy Tammik"
tags: ['csharp', 'elements', 'filtering', 'geometry', 'parameters', 'references', 'revit-api', 'selection', 'transactions', 'views', 'walls']
source_file: "1257_selfilter.htm"
original_url: "https://thebuildingcoder.typepad.com/blog/1257_selfilter.html"
---

### SelFilter, a Powerful Generic Selection Filter Utility

I recently discussed the minimalist generic selection filter implementation
[JtElementsOfClassSelectionFilter](http://thebuildingcoder.typepad.com/blog/2014/11/selection-filters-adjacency-and-the-good-universe.html#2) that
I added to
[The Building Coder samples](https://github.com/jeremytammik/the_building_coder_samples).

Alexander Buschmann of
[IDAT GmbH](http://www.idat.de) responded to that and says:

I saw your generic SelectionFilter class and this reminded me of a little pet project I've done in my spare time that I thought your readers might be interested in.

Two things always bothered me about the ISelectionFilter:

- You have to create a class to use it.
- Though most often you just want to select Elements that match certain criteria, you can't use the already existing ElementFilters instead. It just feels wrong to create my own class to do something that a Revit class can do for me – and most probably it can do it even better than I could do it myself.

I therefore implemented a class **`GetFilter`** that provides fast and easy access to the (probably?) most common selection filtering scenarios:

It supports filtering for one or more classes, for one or more ElementIds, filtering using ElementFilters, some example ReferenceFilterings (for PlanarFaces and for normal vectors of Faces), some methods to filter with delegates or lambdas and some logical combinators, i.e. "and", "or" and "not".

For the logical combinators there are extension methods for ISelectionFilter defined, so every ISelectionFilter can be combined using these.

Using this should be quite easy – here is an
[example command and a Revit project to run it in](zip/SelFilters.zip).

Here are some usage examples:

- SelFilter.GetElementFilter<Wall>() – returns a filter for Walls.
- SelFilter.GetElementFilter(typeof(Wall), typeof(Floor), typeof(Pipe)) – returns a filter for Walls, Floors and Pipes.
- SelFilter.GetElementFilter (elementFilter) – return a filter using an Autodesk.Revit.DB.ElementFilter.
- SelFilter.GetElementFilter(idList) – returns a filter that will let only elements in the idList pass; this might be useful to select faces or edges on some preselected elements.
- filter1.And(filter2) – returns a filter that will check both filter1 and filter2.

Further information:

- The GetFilter class itself is a static class with static methods that return an interface of an instance of the actual (private) filter classes.
- The actual Filter classes are private to reduce namespace cluttering – it's mostly a matter of taste.
- GetElementFilter methods return an IElementSelectionFilter which does the filtering using the AllowElement method; those classes just return true for the AllowReference method.
- GetReferenceFilter, GetPlanarFaceFilter and GetFaceNormalFilter return an IReferenceSelectionFilter that performs the filtering using the AllowReference method; these classes just return true for the AllowElement method.
- As a consequence, the combination of IElementSelectionFilters with IReferenceSelectionFilters is only meaningful using "and". If "or" is used, everything will pass the combined filter.
- The "and" and "or" filter methods return an ILogicalCombinationFilter that combines two or more filters. This interface exposes a property "ExecuteAll" – if it is set to "true", all the filters are called, even when the result already is fixed. This might or might not be helpful for debugging purposes.
- The "not" filter is just an inversion of any other filter and directly returns an "ISelectionFilter".
- The GetFilter method creates a filter class for delegates, method groups or lambdas. This gives the option to create arbitrary and complex filters on the fly without creating classes for them.

I think the mightiest of the methods is the ElementFilter method – all different types of ElementFilter can be used, including CategoryFilters, IntersectionFilters, ExtensibleStorageFilters, ParameterFilters, etc.

If the power of ElementFilters is not needed, the Class and ElementId filters give easy access to simpler filtering, and if more power is needed, this can easily be done with lambdas or delegates.

There are lots of comments in the code, though it is really very easy and straightforward.
The great thing about this class is not something complicated, just having everything needed in one single place.

The
[zip file SelFilters.zip](zip/SelFilters.zip) contains
the SelFilter class, the interface files, an example external command using some of the SelFilter functionality, an add-in manifest file for the example command and a small Revit project to test it in.

Hopefully, you and your readers find this class helpful.

Thank you very much, Alexander, for sharing this!

For the sake of completeness, and especially for the convenience of all those search engines out there, here is the source code for the sample external command mainline Execute method (to see the truncated lines in full, view source or copy and paste to a text editor):

```csharp
//
// Copyright (c) 2014 Alexander Buschmann
//
// Permission is hereby granted, free of charge, to any person obtaining a copy
// of this software and associated documentation files (the "Software"), to
// deal in the Software without restriction, including without limitation the
// rights to use, copy, modify, merge, publish, distribute, sublicense, and/or
// sell copies of the Software, and to permit persons to whom the Software is
// furnished to do so, subject to the following conditions:
//
// The above copyright notice and this permission notice shall be included in
// all copies or substantial portions of the Software.
//
// THE SOFTWARE IS PROVIDED "AS IS", WITHOUT WARRANTY OF ANY KIND, EXPRESS OR
// IMPLIED, INCLUDING BUT NOT LIMITED TO THE WARRANTIES OF MERCHANTABILITY,
// FITNESS FOR A PARTICULAR PURPOSE AND NONINFRINGEMENT. IN NO EVENT SHALL THE
// AUTHORS OR COPYRIGHT HOLDERS BE LIABLE FOR ANY CLAIM, DAMAGES OR OTHER
// LIABILITY, WHETHER IN AN ACTION OF CONTRACT, TORT OR OTHERWISE, ARISING
// FROM, OUT OF OR IN CONNECTION WITH THE SOFTWARE OR THE USE OR OTHER
// DEALINGS IN THE SOFTWARE.
//

using System.Collections.Generic;
using System.Collections.ObjectModel;
using Autodesk.Revit.Attributes;
using Autodesk.Revit.DB;
using Autodesk.Revit.UI;
using Autodesk.Revit.UI.Selection;

namespace RevitSelectionFilters
{
  [Transaction( TransactionMode.ReadOnly )]
  public class SelectionExamples : IExternalCommand
  {
    #region Implementation of IExternalCommand

    public Result Execute(
      ExternalCommandData commandData,
      ref string message,
      ElementSet elements )
    {
      UIDocument uiDoc = commandData.Application.ActiveUIDocument;
      Document doc = uiDoc.Document;
      Selection sel = uiDoc.Selection;

      try
      {
        //
        // Select Wall or Floor
        //
        Reference pickedReference = sel.PickObject(
          ObjectType.Element,
          SelFilter.GetElementFilter( typeof( Wall ), typeof( Floor ) ),
          "Select Wall or Floor" );
        if( pickedReference == null ) return Result.Failed;
        Element firstElement = doc.GetElement( pickedReference );

        TaskDialog.Show(
          "Result",
          "First Selection: " + firstElement.Category.Name + ": " + firstElement.Name + " (" +
          firstElement.Id + ")" );

        //
        // Select anything intersecting:
        //
        ElementFilter filter = new ElementIntersectsElementFilter( firstElement );
        ElementFilter notTheFirst = new ExclusionFilter( new Collection<ElementId> { firstElement.Id } );
        ISelectionFilter intersectionFilter =
          SelFilter.GetElementFilter( filter ).And( SelFilter.GetElementFilter( notTheFirst ) );
        pickedReference = sel.PickObject(
          ObjectType.Element, intersectionFilter,
          "Select anything intersecting the first picked Element" );
        if( pickedReference == null ) return Result.Failed;
        Element secondElement = doc.GetElement( pickedReference );
        TaskDialog.Show(
          "Result",
          "Second Selection: " + secondElement.Category.Name + ": " + secondElement.Name + " (" +
          secondElement.Id + ")" );

        //
        // Select colums or beams or foundations within 20 feet of the second element in any direction,
        // but not if they do intersect with the first element
        //
        ICollection<BuiltInCategory> categories = new[] {
          BuiltInCategory.OST\_StructuralColumns,
          BuiltInCategory.OST\_Columns,
          BuiltInCategory.OST\_StructuralFraming,
          BuiltInCategory.OST\_StructuralFoundation,
        };
        ElementFilter catFilter = new ElementMulticategoryFilter( categories );
        BoundingBoxXYZ box = secondElement.get\_BoundingBox( null );
        XYZ vector = new XYZ( 20, 20, 20 );
        Outline outline = new Outline( box.Min - vector, box.Max + vector );
        ElementFilter boxFilter = new BoundingBoxIntersectsFilter( outline );
        ISelectionFilter selectionFilter = intersectionFilter.Not().And(
          SelFilter.GetElementFilter( catFilter ),
          SelFilter.GetElementFilter( boxFilter ) );
        pickedReference = sel.PickObject(
          ObjectType.Element, selectionFilter,
          "Select Column, Beam, Foundation within 20 feet of second element, not intersecting first element" );
        if( pickedReference == null ) return Result.Failed;
        Element thirdElement = doc.GetElement( pickedReference );
        TaskDialog.Show(
          "Result",
          "Third Selection: " + thirdElement.Category.Name + ": " + thirdElement.Name + " (" +
          thirdElement.Id + ")" );

        //
        // Select Face of thirdElement with normal parallel to BasisZ
        //
        selectionFilter = SelFilter.GetReferenceFilter( ( reference, xyz ) =>
        {
          if( reference.ElementId != thirdElement.Id ) return false;
          Face face =
            thirdElement.GetGeometryObjectFromReference( reference ) as Face;
          if( face == null ) return false;
          XYZ normal = face.ComputeNormal( UV.Zero );
          XYZ crossProduct = normal.CrossProduct( XYZ.BasisZ );
          return ( crossProduct.IsAlmostEqualTo( XYZ.Zero ) );
        } );
        pickedReference = sel.PickObject(
          ObjectType.Face, selectionFilter,
          "Select Face of third element with normal parallel to z-Axis" );
        Face pickedFace = thirdElement.GetGeometryObjectFromReference( pickedReference ) as Face;
        XYZ pickedNormal = pickedFace.ComputeNormal( UV.Zero );
        TaskDialog.Show(
          "Result",
          "Fourth Selection: " + pickedFace.GetType().Name + " (" +
          pickedReference.ConvertToStableRepresentation( doc ) + ")\n" +
          "Normal: (" + pickedNormal.X.ToString( "0.###" ) + " / " + pickedNormal.Y.ToString( "0.###" ) + " / " +
          pickedNormal.Z.ToString( "0.###" ) + ")" );

        //
        // Now select any Face with an X oder Y-Normal from any previously selected Element
        //
        IElementSelectionFilter idFilter = SelFilter.GetElementFilter( firstElement.Id, secondElement.Id, thirdElement.Id );
        IReferenceSelectionFilter xFilter = SelFilter.GetFaceNormalFilter( doc, XYZ.BasisX, true );
        IReferenceSelectionFilter yFilter = SelFilter.GetFaceNormalFilter( doc, XYZ.BasisY, true );
        ILogicalCombinationFilter logicalFilter = idFilter.And( xFilter.Or( yFilter ) );
#if DEBUG
        logicalFilter.ExecuteAll = true;
#endif
        pickedReference = sel.PickObject(
          ObjectType.Face, logicalFilter,
          "Now select any Face with an x oder y-Normal from any previously selected Element" );
        Element element = doc.GetElement( pickedReference );
        pickedFace = element.GetGeometryObjectFromReference( pickedReference ) as Face;
        pickedNormal = pickedFace.ComputeNormal( UV.Zero );
        TaskDialog.Show(
          "Result",
          "Fifth Selection: " + pickedFace.GetType().Name + " (" +
          pickedReference.ConvertToStableRepresentation( doc ) + ")\n" +
          "Normal: (" + pickedNormal.X.ToString( "0.###" ) + " / " + pickedNormal.Y.ToString( "0.###" ) + " / " +
          pickedNormal.Z.ToString( "0.###" ) + ")" );
      }
      catch( Autodesk.Revit.Exceptions.OperationCanceledException )
      {
        TaskDialog.Show( "Cancelled", "User cancelled" );
        return Result.Cancelled;
      }
      return Result.Succeeded;
    }
    #endregion
  }
}
```

Here is the pretty fully documented SelFilter class source code:

```csharp
public static class SelFilter
{
  //
  // Element-Filters: Will filter only for
  // elements, not for references
  //

  /// <summary>
  /// Creates a selection filter that only
  /// elements of type T will pass
  /// </summary>
  public static IElementSelectionFilter GetElementFilter<T>()
  {
    return new ElementTypeFilter( typeof( T ) );
  }

  /// <summary>
  /// Creates a selection filter that elements of
  /// any type in the collection will pass
  /// </summary>
  public static IElementSelectionFilter GetElementFilter(
    IEnumerable<Type> allowedTypes )
  {
    return new ElementTypeFilter( allowedTypes );
  }

  /// <summary>
  /// Creates a selection filter that elements
  /// of any of the given types  will pass
  /// </summary>
  public static IElementSelectionFilter GetElementFilter(
    Type type,
    params Type[] types )
  {
    return new ElementTypeFilter( type, types );
  }

  /// <summary>
  /// Creates a selection filter from an ElementFilter
  /// </summary>
  public static IElementSelectionFilter GetElementFilter(
    ElementFilter filter )
  {
    return new ElementFilterFilter( filter );
  }

  /// <summary>
  /// Creates a selection filter that will use
  /// the "filterMethod" to filter the elements
  /// </summary>
  public static IElementSelectionFilter GetElementFilter(
    Func<Element, bool> filterMethod )
  {
    return new DelegatesFilter( filterMethod, DelegatesFilter.AllReferences );
  }

  /// <summary>
  /// Creates a selection filter that will use
  /// the "filterMethod" to filter the elements
  /// </summary>
  public static IElementSelectionFilter GetElementFilter(
    Predicate<Element> filterMethod )
  {
    return new DelegatesFilter(
      new Func<Element, bool>( filterMethod ),
      DelegatesFilter.AllReferences );
  }

  /// <summary>
  ///  Creates a selection filter that will
  ///  let pass only the elements defined by the ids
  /// </summary>
  public static IElementSelectionFilter GetElementFilter(
    ElementId id,
    params ElementId[] ids )
  {
    return new ElementIdFilter( id, ids );
  }

  /// <summary>
  ///  Creates a selection filter that will
  ///  let pass only the elements defined by the ids
  /// </summary>
  public static IElementSelectionFilter GetElementFilter(
    IEnumerable<ElementId> ids )
  {
    return new ElementIdFilter( ids );
  }

  //
  // Reference-Filters: Will filter only for
  // references, let all elements pass
  //

  /// <summary>
  /// Creates a selection filter that will use
  /// the "filterMethod" to filter the references
  /// </summary>
  public static IReferenceSelectionFilter GetReferenceFilter(
    Func<Reference, XYZ, bool> filterMethod )
  {
    return new DelegatesFilter( DelegatesFilter.AllElements,
      filterMethod );
  }

  /// <summary>
  /// Creates a selection filter that will let only
  /// PlanarFace-References pass
  /// </summary>
  public static IReferenceSelectionFilter GetPlanarFaceFilter( Document doc )
  {
    return GetReferenceFilter( ( reference, xyz ) =>
    {
      Element element = doc.GetElement( reference );
      PlanarFace planarFace = element.GetGeometryObjectFromReference(
        reference ) as PlanarFace;
      return planarFace != null;
    } );
  }

  /// <summary>
  /// Creates a selection filter that will let faces
  /// pass if their normal vector at (0/0) is
  /// codirectional or parallel to the given normal
  /// vector
  /// </summary>
  public static IReferenceSelectionFilter GetFaceNormalFilter(
    Document doc,
    XYZ normal,
    bool acceptParallel = false )
  {
    XYZ \_normal = normal.Normalize();
    XYZ minusNormal = -1 \* \_normal;
    return GetReferenceFilter( ( reference, xyz ) =>
    {
      Element element = doc.GetElement( reference );
      Face face = element.GetGeometryObjectFromReference( reference ) as Face;
      if( face == null ) return false;
      XYZ faceNormal = face.ComputeNormal( UV.Zero );
      bool erg = faceNormal.IsAlmostEqualTo( \_normal );
      if( acceptParallel ) erg |= faceNormal.IsAlmostEqualTo( minusNormal );
      return erg;
    } );
  }

  //
  // Filter for Elements and References
  //

  public static ISelectionFilter GetFilter(
    Func<Element, bool> elementFilterMethod,
    Func<Reference, XYZ, bool> referencesFilterMethod )
  {
    return new DelegatesFilter( elementFilterMethod,
      referencesFilterMethod );
  }

  //
  // Logical-Filters: Will call other filters
  //

  /// <summary>
  /// Creates a logical "or" filter
  /// </summary>
  public static ILogicalCombinationFilter GetLogicalOrFilter(
    ISelectionFilter first,
    ISelectionFilter second,
    bool executeAll = false )
  {
    return new LogicalOrFilter( first, second, executeAll );
  }

  /// <summary>
  /// Creates a logical "or" filter
  /// </summary>
  public static ILogicalCombinationFilter GetLogicalOrFilter(
    ISelectionFilter first,
    params ISelectionFilter[] filters )
  {
    return new LogicalOrFilter( first, filters );
  }

  /// <summary>
  /// Creates a logical "or" filter
  /// </summary>
  public static ILogicalCombinationFilter GetLogicalOrFilter(
    IEnumerable<ISelectionFilter> filters )
  {
    return new LogicalOrFilter( filters );
  }

  /// <summary>
  /// Creates a logical "and" filter
  /// </summary>
  public static ILogicalCombinationFilter GetLogicalAndFilter(
    ISelectionFilter first,
    ISelectionFilter second,
    bool executeAll = false )
  {
    return new LogicalAndFilter( first, second, executeAll );
  }

  /// <summary>
  /// Creates a logical "and" filter
  /// </summary>
  public static ILogicalCombinationFilter GetLogicalAndFilter(
    ISelectionFilter first,
    params ISelectionFilter[] filters )
  {
    return new LogicalAndFilter( first, filters );
  }

  /// <summary>
  /// Creates a logical "and" filter
  /// </summary>
  public static ILogicalCombinationFilter GetLogicalAndFilter(
    IEnumerable<ISelectionFilter> filters )
  {
    return new LogicalAndFilter( filters );
  }

  /// <summary>
  /// Creates a logical "not" filter
  /// </summary>
  public static ISelectionFilter GetLogicalNotFilter(
    ISelectionFilter filter )
  {
    return new LogicalNotFilter( filter );
  }

  //
  // Extension Methods for SelectionFilters
  //

  /// <summary>
  /// Creates a logical "or" filter
  /// </summary>
  /// <param name="\_this"></param>
  /// <param name="filters"></param>
  /// <returns></returns>
  public static ILogicalCombinationFilter Or(
    this ISelectionFilter \_this,
    params ISelectionFilter[] filters )
  {
    return new LogicalOrFilter( \_this, filters );
  }

  /// <summary>
  /// Creates a logical "and" filter
  /// </summary>
  public static ILogicalCombinationFilter And(
    this ISelectionFilter \_this,
    params ISelectionFilter[] filters )
  {
    return new LogicalAndFilter( \_this, filters );
  }

  /// <summary>
  /// Creates a logical "not" filter
  /// </summary>
  public static ISelectionFilter Not(
    this ISelectionFilter \_this )
  {
    return new LogicalNotFilter( \_this );
  }

  //
  // Interfaces
  //

  //
  // Private classes will do the actual work -
  // don't need to be visible.
  // ISelectionFilter is not strictly neccessary on
  //each class, but I like to explicitly say what I
  // am doing.
  //

  /// <summary>
  /// Private class that represents a selection
  /// filter that will filter only for elements
  /// and lets all references pass
  /// </summary>
  private abstract class ElementSelectionFilter
    : IElementSelectionFilter,
    IReferenceSelectionFilter,
    ISelectionFilter
  {
    #region Implementation of ISelectionFilter

    public abstract bool AllowElement( Element elem );

    /// <summary>
    /// This class does not filter for references
    /// </summary>
    public bool AllowReference(
      Reference reference,
      XYZ position )
    {
      return true;
    }
    #endregion
  }

  /// <summary>
  /// Private class that represents a
  /// filter for one or more ElementTypes
  /// </summary>
  private class ElementTypeFilter
    : ElementSelectionFilter,
    IElementSelectionFilter,
    IReferenceSelectionFilter,
    ISelectionFilter
  {
    /// <summary>
    /// List of the allowed types
    /// </summary>
    private readonly List<Type> m\_types
      = new List<Type>();

    /// <summary>
    /// Constructs a Filter for a single Type
    /// </summary>
    public ElementTypeFilter( Type type )
    {
      m\_types.Add( type );
    }

    /// <summary>
    /// Constructs a Filter for a list of Types
    /// </summary>
    public ElementTypeFilter( IEnumerable<Type> types )
    {
      m\_types.AddRange( types );
    }

    /// <summary>
    /// Constructs a Filter for a number of Types
    /// </summary>
    public ElementTypeFilter(
      Type type,
      params Type[] types )
      : this( type )
    {
      m\_types.AddRange( types );
    }

    #region Implementation of ISelectionFilter

    public override bool AllowElement( Element elem )
    {
      return m\_types.Any( type
        => type.IsInstanceOfType( elem ) );
    }
    #endregion
  }

  /// <summary>
  /// Private class that represents a
  /// filter using an ElementFilter
  /// </summary>
  private class ElementFilterFilter
    : ElementSelectionFilter,
    IElementSelectionFilter,
    IReferenceSelectionFilter,
    ISelectionFilter
  {
    /// <summary>
    /// The ElementFilter that is used
    /// </summary>
    private readonly ElementFilter m\_filter;

    /// <summary>
    /// Constructs a SelectionFilter for an ElementFilter
    /// </summary>
    public ElementFilterFilter( ElementFilter filter )
    {
      m\_filter = filter;
    }

    #region Overrides of ElementSelectionFilter

    /// <summary>
    /// An element passes if it passes the ElementFilter
    /// </summary>
    public override bool AllowElement( Element elem )
    {
      return m\_filter.PassesFilter( elem );
    }
    #endregion
  }

  /// <summary>
  /// Private class that represents a filter
  /// for one or more specific elements.
  /// Especially usefull when and-combined
  /// with a ReferenceFilter
  /// </summary>
  private class ElementIdFilter
    : ElementSelectionFilter,
    IElementSelectionFilter,
    IReferenceSelectionFilter,
    ISelectionFilter
  {

    /// <summary>
    /// The list of valid ElementIds
    /// </summary>
    private readonly List<ElementId> m\_ids
      = new List<ElementId>();

    /// <summary>
    /// Constructs a ElementIdFilter for one or more ElementIds
    /// </summary>
    public ElementIdFilter(
      ElementId id,
      params ElementId[] ids )
    {
      m\_ids.Add( id );
      m\_ids.AddRange( ids );
    }

    /// <summary>
    /// Constructs a ElementIdFilter for a
    /// collection of ElementIds
    /// </summary>
    public ElementIdFilter( IEnumerable<ElementId> ids )
    {
      m\_ids.AddRange( ids );
    }

    #region Overrides of ElementSelectionFilter

    /// <summary>
    /// An Element passes if its Id is
    /// in the list of valid Ids
    /// </summary>
    public override bool AllowElement( Element elem )
    {
      return m\_ids.Contains( elem.Id );
    }
    #endregion
  }

  /// <summary>
  /// Private class that represents
  /// a filter using delegate methods
  /// </summary>
  private class DelegatesFilter
    : IElementSelectionFilter,
    IReferenceSelectionFilter,
    ISelectionFilter
  {
    /// <summary>
    /// The delegate used to filter elements
    /// </summary>
    private readonly Func<Element, bool> m\_elementFilter;

    /// <summary>
    /// The delegate used to filter References
    /// </summary>
    private readonly Func<Reference, XYZ, bool> m\_referenceFilter;

    /// <summary>
    /// Constructs a filter that uses the element
    /// and reference filter delegates
    /// </summary>
    public DelegatesFilter(
      Func<Element, bool> elementFilter,
      Func<Reference, XYZ, bool> referenceFilter )
    {
      m\_elementFilter = elementFilter;
      m\_referenceFilter = referenceFilter;
    }

    /// <summary>
    /// Use this if all Elements should pass the Filter
    /// </summary>
    public static Func<Element, bool> AllElements
    {
      get { return element => true; }
    }

    /// <summary>
    /// Use this, if all References should pass the Filter
    /// </summary>
    public static Func<Reference, XYZ, bool> AllReferences
    {
      get { return ( reference, xyz ) => true; }
    }

    #region Implementation of ISelectionFilter

    /// <summary>
    /// Elements that pass the element filter
    /// predicate will pass
    /// </summary>
    public bool AllowElement( Element elem )
    {
      return m\_elementFilter( elem );
    }

    /// <summary>
    /// Elements that pass the reference filter
    /// predicate will pass
    /// </summary>
    public bool AllowReference(
      Reference reference,
      XYZ position )
    {
      return m\_referenceFilter( reference, position );
    }
    #endregion
  }

  /// <summary>
  /// Private class that represents a logical or filter for selection filters.
  /// </summary>
  private abstract class LogicalCombinationFilter : ISelectionFilter, ILogicalCombinationFilter
  {
    /// <summary>
    /// List of the filters to check
    /// </summary>
    protected readonly List<ISelectionFilter> m\_filterList
      = new List<ISelectionFilter>();

    /// <summary>
    /// If true, all filters are executed,
    /// even if the result is already obvious
    /// </summary>
    public bool ExecuteAll { get; set; }

    /// <summary>
    /// Constructs a logical filter for two selectionFilters
    /// </summary>
    protected LogicalCombinationFilter(
      ISelectionFilter first,
      ISelectionFilter second,
      bool executeAll = false )
    {
      this.ExecuteAll = executeAll;
      m\_filterList.Add( first );
      m\_filterList.Add( second );
    }

    /// <summary>
    /// Constructs a logical filter for
    /// a number of selectionsFilters
    /// executeAll is set to false
    /// </summary>
    protected LogicalCombinationFilter(
      ISelectionFilter first,
      params ISelectionFilter[] filters )
    {
      this.ExecuteAll = false;
      m\_filterList.Add( first );
      m\_filterList.AddRange( filters );
    }

    /// <summary>
    /// Constructs a logical filter for
    /// a number of selectionsFilters
    /// executeAll is set to false
    /// </summary>
    /// <param name="filters"></param>
    protected LogicalCombinationFilter(
      IEnumerable<ISelectionFilter> filters )
    {
      ExecuteAll = false;
      m\_filterList.AddRange( filters );
    }

    #region Implementation of ISelectionFilter

    public abstract bool AllowElement(
      Element elem );

    public abstract bool AllowReference(
      Reference reference,
      XYZ position );

    #endregion
  }

  /// <summary>
  /// Private class that represents a logical or filter
  /// </summary>
  private class LogicalOrFilter
    : LogicalCombinationFilter, ISelectionFilter
  {

    /// <summary>
    /// Constructs a logical or filter for two Filters
    /// </summary>
    public LogicalOrFilter(
      ISelectionFilter first,
      ISelectionFilter second,
      bool executeAll )
      : base( first, second, executeAll )
    {
    }

    /// <summary>
    /// Constructs a logical or filter for
    /// a number of filters
    /// </summary>
    public LogicalOrFilter(
      ISelectionFilter first,
      params ISelectionFilter[] filters )
      : base( first, filters )
    {
    }

    /// <summary>
    /// Constructs a logical or filter for
    /// a number of filters
    /// </summary>
    public LogicalOrFilter(
      IEnumerable<ISelectionFilter> filters )
      : base( filters )
    {
    }

    #region Implementation of ISelectionFilter

    /// <summary>
    /// Elements that pass at least on
    /// of the filters will pass
    /// </summary>
    public override bool AllowElement( Element elem )
    {
      bool erg = false;
      if( ExecuteAll )
      {
        m\_filterList.ForEach( filter
          => erg |= filter.AllowElement( elem ) );
      }
      else
      {
        erg = m\_filterList.Any( filter
          => filter.AllowElement( elem ) );
      }
      return erg;
    }

    /// <summary>
    /// References that pass at least one
    /// of the filters will pass
    /// </summary>
    public override bool AllowReference(
      Reference reference,
      XYZ position )
    {
      bool erg = false;
      if( ExecuteAll )
      {
        m\_filterList.ForEach( filter
          => erg |= filter.AllowReference(
            reference, position ) );
      }
      else
      {
        erg = m\_filterList.Any( filter
          => filter.AllowReference(
            reference, position ) );
      }
      return erg;
    }

    #endregion
  }

  /// <summary>
  /// Private class that represents a logical and filter
  /// </summary>
  private class LogicalAndFilter
    : LogicalCombinationFilter, ISelectionFilter
  {

    /// <summary>
    /// Constructs a logical and filter for two Filters
    /// </summary>
    public LogicalAndFilter(
      ISelectionFilter first,
      ISelectionFilter second,
      bool executeAll )
      : base( first, second, executeAll )
    {
    }

    /// <summary>
    /// Constructs a logical and filter
    /// for a number of filters
    /// </summary>
    public LogicalAndFilter(
      ISelectionFilter first,
      params ISelectionFilter[] filters )
      : base( first, filters )
    {
    }

    /// <summary>
    /// Constructs a logical and
    /// filter for a number of filters
    /// </summary>
    public LogicalAndFilter(
      IEnumerable<ISelectionFilter> filters )
      : base( filters )
    {
    }

    #region Implementation of ISelectionFilter

    /// <summary>
    /// Elements that pass all of the filters will pass
    /// </summary>
    public override bool AllowElement( Element elem )
    {
      bool erg = true;
      if( ExecuteAll )
      {
        m\_filterList.ForEach( filter
          => erg = erg & filter.AllowElement( elem ) );
      }
      else
      {
        erg = m\_filterList.All( filter
          => filter.AllowElement( elem ) );
      }
      return erg;
    }

    /// <summary>
    /// References that pass all of the filters will pass
    /// </summary>
    public override bool AllowReference(
      Reference reference,
      XYZ position )
    {
      bool erg = true;
      if( ExecuteAll )
      {
        m\_filterList.ForEach( filter
          => erg = erg & filter.AllowReference(
            reference, position ) );
      }
      else
      {
        erg = m\_filterList.All( filter
          => filter.AllowReference(
            reference, position ) );
      }
      return erg;
    }
    #endregion
  }

  /// <summary>
  /// Private class that represents a logical Not filter
  /// </summary>
  private class LogicalNotFilter : ISelectionFilter
  {
    /// <summary>
    /// The filter, that will get negated
    /// </summary>
    private readonly ISelectionFilter m\_filter;

    /// <summary>
    /// constructs a not-Filter
    /// </summary>
    /// <param name="filter"> </param>
    public LogicalNotFilter( ISelectionFilter filter )
    {
      m\_filter = filter;
    }

    #region Implementation of ISelectionFilter

    /// <summary>
    /// Elements will pass, if they don't
    /// pass the original filter
    /// </summary>
    public bool AllowElement( Element elem )
    {
      return !m\_filter.AllowElement( elem );
    }

    /// <summary>
    /// References will pass, if they
    /// don't pass the original filter
    /// </summary>
    public bool AllowReference(
      Reference reference,
      XYZ position )
    {
      return !m\_filter.AllowReference(
        reference, position );
    }
    #endregion
  }
}
```

#### Culinary Notes from Paris

By the way, I am sitting in the meetup in London now.

Before ending for today, I have two little culinary notes from Paris to share:

I was lucky enough not to stay in horrible huge hotel there, but in a really nice little bed and breakfast instead, right next to
[Montmartre](https://en.wikipedia.org/wiki/Montmartre) and
[Sacré-Cœur](https://en.wikipedia.org/wiki/Sacr%C3%A9-C%C5%93ur,_Paris).

Better still, it was right next to the prize-winning baker and patissier
[Boulangerie Raphaëlle](http://boulangerieraphaelle.fr), so I had great croissants and chocolate for breakfast.

Then, to my surprise, arriving a bit early at the airport, I discovered the
[Ladurée](http://www.laduree.com) restaurant
at the Charles de Gaulle airport and enjoyed my first ever
[truffle omelette](http://www.traditionalfrenchfood.com/truffle-omelette.html) before
hopping on to the plane.