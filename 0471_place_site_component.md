---
post_number: "0471"
title: "Place Detail Instance"
slug: "place_site_component"
author: "Jeremy Tammik"
tags: ['csharp', 'elements', 'family', 'filtering', 'levels', 'parameters', 'references', 'revit-api', 'transactions', 'views']
source_file: "0471_place_site_component.htm"
original_url: "https://thebuildingcoder.typepad.com/blog/0471_place_site_component.html"
---

### Place Detail Instance

After several discussions on placing family instances, such as the use of
[PromptForFamilyInstancePlacement](http://thebuildingcoder.typepad.com/blog/2010/06/place-family-instance.html) and placing
[detail instances in 2D](http://thebuildingcoder.typepad.com/blog/2010/10/place-detail-instance.html),
here is a another question in a similar vein on placing trees and site components from Jeremiah Farmer of
[Land F/X](http://www.landfx.com).

And by the way, I am hoping (ever so slightly) that this discussion and the tools presented below may help Kristoffer with his
[furniture insertion problem](http://thebuildingcoder.typepad.com/blog/2009/02/inserting-a-beam.html?cid=6a00e553e168978833013488b40382970c#comment-6a00e553e168978833013488b40382970c) as well.

**Question:** I am trying to automate the placement of trees into the Revit model.
I can't seem to find the appropriate NewFamilyInstance overload to call, or call it correctly.
And **nowhere** can I locate an example of placing a Site Component through the API.
This is killing me!
Please, please help with this, or if you can direct me somewhere.
Know that this will greatly speed Disney's adoption of Revit, as well as many of our other clients.

**Answer:** What have you tried so far?
Have you seen the discussion on
[placing a detail instance](http://thebuildingcoder.typepad.com/blog/2010/10/place-detail-instance.html)?

Two snippets of code that may be useful for testing that cropped up since I published that:

1. [PlaceInstancesOnViews](#1): This method tests placing a specific family instance in all views, to ensure that a detail instance can indeed be placed in a detail view using the NewFamilyInstance overload taking the arguments XYZ, targetFamily, view.- [TestAllOverloads](#2): This method calls all possible overloads of NewFamilyInstance in order to find one that works.

Here is the code for these two:

#### 1. PlaceInstancesOnViews

Test placing a specific family instance in all views, to ensure that a detail instance can indeed be placed in a detail view using the NewFamilyInstance overload taking the arguments XYZ, targetFamily, view:
```csharp
void PlaceInstancesOnViews( Document doc )
{
  FilteredElementCollector collector
    = new FilteredElementCollector( doc );

  collector.OfClass( typeof( FamilySymbol ) );

  Func<FamilySymbol, bool> isTargetFamily
    = famSym => famSym.Name.Contains( "Rowlock" );

  FamilySymbol targetFamily
    = collector.OfType<FamilySymbol>()
      .Where<FamilySymbol>( isTargetFamily )
      .First<FamilySymbol>();

  collector = new FilteredElementCollector( doc );
  collector.OfClass( typeof( View ) );

  Func<View, bool> isPossibleView
    = v => !v.IsTemplate && !( v is View3D );

  IEnumerable<View> possibleViews
    = collector.OfType<View>()
      .Where<View>( isPossibleView );

  foreach( View view in possibleViews )
  {
    Transaction t = new Transaction( doc,
      "Add instance to " + view.Name );

    t.Start();

    try
    {
      FamilyInstance result
        = doc.Create.NewFamilyInstance(
          XYZ.Zero, targetFamily, view );

      t.Commit();
    }
    catch( Exception ex )
    {
      t.RollBack();
    }
  }
}
```

#### 2. TestAllOverloads

Test calling all possible NewFamilyInstance overloads on the given symbol in order to find one that works:
```csharp
void TestAllOverloads(
  Document doc,
  XYZ startPoint,
  XYZ endPoint,
  FamilySymbol familySymbol )
{
  StructuralType stNon = StructuralType.NonStructural;
  StructuralType stBeam = StructuralType.Beam;

  Autodesk.Revit.Creation.Document cd
    = doc.Create;

  View view = doc.ActiveView;
  SketchPlane sk = view.SketchPlane;
  Level level = view.Level;

  // Create line from user points

  Curve curve = doc.Application.Create.NewLineBound( startPoint, endPoint );

  // Create direction vector from user points

  XYZ dirVec = endPoint - startPoint;

  bool done = false;
  int index = 1;
  while( !done )
  {
    FamilyInstance instance = null;

    // Try different insert methods

    try
    {
      switch( index )
      {
        // public FamilyInstance NewFamilyInstance(
        //   XYZ location, FamilySymbol symbol,
        //   StructuralType structuralType );

        case 1:
          instance = cd.NewFamilyInstance(
            startPoint, familySymbol, stNon );
          break;

        case 2:
          instance = cd.NewFamilyInstance(
            startPoint, familySymbol, stBeam );
          break;

        // public FamilyInstance NewFamilyInstance(
        //   XYZ origin, FamilySymbol symbol,
        //   View specView );

        case 3:
          instance = cd.NewFamilyInstance(
            startPoint, familySymbol, null );
          break;

        case 4:
          instance = cd.NewFamilyInstance(
            startPoint, familySymbol, view );
          break;

        // public FamilyInstance NewFamilyInstance(
        //   XYZ location, FamilySymbol symbol,
        //   Element host, StructuralType structuralType );

        case 5:
          instance = cd.NewFamilyInstance(
            startPoint, familySymbol, sk, stNon );
          break;

        case 6:
          instance = cd.NewFamilyInstance(
            startPoint, familySymbol, sk, stBeam );
          break;

        // public FamilyInstance NewFamilyInstance(
        //   XYZ location, FamilySymbol symbol,
        //   XYZ referenceDirection, Element host,
        //   StructuralType structuralType );

        case 7:
          instance = cd.NewFamilyInstance(
            startPoint, familySymbol, dirVec, sk,
            stNon );
          break;

        case 8:
          instance = cd.NewFamilyInstance(
            startPoint, familySymbol, dirVec, sk,
            stBeam );
          break;

        // public FamilyInstance NewFamilyInstance(
        //   Curve curve, FamilySymbol symbol,
        //   Level level, StructuralType structuralType );

        case 9:
          instance = cd.NewFamilyInstance(
            curve, familySymbol, null, stNon );
          break;

        case 10:
          instance = cd.NewFamilyInstance(
            curve, familySymbol, null, stBeam );
          break;

        case 11:
          instance = cd.NewFamilyInstance(
            curve, familySymbol, level, stNon );
          break;

        case 12:
          instance = cd.NewFamilyInstance(
            curve, familySymbol, level, stBeam );
          break;

        // public FamilyInstance NewFamilyInstance(
        //   XYZ location, FamilySymbol symbol,
        //   Level level, StructuralType structuralType );

        case 13:
          instance = cd.NewFamilyInstance(
            startPoint, familySymbol, null, stNon );
          break;

        case 14:
          instance = cd.NewFamilyInstance(
            startPoint, familySymbol, null, stBeam );
          break;

        case 15:
          instance = cd.NewFamilyInstance(
            startPoint, familySymbol, level, stNon );
          break;

        case 16:
          instance = cd.NewFamilyInstance(
            startPoint, familySymbol, level, stBeam );
          break;

        // public FamilyInstance NewFamilyInstance(
        //   XYZ location, FamilySymbol symbol,
        //   Element host, Level level,
        //   StructuralType structuralType );

        case 17:
          instance = cd.NewFamilyInstance(
            startPoint, familySymbol, null, stNon );
          break;

        case 18:
          instance = cd.NewFamilyInstance(
            startPoint, familySymbol, null, stBeam );
          break;

        case 19:
          instance = cd.NewFamilyInstance(
            startPoint, familySymbol, sk, stNon );
          break;

        case 20:
          instance = cd.NewFamilyInstance(
            startPoint, familySymbol, sk, stBeam );
          break;

        default:
          done = true;
          break;
      }
    }
    catch
    { }

    // If instance was created, mark with identifier so I can see which instances were created

    if( null != instance )
    {
      Parameter param = instance.get\_Parameter( "InstanceIndex" );
      if( null != param )
      {
        param.Set( index );
      }
    }
    index++;
  }
}
```

I hope that this will be of use in you further research.

Please let me know how you fare and what you come up with!

**Response:** I had not seen that example, but that is very similar to what I finally got working last night.

It seems Site Components need to be attached to the Topo Surface, so I end up calling NewFamilyInstance with XYZ, FamilySymbol, TopoSurface, StructuralType.

It turns out the Z coordinate can be zero, by specifying TopoSurface it will be anchored correctly.

I also had to come up with two helper functions, one to locate the TopoSurface, and one to locate the FamilySymbol reference (as LoadFamilySymbol only returns a Boolean, not a reference to the FamilySymbol!
How crazy is that?).

So I think I'm on my way.
I can't tell you how much I appreciate you getting back to me!

I'd be happy to share!
Here is
[placeplant.vb](zip/placeplant.vb) containing
the basic outline of what I have so far, representing the first portion of our Revit add-on.
I should inform you that this is to be part of a commercial application, Land F/X.

The hurdles I have cleared so far were finding the appropriate overload of NewFamilyInstance to use for Site Components, as well as the creation of the two helper functions (which I'm positive can be simplified or optimized in some fashion, but at least they work!).
You can see I have also figured out how to set various parameters of the Family Symbol, as well as the Instance, and have marked up my findings on later creating custom Parameters for these values.

Very many thanks to Jeremiah for his enthusiasm and this solution and sample code for placing site components!