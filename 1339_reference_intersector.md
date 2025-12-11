---
ai_optimized: true
author: Jeremy Tammik
complexity_score: 4.9
content_type: code_example
optimization_date: '2025-12-11T11:44:15.803435'
original_url: https://thebuildingcoder.typepad.com/blog/1339_reference_intersector.html
post_number: '1339'
reading_time_minutes: 6
series: general
slug: reference_intersector
source_file: 1339_reference_intersector.htm
tags:
- csharp
- doors
- elements
- family
- filtering
- geometry
- python
- references
- revit-api
- views
- walls
- windows
title: Using ReferenceIntersector in Linked Files
word_count: 1274
---

### Using ReferenceIntersector in Linked Files

Quite a while ago, we had a Revit API discussion forum thread on the use of the
[ReferenceIntersector in linked files](http://forums.autodesk.com/t5/revit-api/referenceintersector/m-p/5493918).

Apparently, some restrictions on that have been removed in Revit 2016.

Before getting to that, let me mention that I have been very busy in the past few days in the
Revit API discussion forum, answering various cases, and on
The 3D Web Coder, implementing a cloud-based enhancement for the FireRating Revit SDK sample.

#### Steps Towards a Cloud-Based FireRating SDK Sample

I already mentioned some based aspects of the
[existing FireRating SDK sample](http://thebuildingcoder.typepad.com/blog/2015/07/firerating-and-the-revit-python-shell-in-the-cloud-as-web-servers.html#2) and my plans to
[reimplement it as a cloud-based app](http://thebuildingcoder.typepad.com/blog/2015/07/firerating-and-the-revit-python-shell-in-the-cloud-as-web-servers.html#4).

Here is an overview of my explorations so far:

- [My first mongo database](http://the3dwebcoder.typepad.com/blog/2015/06/my-first-mongo-database.html)

- Define the over-all goal and what to store, namely building projects, door instances hosted in them, and each door's fire rating value, based on the venerable old Revit SDK FireRating sample.

- [Implementing relationships](http://the3dwebcoder.typepad.com/blog/2015/07/implementing-mongo-database-relationships.html)

- Define a more complete schema that includes information about the container projects, i.e., the Revit RVT BIM or building information model project files.
- Define and maintain the relationships between the door family instances and their container projects.

- [Starting to Implement the FireRating REST API](http://the3dwebcoder.typepad.com/blog/2015/07/starting-to-implement-the-firerating-rest-api.html)

- Add a REST API to manage and query the database programmatically.

- [Put, Post, Delete and Curl Testing a REST API](http://the3dwebcoder.typepad.com/blog/2015/07/put-post-delete-and-curl-testing-the-firerating-rest-api.html)

- Adding a REST API version prefix.
- Testing the REST API using the browser and cURL.
- Implementing PUT, POST and DELETE.

- [Populating MongoDB via C# .NET REST API](http://the3dwebcoder.typepad.com/blog/2015/07/adding-a-mongodb-document-from-c-net-via-rest-api.html)

- Installing and running and the node.js web server on Windows
- Storing a document in MongoDB via the REST API from a C# .NET Revit add-in

#### Using the ReferenceIntersector in Linked Files

Now let's continue looking at using the ReferenceIntersector in linked files.

**Question:**
According to the API documentation and preliminary testing the ReferenceIntersector won't find any references in linked Revit files in Revit 2015.

So, for example, using the vector of a cable tray won't hit any linked walls... or so it seems.

But then why does it have a Boolean property named `FindReferencesInRevitLinks`?

Setting it 'true' doesn't seem to make any difference.

```csharp
  ReferenceIntersector refIntersector
    = new ReferenceIntersector( intersectFilter,
      FindReferenceTarget.Face, view3D );

  refIntersector.FindReferencesInRevitLinks = true;

  IList<ReferenceWithContext> referencesWithContext
    = refIntersector.Find( startPoint, rayDirection );
```

Has anyone succeeded in getting linked references using this method?

**Comment:**
I can confirm that using any of the following filters does not work in Revit 2015 when the elements are contained in linked files:

- ElementClassFilter
- ElementMulticlassFilter
- ElementCategoryFilter
- ElementMulticategoryFilter

When I say that these filters do not work, I mean that setting a class or category filter to look for walls in a linked file will not return any results from that link.

What does work, however, is creating a class or category filter that looks for a RevitLinkInstance.

So, what you need to do is use a RevitLinkInstance category or class filter (an inverted ElementIsElementType filter worked as well), and then filter the results yourself.

IMO, this is not how the filters should work when looking for elements in a linked file. To me, passing an ElementClassFilter looking for Wall objects to a ReferenceIntersector with FindReferencesInRevitLinks set to true should return walls whether they are in the local or in linked files. The documentation regarding all of this is quite sparse as well.

**Question Continued:**
What I did was to use a ElementSolidIntersectFilter.

So if I want to intersect a cable tray and a linked wall I first use the ElementSolidIntersectFilter on the link.

Done by passing the linked document into the filter.

This requires the link to have a 3D view which it may or may not have but thats ok.

ElementSolidIntersectFilter returns a collection of ElementId's for the linked intersected walls which I pass into ReferenceIntersector.

Then use the centerline of the cable tray as "intersector ray".

The advantage is that if a cable tray only partially intersects, that is if tray centerline falls outside the wall, I can still pick it up and handle it.

Otherwise I would need to project rays for every tray corner to catch this.

The only problem im having is that "FindReferenceTarget" is not working as expected.

FindReferenceTarget has 6 possible enumerations, All, Curve, Egde, Element, Face and Mesh.

When using "Face" I get 3 hits no matter which "FindReferenceTarget" option is used and have to rule out two of them.

What this function does in the end is to place what IFC refers to as IfcProvisionForVoid.

In Revit this is a Generic Model (solid) representing a hole for letting the tray through the wall.

I havent had time to work with the code lately but it works.

**Answer:**
Good news!

This ReferenceIntersector code that wasn’t working in Revit 2015 works great using the same test model in Revit 2016:

```csharp
public Dictionary<Reference, XYZ> GetIntersectPoints(
  Document doc,
  Element intersect )
{
  // Find a 3D view to use for the
  // ReferenceIntersector constructor.

  FilteredElementCollector collector
    = new FilteredElementCollector( doc );

  Func<View3D, bool> isNotTemplate = v3
    => !( v3.IsTemplate );

  View3D view3D = collector
    .OfClass( typeof( View3D ) )
    .Cast<View3D>()
    .First<View3D>( isNotTemplate );

  // Use location point as start point for intersector.

  LocationCurve lp = intersect.Location as LocationCurve;
  XYZ startPoint = lp.Curve.GetEndPoint( 0 ) as XYZ;
  XYZ endPoint = lp.Curve.GetEndPoint( 1 ) as XYZ;

  // Shoot intersector along element.

  XYZ rayDirection = endPoint.Subtract(
    startPoint ).Normalize();

  List<BuiltInCategory> builtInCats
    = new List<BuiltInCategory>();

  builtInCats.Add( BuiltInCategory.OST\_Roofs );
  builtInCats.Add( BuiltInCategory.OST\_Ceilings );
  builtInCats.Add( BuiltInCategory.OST\_Floors );
  builtInCats.Add( BuiltInCategory.OST\_Walls );

  ElementMulticategoryFilter intersectFilter
    = new ElementMulticategoryFilter( builtInCats );

  ReferenceIntersector refIntersector
    = new ReferenceIntersector( intersectFilter,
      FindReferenceTarget.Element, view3D );

  refIntersector.FindReferencesInRevitLinks = true;

  IList<ReferenceWithContext> referencesWithContext
    = refIntersector.Find( startPoint,
      rayDirection );

  IList<XYZ> intersectPoints = new List<XYZ>();

  IList<Reference> intersectRefs
    = new List<Reference>();

  Dictionary<Reference, XYZ> dictProvisionForVoidRefs
    = new Dictionary<Reference, XYZ>();

  foreach( ReferenceWithContext r in
    referencesWithContext )
  {
    dictProvisionForVoidRefs.Add( r.GetReference(),
      r.GetReference().GlobalPoint );
  }
  return dictProvisionForVoidRefs;
}
```

**Addendum:**
I have been using this successfully for a while now.

With perpendicular and rectangular geometry, the bounding box from ElementIntersectSolidFilter is usually sufficient.

I place an unhosted ProvisionForVoid family using the solid intersection bounding box.

I only trigger a ReferenceIntersector when the required action is to place a face or workplane based family on a face, and usually when the geometry is sloped or circular, like a pipe, circular duct or sloped wall or ceiling.

ReferenceIntersector doesn’t seem to return a face or anything else workplane related that can be used to place a family.

Therefore, right now, I iterate the reference geometry to achieve that: I retrieve the face with zero proximity from the intersector hit point and place the face-based family on that.

Logically, though, if the returned ReferenceTarget is a Face, I should be able to cast it as such, thereby avoiding the need to iterate geometry faces.

I need to look into that some more, and see if there are other suitable overloads of the NewFamilyInstance method that take reference face arguments.