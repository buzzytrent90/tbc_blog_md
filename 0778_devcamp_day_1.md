---
ai_optimized: true
author: Jeremy Tammik
complexity_score: 5.8
content_type: code_example
optimization_date: '2025-12-11T11:44:14.573155'
original_url: https://thebuildingcoder.typepad.com/blog/0778_devcamp_day_1.html
post_number: 0778
reading_time_minutes: 8
series: general
slug: devcamp_day_1
source_file: 0778_devcamp_day_1.htm
tags:
- doors
- elements
- family
- filtering
- geometry
- python
- references
- revit-api
- rooms
- selection
- transactions
- views
- walls
title: DevCamp Day One
word_count: 1664
---

### DevCamp Day One

The first thing that happened here at DevCamp was that I was personally really deeply touched and honoured.

The conference was opened by Jim Quanci, director of ADN, the Autodesk Developer Network.
He asked all of us twenty-five-plus Autodesk employees up onto stage and introduced us one by one.
When the turn came to your humble author (or not so humble, after all), he asked all readers of this blog to raise their hands.
Almost every person in the room did.
Deep thanks to all DevCamp participants and to all you readers out there for your appreciation and support; it really helps motivate and improve the work of us all.
Grazie.

Jim's introduction wound up with some impressive cloud and mobile demos.

Adam presented his Revit 3D model cloud uploader and mobile device viewer, which I definitely want to write a separate post on as soon as possible, really soon now!
Come on Adam, let's do it!

Augusto presented his cloud based cost per unit for AEC materials demo.

Nicolas Magnon gave the keynote speech with interesting insights on the AEC market and current Autodesk developments and focus areas.

Some of the questions and answers to Nicolas were really interesting, and probably not public, even though this is a public event, as far as I know.
I don't even dare mention the two main Revit API topics that were discussed.
You'd have to be here, man.

After that we moved on to the various lectures, split into the following five parallel tracks:

- Revit Beginner- Revit Expert- Infrastructure and Other- Cloud/Mobile and Other- Business and Other

I started off presenting the first session in the expert track, and then was able to hang around in the same room to attend the interesting presentations of members of the Revit API development team, going through the following topics, in chronological order:

- [2-8 Extensible storage](#2)- [2-5 Materials, physical properties and compound structure](#3)- [2-6 Geometry API](#4)- [2-4 Core concepts](#5)

#### Extensible Storage

The first session in the Revit expert track was on extensible storage.

I presented a
[class](http://au.autodesk.com/?nd=event_class&session_id=9263&jid=1725932) and a
[lab](http://au.autodesk.com/?nd=event_class&session_id=9726&jid=1725932) on
this topic at Autodesk University 2011, as mentioned in the overview of the AU 2011
[Revit and AEC API sessions](http://thebuildingcoder.typepad.com/blog/2011/09/revit-and-aec-api-classes-at-autodesk-university.html).

I never got around to writing about this class in much detail here on the blog, mainly because the AU handout is so good and complete that there was never any need for it.

Here is an overview of the discussions related to this area that I published here anyway so far:

- [Extensible storage](http://thebuildingcoder.typepad.com/blog/2011/04/extensible-storage.html)- [Estorage of a map](http://thebuildingcoder.typepad.com/blog/2011/05/extensible-storage-of-a-map.html) or dictionary- [Estorage features](http://thebuildingcoder.typepad.com/blog/2011/06/extensible-storage-features.html) a collection of questions and answers- [AU 2011 Estorage materials](http://thebuildingcoder.typepad.com/blog/2012/03/melbourne-day-two.html#3), handouts and sample code- [Project Wide Data Storage](http://thebuildingcoder.typepad.com/blog/2012/05/devblog-devcamp-element-and-project-wide-data.html#4)- [DataStorage Element](http://thebuildingcoder.typepad.com/blog/2012/05/devblog-devcamp-element-and-project-wide-data.html#5)
            - [Simple DataStorage Sample](http://thebuildingcoder.typepad.com/blog/2012/05/devblog-devcamp-element-and-project-wide-data.html#6)- [Identifying DataStorage Elements](http://thebuildingcoder.typepad.com/blog/2012/05/devblog-devcamp-element-and-project-wide-data.html#7)

Anyway, the first session here at AEC DevCamp was based on that AU class, so I now updated the sample code and presentation for Revit 2013.

The migration was done in ten minutes after I got up this morning.
That also gave me a chance to admire the beautiful sunrise before the sun disappeared again up into the clouds above.

As in previous migrations, I simply updated the Revit API assembly references from 2012 to 2013, bumped the .NET framework version from 3.5 to 4, and replaced the obsolete Document Element property usages via the get\_Element method to its new GetElement replacement.

There was one single further issue I ran into, in the wall face selection filter, which was using the obsolete Reference GeometryObject property.
I fixed that like this:
```python
/// <summary>
/// Selection filter allowing only wall elements.
/// </summary>
class WallFilter : ISelectionFilter
{
  public bool AllowElement( Element e )
  {
    return e is Wall;
  }

  public bool AllowReference( Reference r, XYZ p )
  {
    //return r.GeometryObject is Face; // 2012

    return r.ElementReferenceType == // 2013
      ElementReferenceType.REFERENCE\_TYPE\_SURFACE;
  }
}
```

After migrating the add-in, I also rearranged it slightly.
I renamed the About command and added an external command Cmd\_7\_DataStorage to exercise the new Revit 2013 DataStorage element, based on
[Victor's
DataStorage sample](http://thebuildingcoder.typepad.com/blog/2012/05/devblog-devcamp-element-and-project-wide-data.html#6).

Here is the list of the commands now provided by this add-in, or rather a list of its project files:
![Estorage project](img/Estorage_2013_project.png)

It is worthwhile pointing out that the external application implementation is neat, since it generates the ribbon panel and the entries for the individual commands in a loop.
Here is the resulting panel:
![Estorage commands](img/Estorage_2013_commands.png)

The About box implementation is also neat, since it reads the information it displays from the add-in assembly DLL attributes instead of duplicating that information:
![Estorage About box](img/Estorage_2013_about.png)

Here is
[Estorage\_2013.zip](zip/Estorage_2013.zip) containing
the full updated sample application including its add-in manifest, Visual Studio solution and full source.
For good measure, here is the updated
[slide deck](file:///C:/a/j/adn/devcamp/2012/doc/2-8_estorage.pptx) as
well.

#### Materials, Physical Properties and Compound Structure

After my own session on extensible storage, I could stay put right there in the same room to join the one by Steven Mycynek on 'Revit Materials, Physical Properties and Compound Structure API Basics', with the following agenda:

- Material Model History- Material Properties- Dealing with Units- Working with Materials- GbXMLs role in Revit- Family Thermal Properties- Layered Assemblies- Layered Assembly Thermal Properties- A few last enhancements

From here on, my notes are very fragmentary.
Hopefully, you will be able to access the DevCamp materials on the internet quite soon.
At that point, these notes might help you decide what to explore.
Actually, I added a
[snapshot of the materials](#6)
in their current state below, so please refer to those for more details.

Revit 2011 material inheritance model, in 2012 property set, obsolete material subclasses.

Revit 2013 material asset model, material subclasses removed.

Why? Extensibility, add new assets instead of deriving.

Airmax, consolidate all material handling across AutoCAD, Inventor, Revit, all Autodesk products to share same material library.

Material property hierarchy, PropertySetElement, StructuralAsset and ThermalAsset, discoverable properties, new in 2013.

Asset types and properties.

Units: length in feet, all other units in SI units, making compound units a bit confusing.

UnitUtility.cs Steve's personal sample code...

ExporterIFCUtils.ConvertUnits...

There is no official API from display units back to system units, but Steve's sample includes the method DisplayUnitsToSystem.

FormatUnitValue<ValueType>...

Using ProjectUnits.get\_FormatOptions( unitType )

Duplicating a material, a deep copy, including all assets; creating one from scratch.

FamilyThermalProperties...

#### Geometry API

After that, I attended Scott Conover's session on 'Geometry API in Autodesk Revit'.
It is a continuation of his AU 2011 presentation
[CP4011](http://au.autodesk.com/?nd=event_class&session_id=9124&jid=1725932) 'Geometric Progression: Further Analysis of Geometry Using the Autodesk Revit 2012 API',
enhanced for the Revit 2013 API.

In consists of two main sections, on geometry extraction fundamentals and tools, and support you in the following areas:

- Extract and analyze the geometry of existing Revit elements- Create and manipulate temporary curve and solid geometry- Find elements by 3D intersection- Find elements by ray projection and filtering- Apply an ExtrusionAnalyzer to geometry- Utilize parts to analyze geometry of HostObjects and their layers- Extract and analyze the boundary geometry of rooms and spaces- Analyze the geometry of point clouds

More details are available on the developer guide wiki, and examples of use of all the tools discussed are provided in the sample materials.

Display solids on the graphics screen marking the areas around doors that must not be obstructed in order to ensure safe fire or other emergency egress.

[ReferenceIntersector](http://thebuildingcoder.typepad.com/blog/2012/03/revit-2013-and-its-api.html) class...

Point cloud geometry analysis...

#### Hashing Instead of Core Concepts

The last session of the day in the Revit expert track was Arnošt Löbel's class on the Core Revit API Frameworks, covering the following topics:

- Regeneration modes- Transaction modes- Transaction phases- Document modifiability- Element validity- Object lifespan- API events- External commands- API callbacks- Dynamic updaters- API Scopes- API Firewall

I would have loved to attend, but was unfortunately forced to flee, breathe some fresh air, escape the so-called air conditioning, go running, join the hashers, the
[Boston Hash House Harriers](http://bostonhash.com),
the "drinking club with a running problem", that Michael Priestman introduced me to
[two years back](http://thebuildingcoder.typepad.com/blog/2010/06/hashing-and-devlabs.html).
Once again, it was great fun.

#### Snapshot of Materials

Here is a snapshot of the materials of today's four Revit API expert sessions:

- [2-4 Core frameworks](devcamp_2012_2-4_Core_Revit_API_Frameworks.pdf)- [2-5 Materials, physical properties and compound structure](devcamp_2012_2-5_Materials_and_Assemblies.pptx)- [2-5 Materials sample code](devcamp_2012_2-5_PropertyUtilityApp.zip)- [2-6 Geometry API](devcamp_2012_2-6_Geometry_API.pptx)- [2-6 Geometry sample code](devcamp_2012_2-6_Geometry_API_Sample_code.zip)- [2-8 Extensible storage](devcamp_2012_2-6_Geometry_API.pptx)- [2-8 Estorage sample code](Estorage_2013.zip)

Please note that this is just a snapshot.
Soon after the conference completes, the complete and final materials will be published.