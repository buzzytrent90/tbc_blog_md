---
ai_optimized: true
author: Jeremy Tammik
complexity_score: 3.1
content_type: qa
optimization_date: '2025-12-11T11:44:15.782551'
original_url: https://thebuildingcoder.typepad.com/blog/1328_transform_point.html
post_number: '1328'
reading_time_minutes: 4
series: geometry
slug: transform_point
source_file: 1328_transform_point.htm
tags:
- csharp
- family
- geometry
- revit-api
title: Create Duct, Pipe and Point Transform
word_count: 885
---

### Create Duct, Pipe and Point Transform

Our topics for today are on creating ducts, pipes and transforming a point:

- [Creating specific ducts and pipes](#2)
- [How to use Transform.CreateRotationAtPoint](#3)

I am writing this from the village Agii Apostoli on the east side of the Greek island of
[Euboea](https://en.wikipedia.org/wiki/Euboea),
on my way to the second
[I love 3D – Athens](http://www.meetup.com/I-love-3D-Athens) meetup
on June 5, followed by the
[AngelHack hackathon](http://angelhack.com/hackathon/athens-2015) on June 6-7.
For more details, please refer to
[The 3D Web Coder](http://the3dwebcoder.typepad.com/blog/2015/06/athens-angelhack-hackathon-and-nodejs-rest-workshop.html#2).

#### Creating Specific Ducts and Pipes

**Question:**
I am just getting started working with the Revit MEP API.

I would like to create a 'normal' pipe or duct, or, more specifically, use a fitting to represent a piece of pipe.

Is this possible?

None of the existing duct or pipe fittings that I can find seem usable for this purpose.

**Answer:**
In Revit, pipes and ducts make use of built-in system types.

You cannot use standard family definitions read in from external sources such as RFA files.

Of course, you can duplicate and modify the system types to adapt them for your needs.

You can certainly implement your own family to represent a straight piece of piping or ductwork, and probably even make it work, to a certain extent.

However: That would probably be a really bad idea.

You should probably make use of the built-in system pipe and duct types, or you will be working against the system, fighting it, instead of working with it.

You can modify the pipe and duct type properties to match your needs, you know.

If you don't like the geometry that Revit uses to represent it, you can even implement your own alternative geometry representation using direct shapes.

Are you sure you have fully explored the Revit MEP optimal workflow and best practices?

For an example of creating complete connected systems of pipes, ducts and fittings, please refer to the AutoRoute and AvoidObstruction Revit SDK samples.

They have been mentioned by The Building Coder numerous times, mostly in connection with other related samples at the same time.

One example that I spent quite a lot of time researching and implementing fully is the rolling offset:

- [Calculating a Rolling Offset Between Two Pipes](http://thebuildingcoder.typepad.com/blog/2014/01/calculating-a-rolling-offset-between-two-pipes.html)
- [Creating a Rolling Offset Pipe Between Two Pipes](http://thebuildingcoder.typepad.com/blog/2014/01/creating-a-rolling-offset-pipe-between-two-pipes.html)
- [Connecting the Rolling Offset Pipe to its Neighbour Pipes](http://thebuildingcoder.typepad.com/blog/2014/01/connecting-the-rolling-offset-pipe-to-its-neighbour-pipes.html)
- [Explicitly Placing Rolling Offset Elbow Fittings](http://thebuildingcoder.typepad.com/blog/2014/01/explicitly-placing-rolling-offset-pipe-elbow-fittings.html)

That should show you all there is to know about this topic.

All that I know, anyway.

I hope this helps.

Good luck making the right choices!

#### How to use Transform.CreateRotationAtPoint

Raised by Redsky in the Revit API discussion forum thread on
[figuring out how Transform.CreateRotationAtPoint works](http://forums.autodesk.com/t5/revit-api/cannot-figure-out-how-the-transform-createrotationatpoint-works/m-p/5663052):

**Question:**
I am trying to take an XYZ point and rotate it by 1 Pi (180 degrees) around another base point.
I think I got the correct method but I cannot figure out how to use it.
See this image:

![Rotate a point around a base point](img/create_rotation_at_point.png)

Is the base point equal to the XYZ Origin? Is the base point considered the origin? I think so...

How do I define the AXIS using an XYZ? Does anyone have an image to explain this? I couldn't find any documentation. I want to rotate the point along the XY plane (in plan orientation).

The angle is in Radians, I got that one.

```csharp
  Transform t = Transform.CreateRotationAtPoint(
    new XYZ( Origin.X, Origin.Y, Origin.Z + 1 ),
    Math.Pi, Origin );

  t.whatMethodHereToGetNewXYZLocationOfPoint?
```

Once I get the transform, how do I get the new XYZ location of my rotated point?

Thanks.

**Answer:**
The Transform.CreateRotation method creates a rotation transformation given just two arguments, the rotation axis and angle.

CreateRotationAtPoint takes three arguments: the axis, rotation angle and base point.

Here is one short description of
[how not to use CreateRotationAtPoint](http://thebuildingcoder.typepad.com/blog/2014/02/different-revit-api-aspects-and-features.html#5.2)   :-)

To define an axis by an XYZ, simply consider the XYZ a vector instead of a point.

Your attempt to define the axis looks like a mixture between a point and a vector to me; you have added the axis you need, the Z axis vector, to the base point. Good try, but no cigar.

In your case, to determine the rotation of the point pOld to the result pNew by 180 degrees around the base point base\_point in the XY plane, i.e., around the Z axis, you can do this:

```csharp
  XYZ axis = XYZ.BasisZ;
  double angle = Math.PI;
  Transform t = Transform.CreateRotationAtPoint(
    axis, angle, base\_point );
  XYZ pNew = t.OfPoint( pOld );
```