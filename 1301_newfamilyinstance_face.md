---
ai_optimized: true
author: Jeremy Tammik
complexity_score: 8.7
content_type: qa
optimization_date: '2025-12-11T11:44:15.717027'
original_url: https://thebuildingcoder.typepad.com/blog/1301_newfamilyinstance_face.html
post_number: '1301'
reading_time_minutes: 15
series: elements
slug: newfamilyinstance_face
source_file: 1301_newfamilyinstance_face.htm
tags:
- csharp
- elements
- family
- filtering
- levels
- parameters
- references
- revit-api
- selection
- transactions
- views
title: Re-Researching Lighting Family Instance Placement
word_count: 2971
---

### Re-Researching Lighting Family Instance Placement

The placement of family instances can sometimes be a tricky topic in the Revit API.

Programmatically, this is always achieved using the NewFamilyInstance method.

However, this method provides 13 different overloads to choose from, which can be a non-trivial task:

- NewFamilyInstance(Face, Line, FamilySymbol) – Inserts a new instance of a family onto a face of an existing element, using a line on that face for its position, and a type/symbol.
- NewFamilyInstance(Line, FamilySymbol, View) – Add a line based detail family instance into the Autodesk Revit document, using an line and a view where the instance should be placed.
- NewFamilyInstance(Reference, Line, FamilySymbol) – Inserts a new instance of a family onto a face referenced by the input Reference instance, using a line on that face for its position, and a type/symbol.
- NewFamilyInstance(XYZ, FamilySymbol, StructuralType) – Inserts a new instance of a family into the document, using a location and a type/symbol.
- NewFamilyInstance(XYZ, FamilySymbol, View) – Add a new family instance into the Autodesk Revit document, using an origin and a view where the instance should be placed.
- NewFamilyInstance(Curve, FamilySymbol, Level, StructuralType) – Inserts a new instance of a family into the document, using a curve, type/symbol and reference level.
- NewFamilyInstance(Face, XYZ, XYZ, FamilySymbol) – Inserts a new instance of a family onto a face of an existing element, using a location, reference direction, and a type/symbol.
- NewFamilyInstance(Reference, XYZ, XYZ, FamilySymbol) – Inserts a new instance of a family onto a face referenced by the input Reference instance, using a location, reference direction, and a type/symbol.
- NewFamilyInstance(XYZ, FamilySymbol, Level, StructuralType) – Inserts a new instance of a family into the document, using a location, type/symbol and a base level.
- NewFamilyInstance(XYZ, FamilySymbol, Element, StructuralType) – Inserts a new instance of a family into the document, using a location, type/symbol, and the host element.
- NewFamilyInstance(XYZ, FamilySymbol, Element, Level, StructuralType) – Inserts a new instance of a family into the document, using a location, type/symbol, the host element and a base level.
- NewFamilyInstance(XYZ, FamilySymbol, XYZ, Element, StructuralType) – Inserts a new instance of a family into the document, using a location, type/symbol, the host element and a reference direction.

The Building Coder topic group
[5.25](http://thebuildingcoder.typepad.com/blog/about-the-author.html#5.25) deals with the
[Family API and placing family instances](http://thebuildingcoder.typepad.com/blog/about-the-author.html#5.25) in
general and explored the more specialised issue of selecting the correct
[NewFamilyInstance method overload](http://thebuildingcoder.typepad.com/blog/2011/01/newfamilyinstance-overloads.html) to
use for specific given families way back in 2011.

The addition of the
[family instance placement type](http://thebuildingcoder.typepad.com/blog/2013/09/family-instance-placement.html) functionality
with the FamilyPlacementType enumeration and a corresponding property on the Family class simplified this task significantly.

It can still be non-trivial figuring out which overload to use though, especially if you are lazy or forgetful and don't remember all of the possibilities, as in my case.

Here is an interesting example of a completely useless and much too exhaustive search to determine how to create copies of a ceiling hosted light fixture, only to end up with the same result we already published in 2012 on
[hosting a light fitting on a reference plane](http://thebuildingcoder.typepad.com/blog/2012/02/hosting-a-light-fitting-on-a-reference-plane.html).

Well, hopefully not completely useless, after all, since this exploration does show you several steps and techniques to approach and narrow down the problem to finally rediscover a solution that could also have been found faster and more easily:

**Question:**

In my add-in I am trying to:

1. Get user to select light already placed in model.
2. Allow user to pick new point to place previously selected light
3. Attach light to new point hosted on the same reference plane as the original light.

I have no problems with the first two steps, but the last step that gives me the following error:

"Reference direction is parallel to face normal at insertion point."

I read the discussions on
[hosting a light fitting on a reference plane](http://thebuildingcoder.typepad.com/blog/2012/02/hosting-a-light-fitting-on-a-reference-plane.html) and
[family instance on reference plane with API](http://forums.autodesk.com/t5/revit-api/family-instance-on-reference-plane-with-api/td-p/4962340),
but they did not help.

All I can find on-line are more people with the exact same issue and no solutions.

Please can you help?

I provided a minimal sample with a single lighting fixture hosted by a ceiling and an external command that should allow you to select one of the lights, then prompt you to pick a new spot for a copy of it.

The new light should be placed on the same work plane as the original.

**Answer:**
I implemented a complete add-in to test your external command and tried it out in your model:

![Minimal sample model](img/place_light_1_model.png)

If you look at the lighting fixture in RevitLookup, you will note that it is hosted by the ceiling element and has a valid Level property value:

![Snoop light fixture family instance](img/place_light_2_snoop_instance.png)

When run the add-in command, it executes fine for me, with no errors produced at all.

However, I do not see anything added to the graphics screen.

There are several things you can and should do at this point to find out more:

- Check the return value of the NewFamilyInstance method.
- Use the
  [element lister](http://thebuildingcoder.typepad.com/blog/2014/09/debugging-and-maintaining-the-image-relationship.html#2) to see what elements were added to the database, if any.

In this case, I can also simply snoop the database for all family instances:

![Snoop light fixture family instance copy](img/place_light_3_snoop_copy.png)

There are two of them now.

One of them is the new copy that we successfully created.

The copy, however, is hosted by an abstract reference plane, not by the ceiling element, like the original instance.

Exploring its Level property will show that it has a value of -1, i.e. an invalid element id.

That probably explains why it is not displayed in the Level 1 ceiling plan view like the original instance.

Ah, no, it does not; I see that the original's level property is also -1, so that's apparently not a problem.

Anyway, that is the original implementation.

I created a new
[PlaceLight GitHub repository](https://github.com/jeremytammik/PlaceLight) for
this project and saved the initial version is saved as
[release 2015.0.0.0](https://github.com/jeremytammik/PlaceLight/releases/tag/2015.0.0.0).

Next step: specify the host element when calling NewFamilyInstance:

```csharp
static FamilyInstance PlaceALight(
XYZ lightPlacePoint,
Element host,
FamilySymbol lightSymbol )
{
Document doc = lightSymbol.Document;
return doc.Create.NewFamilyInstance(
lightPlacePoint, lightSymbol, host,
Autodesk.Revit.DB.Structure.StructuralType
.NonStructural );
}
```

As you can see, it is much simpler to just specify the host rather than create a new reference plane for it.

It is called like this now to pass in the original host:

```csharp
PlaceALight( placeXyzPoint, lightFamilyInstance.Host, lightFamilySymbol );
```

I stored this attempt as
[release 2015.0.0.1](https://github.com/jeremytammik/PlaceLight/releases/tag/2015.0.0.1).

However, it still does not work perfectly.

Initially, I saw no new element added at all.

Peeping under the ceiling in 3D view showed the new element offset downwards:

![Light fixture copy offset downwards](img/place_light_4_copy_offset.png)

Why is it offset downwards?

I compared the parameter values of the original and copy using the
[BipChecker](http://thebuildingcoder.typepad.com/blog/2014/05/bipchecker-for-revit-2015-on-github.html).

One difference that I noticed was in the built-in parameter SKETCH\_PLANE\_PARAM.

I tried to set that as demonstrated in
[release 2015.0.0.2](https://github.com/jeremytammik/PlaceLight/releases/tag/2015.0.0.2).

That throws an exception, because the parameter is read-only.

Next, I tried a much simpler solution:

Still comparing the original with the copy, I notice that the original location point Z value in non-zero, whereas the copy's is zero.

So I simply set the copy's Z value equal to the original before creating the new instance, et voila, it appears to work fine:

![Light fixture copy with Z elevation](img/place_light_5_copy_with_z.png)

I stored that version as
[release 2015.0.0.3](https://github.com/jeremytammik/PlaceLight/releases/tag/2015.0.0.3), then cleaned it up a bit to illustrate the various failed and successful attempts more clearly, resulting in
[release 2015.0.0.4](https://github.com/jeremytammik/PlaceLight/releases/tag/2015.0.0.4):

```csharp
#region Namespaces
using System;
using Autodesk.Revit.ApplicationServices;
using Autodesk.Revit.Attributes;
using Autodesk.Revit.DB;
using Autodesk.Revit.UI;
using Autodesk.Revit.UI.Selection;
using OperationCanceledException = Autodesk.Revit.Exceptions.OperationCanceledException;
#endregion

namespace PlaceLight
{
  public class LightPickFilter : ISelectionFilter
  {
    public bool AllowElement( Element e )
    {
      return e.Category.Id.IntegerValue.Equals(
        (int) BuiltInCategory.OST\_LightingFixtures );
    }

    public bool AllowReference( Reference r, XYZ p )
    {
      return false;
    }
  }

  [Transaction( TransactionMode.Manual )]
  public class Command : IExternalCommand
  {
    static FamilyInstance PlaceALight(
      XYZ lightPlacePoint,
      Element host,
      FamilySymbol lightSymbol )
    {
      Document doc = lightSymbol.Document;

      // This does not work, because we need to
      // specify a valid BIM element host.

      //XYZ bubbleEnd = new XYZ( 5, 0, 0 );
      //XYZ freeEnd = new XYZ( -5, 0, 0 );
      //XYZ thirdPt = new XYZ( 0, 0, 1 );
      //ReferencePlane referencePlane
      //  = doc.Create.NewReferencePlane2( bubbleEnd,
      //    freeEnd, thirdPt, doc.ActiveView );
      //XYZ xAxisOfPlane = new XYZ( 0, 0, -1 );
      //doc.Create.NewFamilyInstance(
      //  referencePlane.Reference, lightPlacePoint,
      //  xAxisOfPlane, lightSymbol );

      FamilyInstance inst = doc.Create.NewFamilyInstance(
        lightPlacePoint, lightSymbol, host,
        Autodesk.Revit.DB.Structure.StructuralType
          .NonStructural );

      // This does not work, because the parameter
      // is read-only, so an exception is thrown.

      //inst.get\_Parameter(
      //  BuiltInParameter.SKETCH\_PLANE\_PARAM )
      //    .Set( sketchPlaneName );

      return inst;
    }

    public Result Execute(
      ExternalCommandData commandData,
      ref string message,
      ElementSet elements )
    {
      var uiApp = commandData.Application;
      var doc = uiApp.ActiveUIDocument.Document;

      try
      {
        Selection selection = uiApp.ActiveUIDocument.Selection;

        // Pick a light fixture.

        var pickedLightReference = selection.PickObject(
          ObjectType.Element, new LightPickFilter(),
          "Please select lighting fixture to place" );

        if( pickedLightReference == null )
        {
          return Result.Failed;
        }

        // Get Family Instance of the selected light reference.

        FamilyInstance lightFamilyInstance
          = doc.GetElement( pickedLightReference )
            as FamilyInstance;

        // Get FamilySymbol of the family instance.

        if( lightFamilyInstance == null )
        {
          return Result.Failed;
        }

        FamilySymbol lightFamilySymbol
          = lightFamilyInstance.Symbol;

        // Forget this, it is read-only anyway.
        //Parameter sketchPlaneParam = lightFamilyInstance
        //  .get\_Parameter( BuiltInParameter.SKETCH\_PLANE\_PARAM );
        //string sketchPlaneName = sketchPlaneParam.AsString();

        // Get new light location.

        XYZ placeXyzPoint = selection.PickPoint(
          "Select Point to place light:" );

        // Assuming the ceiling is horizontal, set
        // the location point Z value for the copy
        // equal to the original.

        placeXyzPoint = new XYZ( placeXyzPoint.X,
          placeXyzPoint.Y, ( lightFamilyInstance
            .Location as LocationPoint ).Point.Z );

        using( var trans = new Transaction( doc ) )
        {
          trans.Start( "LightArray" );

          // Start placing lights.

          FamilyInstance lightFamilyInstance2
            = PlaceALight( placeXyzPoint,
              lightFamilyInstance.Host,
              lightFamilySymbol );

          trans.Commit();
        }
      }
      catch( OperationCanceledException )
      {
        return Result.Cancelled;
      }
      catch( Exception ex )
      {
        message = ex.Message;
        return Result.Failed;
      }
      return Result.Succeeded;
    }
  }
}
```

As a result of all this, we can remove the PlaceALight method definition, since it has been reduced to a one-liner, and simplify the entire external command implementation to the one and only single Execute method implementation.

I also removed all the unneeded code and comments illustrating the various failed and successful attempts and stored the final minimal working version as
[release 2015.0.0.5](https://github.com/jeremytammik/PlaceLight/releases/tag/2015.0.0.5):

Here is the Execute method implementation, with no other helper methods required:

```csharp
  public Result Execute(
    ExternalCommandData commandData,
    ref string message,
    ElementSet elements )
  {
    var uiApp = commandData.Application;
    var doc = uiApp.ActiveUIDocument.Document;

    try
    {
      Selection selection = uiApp.ActiveUIDocument
        .Selection;

      // Pick a light fixture.

      var pickedLightReference = selection.PickObject(
        ObjectType.Element, new LightPickFilter(),
        "Please select lighting fixture to place" );

      if( null == pickedLightReference )
      {
        return Result.Failed;
      }

      // Get Family Instance of the selected light reference.

      FamilyInstance lightFamilyInstance
        = doc.GetElement( pickedLightReference )
          as FamilyInstance;

      // Get FamilySymbol of the family instance.

      if( lightFamilyInstance == null )
      {
        return Result.Failed;
      }

      FamilySymbol lightFamilySymbol
        = lightFamilyInstance.Symbol;

      // Determine the host BIM element.

      Element host = lightFamilyInstance.Host;

      // Get new light location.

      XYZ placeXyzPoint = selection.PickPoint(
        "Select Point to place light:" );

      // Assuming the ceiling is horizontal, set
      // the location point Z value for the copy
      // equal to the original.

      placeXyzPoint = new XYZ( placeXyzPoint.X,
        placeXyzPoint.Y, ( lightFamilyInstance
          .Location as LocationPoint ).Point.Z );

      // All lighting fixtures are non-strucutral.

      Autodesk.Revit.DB.Structure.StructuralType
        non\_structural = Autodesk.Revit.DB.Structure
          .StructuralType.NonStructural;

      using( var trans = new Transaction( doc ) )
      {
        trans.Start( "LightArray" );

        // Start placing lights.

        FamilyInstance lightFamilyInstance2
          = doc.Create.NewFamilyInstance(
            placeXyzPoint, lightFamilySymbol,
            host, non\_structural );

        trans.Commit();
      }
    }
    catch( OperationCanceledException )
    {
      return Result.Cancelled;
    }
    catch( Exception ex )
    {
      message = ex.Message;
      return Result.Failed;
    }
    return Result.Succeeded;
  }
```

To summarise, I never saw the error you reported, but encountered and solved two other fundamental problems:

- No BIM element host specified
- No Z elevation value specified

Fixing those two was all it took.

**Response:**

Looking good, however there are a few issues.

I ran the routine and the light was placed as expected.

If you select the newly placed light and bring up its properties the Offset is no longer ‘0’ it is ‘2600’

Also the Work Plane says ‘???’ not Compound Ceiling: Plain (as the original)

Here are the properties displayed for a newly placed light (using the routine):

![Light fixture copy properties](img/place_light_6_copy_properties.png)

I would expect the light to be placed with the same offset and work plane as the original.

**Answer:**
I continued research to find the correct solution to your issue, exploring various characteristics obtainable for the lighting instance properties and methods such as GetFamilyPointPlacementReferences, LevelId, HostFace, Location and trying to feed that data into different other overloads of the NewFamilyInstance method.

An important question is the family placement type that can be determined like this:

```csharp
  FamilyPlacementType placementType
    = lightFamilySymbol.Family
      .FamilyPlacementType;
```

In this case, it returns placementType = WorkPlaneBased.

Using that, we can return to the ItemFactoryBase class list of NewFamilyInstance overloads and choose a suitable one for that placement type.

That still leaves several different overloads to choose from.

I finally discovered that I can simply take the reference returned by the HostFace property and feed it into the NewFamilyInstance overload taking a reference, insertion point, direction and the family symbol to use.

[Release 2015.0.0.6](https://github.com/jeremytammik/PlaceLight/releases/tag/2015.0.0.6) shows
the final correctly working solution with parts of the code from previous attempts commented out.

After discovering this, I returned to the old blog post on
[hosting a light fitting on a reference plane](http://thebuildingcoder.typepad.com/blog/2012/02/hosting-a-light-fitting-on-a-reference-plane.html) that
you originally mentioned, only to see that it already clearly discusses the importance of choosing the right NewFamilyInstance method overload for the task, and points out that this exact one taking a reference plane, placement point, direction and family symbol is the right one to do the job in this case – the same result I ended up with after all this exhaustive renewed exploration for your case.

On one hand, I am sorry that I did not take a closer look at it to start with.

I am even more sorry that the post apparently did not get the message across in the first place.

Still, I hope that this renewed research clarifies and adds one or two bits of new interesting information.

It maybe also illustrates that perseverance can pay off, even if you are as stupid as I am and refuse to learn from experience, or refer back to it... :-)

Here is the final external command implementation after removing all the experimental code snippets:

```csharp
[Transaction( TransactionMode.Manual )]
public class Command : IExternalCommand
{
  public Result Execute(
    ExternalCommandData commandData,
    ref string message,
    ElementSet elements )
  {
    UIApplication uiapp = commandData.Application;
    UIDocument uidoc = uiapp.ActiveUIDocument;
    Document doc = uidoc.Document;

    try
    {
      Selection selection = uidoc.Selection;

      // Pick a light fixture.

      var pickedLightReference = selection.PickObject(
        ObjectType.Element, new LightPickFilter(),
        "Please select lighting fixture to place" );

      if( null == pickedLightReference )
      {
        return Result.Failed;
      }

      // Get Family Instance of the selected light reference.

      FamilyInstance lightFamilyInstance
        = doc.GetElement( pickedLightReference )
          as FamilyInstance;

      // Get FamilySymbol of the family instance.

      if( lightFamilyInstance == null )
      {
        return Result.Failed;
      }

      FamilySymbol lightFamilySymbol
        = lightFamilyInstance.Symbol;

      // Determine this family's placement type.
      // This is an important step towards determining
      // which NewFamilyInstance overload to use to
      // place new instances of it.

      FamilyPlacementType placementType
        = lightFamilySymbol.Family
          .FamilyPlacementType;

      // Placement type is WorkPlaneBased, so determine
      // the host face that defines the work plane.

      Reference hostFace = lightFamilyInstance.HostFace;

      // Prompt for placement point of copy.

      XYZ placeXyzPoint = selection.PickPoint(
        "Select point to place new light:" );

      // The location point gives Z elevation value.

      LocationPoint lp = lightFamilyInstance.Location
        as LocationPoint;

      // Assuming the ceiling is horizontal, set
      // the location point Z value for the copy
      // equal to the original.

      placeXyzPoint = new XYZ( placeXyzPoint.X,
        placeXyzPoint.Y, lp.Point.Z );

      using( var trans = new Transaction( doc ) )
      {
        trans.Start( "LightArray" );

        FamilyInstance lightFamilyInstance2
          = doc.Create.NewFamilyInstance(
            hostFace, placeXyzPoint, XYZ.BasisX,
            lightFamilySymbol );

        trans.Commit();
      }
    }
    catch( OperationCanceledException )
    {
      return Result.Cancelled;
    }
    catch( Exception ex )
    {
      message = ex.Message;
      return Result.Failed;
    }
    return Result.Succeeded;
  }
}
```

As said, the most up-to-date version is provided by the
[PlaceLight GitHub repository](https://github.com/jeremytammik/PlaceLight).
Here is an overview of the releases so far:

- 2015.0.0.7 removed obsolete testing and commented code
- 2015.0.0.6 finally found right NewFamilyInstance overload to use
- 2015.0.0.5 removed unneeded code and comments for final working version
- 2015.0.0.4 cleaned up for publication
- 2015.0.0.3 just passing in the host and setting the correct Z value seems to work fine
- 2015.0.0.2 set SKETCH\_PLANE\_PARAM, which throws an exception being read-only
- 2015.0.0.1 use NewFamilyInstance with a given host BIM element
- 2015.0.0.0 initial implementation copies fixture with wrong host