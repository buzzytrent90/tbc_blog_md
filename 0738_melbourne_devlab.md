---
ai_optimized: true
author: Jeremy Tammik
complexity_score: 4.7
content_type: code_example
optimization_date: '2025-12-11T11:44:14.490834'
original_url: https://thebuildingcoder.typepad.com/blog/0738_melbourne_devlab.html
post_number: 0738
reading_time_minutes: 9
series: general
slug: melbourne_devlab
source_file: 0738_melbourne_devlab.htm
tags:
- csharp
- elements
- family
- filtering
- geometry
- parameters
- python
- references
- revit-api
- schedules
- sheets
- transactions
- views
- windows
title: Melbourne DevLab
word_count: 1804
---

### Melbourne DevLab

The last few days I started walking on foot from Flinder Station to Queen's Road.
I can enjoy lush green grass, beautiful huge trees and strange birdsong in morning sunlight all the way past the Shrine of Remembrance.
It takes just ten minutes longer than waiting for the overfilled noisy slow tram, in which I am forced to stand and therefore can't look out through the windows due to my height.

The Melbourne Revit API training
[day one](http://thebuildingcoder.typepad.com/blog/2012/03/melbourne-day-one.html) and
[day two](http://thebuildingcoder.typepad.com/blog/2012/03/melbourne-day-two.html) were
followed by a DevLab.

A DevLab is an open event where developers roll up bringing their current work and issues and continue developing side by side with one or more ADN DevTech engineers, in this case poor old me.

We had three days to play with and achieved some quite exciting and impressive results.
Here are some of them:

- Delete all 3D views.- [Retrieve specific views](#1): GetNicksViews.- Find unnamed reference planes using a parameter filter.- [Delete reference planes not hosting any elements](#2): DeleteUnnamedNonHostingReferencePlanes.- Find elements which have had certain information added, so they can be deleted and updated.- [Automatically create setout points](#3) for on-site location and construction of structural elements.

The first couple of items were things I worked on with Nicholas Broadbent of
[COX Architecture](http://www.coxarchitecture.com.au),
who is also involved in a whole bunch of other exciting projects, such as:

- A Navisworks clash detection data importer for Revit.
  - Save Navisworks clash detection data into the clashing elements in Revit.- A link between Rhino (using the Grasshopper plug-in) and Revit.
    - Draw Revit straight and arced beams from points exported from Rhino.- Place family instances and adaptive components using the same exported data.- Avoid using SAT files. A known issue is that they can crash Revit.- Exporting a Revit model to the Unity Gaming engine.
      - Looking up Revit data in an Access database based on a Revit element id.- Display Revit data in the Unity gaming engine when you click on an element.- FBX C++ API
        - Modify an FBX file, renaming and merging nodes.

The second and third days I spent most of the time together with Paul Hellawell of
[GHD](http://www.ghd.com) working
on a full-blown end-user-capable little application for the automatic creation of setout points for on-site location and construction of structural elements.
I'll say a little bit about it
[below](#3) and
we plan to post the full description, add-in and support files sometime soon.

After DevLab, we went off on a visit to a pub or two which is right now starting to turn into a crawl.
Here are The DevLab Boys at the
[Loop](http://www.looponline.com.au):

![The DevLab boys in the Loop](file:////j/photo/jeremy/2012/2012-03-23_pub_melbourne/loop_the_devlab_boys.jpg)

This place calls itself an architectural and experimental space and used to be the venue of
[Revic](http://www.revic.org.au),
the Revit Users Group of Victoria.

From there we moved on to
[Siglo](http://www.theage.com.au/news/bar-reviews/siglo-bar/2008/05/12/1210444319156.html),
in the same house as the
[Supper Club](http://www.theage.com.au/news/bar-reviews/the-melbourne-supper-club/2006/04/03/1143916451849.html) that
I visited
[just a few days ago](http://thebuildingcoder.typepad.com/blog/2012/03/melbourne-day-two.html):

![Siglo](file:////j/photo/jeremy/2012/2012-03-23_pub_melbourne/siglo.jpg)

#### Retrieving Specific Views

Actually, we have had a couple of beers by now, and both Nick and Paul are enjoying 40 dollar cigars (credit: Nick bought), so I will simply post this with no further comment:
```python
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
    Application app = uiapp.Application;
    Document doc = uidoc.Document;

    List<ElementFilter> a
      = new List<ElementFilter>( 4 );

    a.Add( new ElementClassFilter( typeof( View3D ) ) );
    a.Add( new ElementClassFilter( typeof( ViewDrafting ) ) );
    a.Add( new ElementClassFilter( typeof( ViewPlan ) ) );
    a.Add( new ElementClassFilter( typeof( ViewSection ) ) );

    LogicalOrFilter f = new LogicalOrFilter( a );

    FilteredElementCollector col
      = new FilteredElementCollector( doc )
        .WhereElementIsNotElementType()
        .WherePasses( f );

    foreach( View view in col )
    {
      //Debug.WriteLine( view.IsTemplate.ToString()
      // + ":" + view.ViewType.ToString() + ":"
      // + view.ViewName.ToString() + ":"
      // + view.Id.ToString() );

      Debug.Print(
        "{0} : {1} : {2} : {3}",
        view.IsTemplate, view.ViewType,
        view.ViewName, view.Id );

      if( !view.IsTemplate )
      {
        // . . .
      }
    }

    return Result.Succeeded;
  }
}
```

#### Delete Reference Planes Not Hosting Any Elements

A complex model can contain a pretty huge number of useless reference planes.

Nick proposed deleting all reference planes that

1. Have not been assigned a name, and- Do not host any elements.

To maximise performance, we use a parameter filter to check that the reference plane name is non-empty.
That is about twice as efficient as just filtering for all reference planes and then using LINQ or an explicit loop in .NET to check for a non-empty name.

Once we have found all the reference planes with a non-empty name, we need to check for each whether it hosts any elements.
This topic has been covered several times in the past, e.g. looking for
[generic object relationships](http://thebuildingcoder.typepad.com/blog/2010/03/object-relationships.html) (also
available for
[VB](http://thebuildingcoder.typepad.com/blog/2010/03/object-relationships-in-vb.html)).

The simplest approach that I am aware of is to delete it and check the number of element returned by the Delete method.
This list should include at least the element id of the reference plane itself, if the deletion succeeded.
If its length is larger than one, other elements were affected, which in this case probably means that they were hosted by the plane.

So, if the number of deleted elements equals one, we are happy and have deleted a reference plane hosting no other elements.
If the number is larger than one, it probably hosts something.
In that case, we can simply abort the transaction to undo the deletion attempt.

The deletion including the commit or rollback is achieved by the DeleteIfNotHosting method:
```csharp
bool DeleteIfNotHosting( ReferencePlane rp )
{
  bool rc = false;

  Document doc = rp.Document;

  Transaction tx = new Transaction( doc );

  tx.Start( "Delete ReferencePlane "
    + (++\_i).ToString() );

  // Deletion simply fails if the reference plane
  // hosts anything. If so, the return value ids
  // is null:

  ICollection<ElementId> ids = doc.Delete( rp );

  if( null == ids || 1 < ids.Count )
  {
    tx.RollBack();
  }
  else
  {
    tx.Commit();
    rc = true;
  }
  return rc;
}
```

If you look carefully, you'll notice that we number the transactions that we generate so that they can be differentiated in the user interface undo list.

Actually, during debugging, I noticed that the deletion simply fails if the reference plane is hosting any elements, so the approach that I originally envisaged is not working.
However, the failed deletion attempt returns a null collection of element ids instead of one containing more than one element.
I added the null check and roll back the transaction in that case also, and all is fine.

Since the deletion fails anyway for a reference plane hosting anything, we could just as well commit the transaction, I suppose, but I prefer to roll it back.
Given more time to play, I could check which is faster or uses less resources, rolling back or committing.
In AutoCAD ObjectARX, rolling back is significantly more expensive than committing, but I assume that is a special case.

This method is driven by the mainline Execute method which implements the parameter filtering for non-empty reference plane names:
```python
  public Result Execute(
    ExternalCommandData commandData,
    ref string message,
    ElementSet elements )
  {
    UIApplication uiapp = commandData.Application;
    UIDocument uidoc = uiapp.ActiveUIDocument;
    Application app = uiapp.Application;
    Document doc = uidoc.Document;

    // Construct a parameter filter to get only
    // unnamed reference planes, i.e. reference
    // planes whose name equals the empty string:

    BuiltInParameter bip
      = BuiltInParameter.DATUM\_TEXT;

    ParameterValueProvider provider
      = new ParameterValueProvider(
        new ElementId( bip ) );

    FilterStringRuleEvaluator evaluator
      = new FilterStringEquals();

    FilterStringRule rule = new FilterStringRule(
      provider, evaluator, "", false );

    ElementParameterFilter filter
      = new ElementParameterFilter( rule );

    FilteredElementCollector col
      = new FilteredElementCollector( doc )
        .OfClass( typeof( ReferencePlane ) )
        .WherePasses( filter );

    int n = 0;
    int nDeleted = 0;

    // No need to cast ... this is pretty nifty,
    // I find ... grab the elements as ReferencePlane
    // instances, since the filter guarantees that
    // only ReferencePlane instances are selected.

    foreach( ReferencePlane rp in col )
    {
      ++n;
      nDeleted += DeleteIfNotHosting( rp ) ? 1 : 0;
    }

    Debug.Print(
      "{0} unnamed reference plane{1} examined, {2} of them were deleted.",
      n, (1==n?"":"s"), nDeleted );

    return Result.Succeeded;
  }
}
```

Here is
 [containing
the source code, manifest file, and Visual Studio solution.

#### Automatic Setout Points

Why are we doing this?

It has been known to mankind for the following kind of situation to occur:

A large complex structure is constructed off-site.
When assembled on site, it does not fit.
The workers continue nonetheless and force it into place.
The error is discovered later and corrected using brute force and post adaption.
Overall cost of exercise: several days of delay and tens of thousands in cost overruns.

Here is Paul's description of this:

**The Challenge:** Provide a way to find the real world coordinates, for the corners of potential elements we want to set out in the field, automatically from the model.
In this example, concrete structural elements.
Ideally display these points in a Revit schedule, so that they can be formatted and placed on a drawing sheet.
Then define which of the points could be used as set out points list them in a schedule, preferably numbered sequentially for readability.

**In Practice:** Initially we could get the points listing, all from the API, but transferring this information into a schedule proved to be not possible.
Instead we created a family that contained shared parameters to hold the data we required for scheduling.
Then we place the created family at the points we have extracted from the geometry.
Through the API it is then possible to populate the parameters with the X, Y, Z coordinates we have extracted from the geometry, and then add these values to the extracted project location, to convert the values to real world coordinates.

Initially one type was used in the set out point family.
However this did not enable the ability to select which points we want as scheduled set out points.
For this two types were created within the family, SetoutPoint\_Major and SetoutPoint\_Minor.
This allowed two separate schedules to be created, one capturing all the points, another to capture points defined as major points for use on a drawing sheet for set out.
The 'all points' schedule is useful for export to a CSV file for import into a surveying device such as a
[Trimble](http://www.trimble.com)
allowing site set-out directly from the model.

To be continued...](zip/DeleteUnnamedNonHostingReferencePlanes.zip)