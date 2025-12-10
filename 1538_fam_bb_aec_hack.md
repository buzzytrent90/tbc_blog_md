---
post_number: "1538"
title: "Fam Bb Aec Hack"
slug: "fam_bb_aec_hack"
author: "Jeremy Tammik"
tags: ['elements', 'family', 'filtering', 'levels', 'revit-api', 'rooms', 'sheets', 'views']
source_file: "1538_fam_bb_aec_hack.md"
original_url: "https://thebuildingcoder.typepad.com/blog/1538_fam_bb_aec_hack.html"
---

### Family Bounding Box and AEC Hackathon Munich
I briefly mentioned
the [AEC Hackathon in Munich](http://thebuildingcoder.typepad.com/blog/2017/03/events-uv-coordinates-and-rooms-on-level.html#3) yesterday.
Here is some more information on that, highlighting the exciting speaker line-up and target topics, plus a solution for determining the bounding box of an entire family and a suggestion to implement a continuous integration service for RevitLookup:
- [AEC Hackathon Munich Topics and Speakers](#2)
- [Family bounding box](#3)
- [Continuous integration for RevitLookup?](#4)
#### AEC Hackathon Munich Topics and Speakers
The AEC Hackathon is coming to Germany, March 31 - April 2, at the Technical University in Munich, Germany.
I am unable to attend, unfortunately, and will miss it sorely.
It brings a weekend of geeking at its finest.
It gives those designing, building and maintaining our built environment, the opportunity to engage, collaborate and interact on multiple aspects of the building industry.
Not all built environment professionals have to code or have much background with tech.
Just come with your knowledge, an open mind, collaborative spirit, and willingness to solve Mon-Fri problems leveraging various kinds of technology.
We've shaped an exhilarating Program on Robotics, Generative Design, IOT, AR/VR and web services for you!
Check out some of our Speakers:
- [Matt Jezyk](http://aechackathon-germany.de/speaker/matt-jezyk/), Senior Product Line Manager, AEC Conceptual Design Products, Autodesk (USA)
- [Jaime Ronsales](http://aechackathon-germany.de/speaker/chelsie/), Senior Developer Consultant [FORGE](https://forge.autodesk.com/), Autodesk (USA)
- [Maximilian Thumfart](http://aechackathon-germany.de/speaker/maximilian-thumfart/), Developer
of [Grevit](http://grevit-dev.github.io/Grevit/), [Dynamo for Rebar](https://github.com/tt-acm/DynamoForRebar),
[Dynamo PDF](https://github.com/grevit-dev/DynamoPDF) (UK)
- [Robin-Manuel Thiel](http://aechackathon-germany.de/speaker/robin-manuel-thiel/) and [Malte Lantin](http://aechackathon-germany.de/speaker/malte-lantin/), Technical Evangelists, Microsoft Developer Experience (GER)
- [Sigrid Brell Cokcan](http://aechackathon-germany.de/speaker/robots-in-architecture/), IP/RWTH Aachen and [Johannes Braumann](http://aechackathon-germany.de/speaker/robots-in-architecture/), UFG Linz – [Robots in Architecture](http://www.robotsinarchitecture.org/)
- And many more...
Why you should not miss this Event:
- Test out new cutting edge Technologies
- Connect with leading BIM Experts, Developers and Students
- Solve Real World Industry Problems
Did we get you excited? If you like to learn more about the Event have a look at our website:
[www.aechackathon-germany.de](http://www.aechackathon-germany.de/)
How can you Engage in the Event?
1. Come on by
Join us for the weekend, observe, and offer your industry expertise to teams as needed. Encourage others in your company to join you as this is a great way to get exposed to new technologies and meet other innovators in top AEC firms.
è Register
2. Team up and solve an Industry Problems:
Bring a problem you want to solve or company/industry challenge. The Friday Night Lightning Rounds are open to all that want to propose an idea and form a team around improving some element of the design, build, operational process. Past teams have created high-tech solutions for tracking tools, high End BIM Solutions, using virtual reality for jobsite safety training, and much more. Visit our YouTube channel to watch past team presentation.
3. Learn how to develop on Innovative Software Platforms:
If you are an IT affinitive User or a Software Developer and you always wanted to learn how to develop on FORGE, Dynamo Studio, KUKA Robot, Microsoft Mixed Reality, IoT Hub, Azure Functions and Cognitive Services come and join our free Developer Workshops in the afternoon before the Event.
Take a look at our [Prep Workshops](http://aechackathon-germany.de/dev-prep-workshops/).
If you have any Questions, please have a look at the [Frequently asked Questions](http://aechackathon-germany.de/faq/).
![AEC Hackathon at TU Munich](img/2017_aec_hack_tu_muc.jpg)
#### Family Bounding Box
Kevin [@kelau1993](https://forums.autodesk.com/t5/user/viewprofilepage/user-id/4517553) Lau shared a solution to
determine the bounding box of a family in the family document environment in
the [Revit API discussion forum](http://forums.autodesk.com/t5/revit-api-forum/bd-p/160) thread
on [family bounding box in family document](https://forums.autodesk.com/t5/revit-api-forum/family-boundingbox-in-family-document/m-p/6946049).
It looks very cool indeed to me, so I went ahead and created a complete Visual Studio project
and [FamilyBoundingBox GitHub repository](https://github.com/jeremytammik/FamilyBoundingBox) for it to make it more accessible.
Here is Kevin's code, taken from
the [FamilyBoundingBox Command.cs module](https://github.com/jeremytammik/FamilyBoundingBox/blob/master/FamilyBoundingBox/Command.cs):
```csharp
///
/// Computes the effective 'BoundingBoxXYZ' of the
/// whole family, including all combined and
/// individual forms.
/// summary>
/// The Revit document that
/// contains the family.param>
/// The effective 'BoundingBoxXYZ' of the
/// whole family.returns>
private static BoundingBoxXYZ ComputeFamilyBoundingBoxXyz(
Document document )
{
BoundingBoxXYZ familyBoundingBoxXyz = null;
HashSet genericFormExclusionSet
= new HashSet();
familyBoundingBoxXyz
= MergeGeomCombinationBoundingBoxXyz( document,
familyBoundingBoxXyz,
genericFormExclusionSet );
familyBoundingBoxXyz
= MergeSolidBoundingBoxXyz( document,
familyBoundingBoxXyz,
genericFormExclusionSet );
return familyBoundingBoxXyz;
}
///
/// Merge  with the
/// 'BoundingBoxXYZ's of all 'GeomCombination's in
///  into a new 'BoundingBoxXYZ'.
/// Collect all members of the 'GeomCombination's
/// found into .
/// summary>
/// The Revit 'Document' to search
/// for all 'GeomCombination's.param>
/// The 'BoundingBoxXYZ' to merge with.param>
/// A 'HashSet' to collect all of the
/// 'GeomCombination' members that form the 'GeomCombination'.param>
/// The new merged 'BoundingBoxXYZ' of
///  and all 'GeomCombination's
/// in returns>
private static BoundingBoxXYZ MergeGeomCombinationBoundingBoxXyz(
Document document,
BoundingBoxXYZ boundingBoxXyz,
HashSet geomCombinationMembers )
{
BoundingBoxXYZ mergedResult = boundingBoxXyz;
FilteredElementCollector geomCombinationCollector
= new FilteredElementCollector( document )
.OfClass( typeof( GeomCombination ) );
foreach( GeomCombination geomCombination in geomCombinationCollector )
{
if( geomCombinationMembers != null )
{
foreach( GenericForm genericForm in geomCombination.AllMembers )
{
geomCombinationMembers.Add( genericForm.Id );
}
}
BoundingBoxXYZ geomCombinationBoundingBox
= geomCombination.get_BoundingBox( null );
if( mergedResult == null )
{
mergedResult = new BoundingBoxXYZ();
mergedResult.Min = geomCombinationBoundingBox.Min;
mergedResult.Max = geomCombinationBoundingBox.Max;
continue;
}
mergedResult = MergeBoundingBoxXyz(
mergedResult, geomCombinationBoundingBox );
}
return mergedResult;
}
///
/// Merge  with the 'BoundingBoxXYZ's of
/// all 'GenericForm's in  that are solid into
/// a new 'BoundingBoxXYZ'.
/// Exclude all 'GenericForm's in
///  from being found in
/// .
/// summary>
/// The Revit 'Document' to search for all
/// 'GenericForm's excluding the ones in
/// .param>
/// The 'BoundingBoxXYZ' to merge with.param>
/// A 'HashSet' of all the
/// 'GenericForm's to exclude from being merged with in
/// .param>
/// The new merged 'BoundingBoxXYZ' of
///  and all 'GenericForm's excluding
/// the ones in
/// in returns>
private static BoundingBoxXYZ MergeSolidBoundingBoxXyz(
Document document,
BoundingBoxXYZ boundingBoxXyz,
HashSet genericFormExclusionSet )
{
BoundingBoxXYZ mergedResult = boundingBoxXyz;
FilteredElementCollector genericFormCollector
= new FilteredElementCollector( document )
.OfClass( typeof( GenericForm ) );
if( genericFormExclusionSet != null
&& genericFormExclusionSet.Any() )
{
genericFormCollector.Excluding(
genericFormExclusionSet );
}
foreach( GenericForm solid in
genericFormCollector
.Cast()
.Where( genericForm => genericForm.IsSolid ) )
{
BoundingBoxXYZ solidBoundingBox
= solid.get_BoundingBox( null );
if( mergedResult == null )
{
mergedResult = new BoundingBoxXYZ();
mergedResult.Min = solidBoundingBox.Min;
mergedResult.Max = solidBoundingBox.Max;
continue;
}
mergedResult = MergeBoundingBoxXyz(
mergedResult, solidBoundingBox );
}
return mergedResult;
}
///
/// Merge  and
///  into a new 'BoundingBoxXYZ'.
/// summary>
/// A 'BoundingBoxXYZ' to mergeparam>
/// A 'BoundingBoxXYZ' to mergeparam>
/// The new merged 'BoundingBoxXYZ'.returns>
static BoundingBoxXYZ MergeBoundingBoxXyz(
BoundingBoxXYZ boundingBoxXyz0,
BoundingBoxXYZ boundingBoxXyz1 )
{
BoundingBoxXYZ mergedResult = new BoundingBoxXYZ();
mergedResult.Min = new XYZ(
Math.Min( boundingBoxXyz0.Min.X, boundingBoxXyz1.Min.X ),
Math.Min( boundingBoxXyz0.Min.Y, boundingBoxXyz1.Min.Y ),
Math.Min( boundingBoxXyz0.Min.Z, boundingBoxXyz1.Min.Z ) );
mergedResult.Max = new XYZ(
Math.Max( boundingBoxXyz0.Max.X, boundingBoxXyz1.Max.X ),
Math.Max( boundingBoxXyz0.Max.Y, boundingBoxXyz1.Max.Y ),
Math.Max( boundingBoxXyz0.Max.Z, boundingBoxXyz1.Max.Z ) );
return mergedResult;
}
```
#### Continuous integration for RevitLookup?
Peter Hirn of [Build Informed GmbH](https://www.buildinformed.com) very kindly offered to implement
a [continuous integration](https://en.wikipedia.org/wiki/Continuous_integration) service
for [RevitLookup](https://github.com/jeremytammik/RevitLookup), similar to the daily builds provided
for [Dynamo](http://www.dynamobim.org/)
at [dynamobuilds.com](http://dynamobuilds.com/).
He raised
the [Revit API discussion forum](http://forums.autodesk.com/t5/revit-api-forum/bd-p/160) thread
on [CI for RevitLookup](https://forums.autodesk.com/t5/revit-api-forum/ci-for-revit-lookup/m-p/6947111) to
discuss the issue.
> We're considering to set up a public CI for [RevitLookup](https://github.com/jeremytammik/RevitLookup).
> The output could be something like the [Dynamo builds](http://dynamobuilds.com).
> I'm thinking [Travis](https://travis-ci.org/) or [Jenkins](https://jenkins.io/) for the builds.
> We would provide the infrastructure and maintenance for this service.
> I'd love to get your feedback on this idea.
If you have any thought or suggestion on this, please participate in
the [CI for RevitLookup discussion](https://forums.autodesk.com/t5/revit-api-forum/ci-for-revit-lookup/m-p/6947111) and
let us know.
Thank you!