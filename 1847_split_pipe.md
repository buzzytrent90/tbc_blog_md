---
ai_optimized: true
author: Jeremy Tammik
complexity_score: 6.3
content_type: qa
optimization_date: '2025-12-11T11:44:16.881814'
original_url: https://thebuildingcoder.typepad.com/blog/1847_split_pipe.html
post_number: '1847'
reading_time_minutes: 8
series: mep
slug: split_pipe
source_file: 1847_split_pipe.md
tags:
- elements
- family
- filtering
- levels
- parameters
- revit-api
- sheets
- transactions
- views
- mep
title: Split Pipe
word_count: 1621
---

### Split Pipe Tutorial and Related Cases
Today, let's focus on splitting pipes and other things, starting with a nicely structured tutorial shared
by Abdelaziz [Zizobiko25](https://github.com/Zizobiko25) Fadoul:
- [Abdelaziz' split pipe tutorial](#3)
- [Calling the SL split element command](#4)
- [Splitting a conduit](#5)
- [George Floyd in Memoriam](#6)
#### Abdelaziz' Split Pipe Tutorial
Hi folks,
Hope you are all safe and keeping well.
As there is no explicit SDK sample for splitting pipe instances into standard lengths without using the fabrication configurations, so I thought to share with you the below snippet. This is built upon a contribution that I came across on this platform, and uses functions that are developed on other samples (e.g. Find connected element, etc.), however, I thought to enhance it and explain the steps to achieve such a task. The fundamental idea for this is to delete the selected pipe and replace it with the creation of 2 pipes instead.
1- Establish a new transaction: As this is going to modify the Revit document, then we need to have it done within a transaction context.
```csharp
using( Transaction tx = new Transaction( activeDoc.Document ) )
{
tx.Start( "split pipe" );
ElementId systemtype = system.GetTypeId();
SplitPipe( pipes[ 0 ], system, activeDoc, systemtype, pipeType );
tx.Commit();
}
```
2- We then obtain the selected pipe basic data including its length, associated level Id and system Id.
```csharp
ElementId levelId = segment.get_Parameter(
BuiltInParameter.RBS_START_LEVEL_PARAM )
.AsElementId();
// system.LevelId;
ElementId systemtype = _system.GetTypeId();
// selecting one pipe and taking its location.
Curve c1 = (segment.Location as LocationCurve).Curve;
//Pipe diameter
double pipeDia = UnitUtils.ConvertFromInternalUnits(
segment.get_Parameter( BuiltInParameter.RBS_PIPE_DIAMETER_PARAM ).AsDouble(),
DisplayUnitType.DUT_MILLIMETERS );
```
3- We then compare the obtained length, including the fitting length, with the standard length that we want to assign (for instance, 6m here).
If it is longer than that, then we proceed.
```csharp
//Standard length
double l = 6000;
//Coupling length
double fittinglength = (1.1 \* pipeDia + 14.4);
// finding the length of the selected pipe.
double len = UnitUtils.ConvertFromInternalUnits( segment.get_Parameter( BuiltInParameter.CURVE_ELEM_LENGTH ).AsDouble(), DisplayUnitType.DUT_MILLIMETERS );
if( len <= l )
return;
```
4- We then determine the splitting point by taking the fracture between the required length to the total length.
```csharp
var startPoint = c1.GetEndPoint( 0 );
var endPoint = c1.GetEndPoint( 1 );
XYZ splitpoint = (endPoint - startPoint) \* (l / len);
var newpoint = startPoint + splitpoint;
Pipe pp = segment as Pipe;
// Find two connectors which pipe's two ends connector connected to.
Connector startConn = FindConnectedTo( pp, startPoint );
Connector endConn = FindConnectedTo( pp, endPoint );
```
5- We are then able to create the first and second pipe segments using the command create:
```csharp
// creating first pipe
Pipe pipe = null;
if( null != _pipeType )
{
pipe = Pipe.Create( _activeDoc.Document,
_pipeType.Id, levelId, startConn, newpoint );
}
Connector conn1 = FindConnector( pipe, newpoint );
//Check + fitting
XYZ fittingend = (endPoint - startPoint)
\* ((l + (fittinglength / 2)) / len);
//New point after the fitting gap
var endOfFitting = startPoint + fittingend;
Pipe pipe1 = Pipe.Create( _activeDoc.Document,
systemtype, _pipeType.Id, levelId, endOfFitting,
endPoint );
// Copy parameters from previous pipe to the following Pipe.
CopyParameters( pipe, pipe1 );
Connector conn2 = FindConnector( pipe1, endOfFitting );
_ = _activeDoc.Document.Create.NewUnionFitting( conn1, conn2 );
if( null != endConn )
{
Connector pipeEndConn = FindConnector( pipe1, endPoint );
pipeEndConn.ConnectTo( endConn );
}
```
6- Once this is done successfully, we can then delete the original pipe segment
```csharp
ICollection deletedIdSet
= _activeDoc.Document.Delete( segment.Id );
if( 0 == deletedIdSet.Count )
{
throw new Exception( "Deleting the selected elements in Revit failed." );
}
```
Link to the full code is accessible from
the [Pipe-Split GitHub repository](https://github.com/Zizobiko25/Pipe-Split).
Hope this useful for someone. Give me a shout if you have any questions.
Thanks, Abdelaziz
Many thanks to \*You\*, Abdelaziz, for putting together and sharing this very nice and clean solution and helpful explanation!
#### Calling the SL Split Element Command
While on the topic of splitting things, here is another related old case, 14201186 \*Need to call command Split Element (SL) with defined filter and length\*:
\*\*Question:\*\* I wanted to call the Split Element (SL) command through API and wants to give the filter to select only Pipe with predefined length to split.
Can you please help us with this?
\*\*Answer:\*\* You can call many built-in Revit commands using
the [`PostCommand` API](http://thebuildingcoder.typepad.com/blog/about-the-author.html#5.3).
However, this just launches the built-in command as is, prompting the user for all further input and other interaction.
Therefore, if you wish to drive this completely automatically, it will be hard to make use of the built-in command.
The specific question of splitting a pipe into shorter segments of a given length has been discussed repeatedly in the Revit API discussion forum:
- [Split a pipe at a specific lenght](https://forums.autodesk.com/t5/revit-api-forum/split-a-pipe-at-a-specific-lenght/m-p/5533100)
- [Pipe / Duct spliting using the Revit API](https://forums.autodesk.com/t5/revit-api/pipe-duck-spliting-using-the-revit-api/m-p/3312303)
- [Split Ducts with Union](https://forums.autodesk.com/t5/revit-api/split-ducts-with-union/m-p/5489135)
#### Splitting a Conduit
Another more recent related case is 16561381 \*Programmatically Place Conduit Fitting\*, on splitting a conduit:
\*\*Question:\*\* We would like to know how to programmatically place a conduit fitting at a certain location on a piece of conduit.
Basically, we want to exactly what this tool does (screenshots included as well) but we do not want the user to have to click where they place it – we want to programmatically control it's placement on a piece of conduit.
![Place conduit fitting](img/place_conduit_fitting_1.png "Place conduit fitting")
![Place conduit fitting](img/place_conduit_fitting_2.png "Place conduit fitting")
We have been unable to find anything within the API or forums which accomplish this. Please advise on how this can be done.
Tool help info:
- [Add Conduit Fittings](https://help.autodesk.com/view/RVT/2019/ENU/?guid=GUID-12A049FB-F779-4184-97A9-2E700945DE66)
\*\*Answer:\*\* I believe that one possible approach to achieve this would be to place the fitting in the appropriate position in the model first using `NewFamilyInstance`, and then connect it to the neighbouring conduits.
I would expect them to adjust automatically.
I discovered and demonstrated that approach in the series of posts
on [implementing a rolling offset](http://thebuildingcoder.typepad.com/blog/2014/01/final-rolling-offset-using-pipecreate.html).
To make sure I am not missing anything, I am checking with the development team for you as well.
Meanwhile, here are some other discussions on placing similar fittings:
- [Cable Tray Orientation and Fittings](http://thebuildingcoder.typepad.com/blog/2010/05/cable-tray-orientation-and-fittings.html)
- [Use of NewTakeOffFitting on a Duct](http://thebuildingcoder.typepad.com/blog/2011/02/use-of-newtakeofffitting-on-a-duct.html)
- [Set Elbow Fitting Type](http://thebuildingcoder.typepad.com/blog/2011/02/set-elbow-fitting-type.html)
- [Use of NewTakeOffFitting on a Pipe](http://thebuildingcoder.typepad.com/blog/2011/04/use-of-newtakeofffitting-on-a-pipe.html)
- [Simpler Rolling Offset Using NewElbowFitting](http://thebuildingcoder.typepad.com/blog/2014/01/newelbowfitting-easily-places-rolling-offset-elbow-fittings.html)
- [NewCrossFitting Connection Order](http://thebuildingcoder.typepad.com/blog/2014/10/newcrossfitting-connection-order.html)
The development team added some more detailed advice:
You can use `Document.Create.NewElbowFitting` taking two `Connector` arguments.
Depending on the orientation of the MEP Curves it might fail and then require a Transition or a Union instead.
If there are three connectors, it requires a Tee; if there are four connectors, it requires a Cross.
It uses the family routing settings to assign the type and the `size_lookup` in the fitting family to determine the geometrical representation of the FamilyInstance.
So, in case of different diameters to connect, it will create other objects in between, such as transitions (e.g., for pipes or ducts) or a junction box (e.g., for conduits).
\*\*Response:\*\* We are already doing something similar, but it seems wildly inefficient, and I was hoping the API provided something we were missing as Pipe (and other curve based families) seem to have 'special' methods that conduit lacks.
[Trouble of my own implementation of BreakCurve on Conduit](https://forums.autodesk.com/t5/revit-api-forum/trouble-of-my-own-implementation-of-breakcurve-on-conduit/m-p/8426899)
To be sure, we did research this thoroughly but as noted above our implementation has a 'clunky' feel to it when compared to the ease of the fitting tool in the UI (although watching updater messages shows Revit may be handling it in a similar way we are). We are under a time crunch at the moment and hope this methodology will ease some of the 'heartburn' we are experiencing with fittings on sloped conduit as well.
\*\*Answer:\*\* Yes, it does indeed look as if the Revit API forum thread you pointed out covers it pretty well.
I'm afraid I have nothing constructive to add to that, and, as you say, Revit probably does something similar internally as well.
#### George Floyd in Memoriam
"It's my face man

I didn't do nothing serious man

please

please

please I can't breathe

please man

please somebod

please man

I can't breathe

I can't breathe

please

(inaudible)

man can't breathe, my face

just get u

I can't breathe

please (inaudible)

I can't breathe sh\*t

I will

I can't move

mama

mama

I can't

my knee

my nuts

I'm through

I'm through

I'm claustrophobic

my stomach hurt

my neck hurts

everything hurts

some water or something

please

please

I can't breathe officer

don't kill me

they gon' kill me man

come on man

I cannot breathe

I cannot breathe

they gon' kill me

they gon kill me

I can't breathe

I can't breathe

please sir

please

please

please I can't breathe"