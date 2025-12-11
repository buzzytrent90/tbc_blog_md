---
ai_optimized: true
author: Jeremy Tammik
complexity_score: 2.5
content_type: tutorial
optimization_date: '2025-12-11T11:44:17.065355'
original_url: https://thebuildingcoder.typepad.com/blog/1929_formit_quaternion.html
post_number: '1929'
reading_time_minutes: 5
series: general
slug: formit_quaternion
source_file: 1929_formit_quaternion.md
tags:
- family
- revit-api
- sheets
- views
title: Formit Quaternion
word_count: 966
---

### Installer Asset, FormIt and Quaternions
Notes on FormIt and its API, the new automatically generated RevitLookup installer asset, transformations and quaternions:
- [FormIt API and geographical context](#2)
- [RevitLookup MSI installer asset](#3)
- [Transform and quaternions](#4)
#### FormIt API and Geographical Context
Good things are happening
with [FormIt](https://formit.autodesk.com),
a multi-platform architectural modelling, conceptual design and analysis tool.
With FormIt, you can sketch, collaborate, analyse, and revise early-stage design concepts with BIM-based conceptual design.
Kean Walmsley took a closer look
at [FormIt and its JavaScript API](https://www.keanw.com/2021/11/autodesk-formit-and-its-javascript-api.html) and
describes in detail how to get started with FormIt plugins.
That prompted Radu [@radugidei](https://twitter.com/radugidei) Gidei
to [mention](https://twitter.com/radugidei/status/1458370952652378113?s=20)
the Matterlab FormIt plugin making use of this to provide geographical context in Revit;
> Nice one and great intro to FormIt API!
Look forward to the VASA integration, sounds very cool!
Btw, some of our team are working on some FormIt stuff as we speak, some cool things coming for the community! (cough docs cough)
- [Matterlab FormiIt 3D Context Creator](https://github.com/matterlab-co/FormIt-Context-Plugin)
![3D context creator](img/formit_3d_context_creator.png "3D context creator")
Thanks to Kean and Radu for sharing these!
#### RevitLookup MSI Installer Asset
Yet another update to RevitLookup brings us
to [release 2022.0.2.5](https://github.com/jeremytammik/RevitLookup/releases/tag/2022.0.2.5),
adding automatic generation of a release for the master branch and attaching the completed installer as an asset to the release:
![RevitLookup 2022.0.2.5](img/revitlookup_2022_0_2_5.png "RevitLookup 2022.0.2.5")
This is the result of [pull request #118 to release by GitAction](https://github.com/jeremytammik/RevitLookup/pull/118),
including an intensive and very instructive conversation
between [Roman @Nice3point](https://github.com/Nice3point)
and Luiz Henrique [@ricaun](https://github.com/ricaun) Cassettari
on how to optimally set it up, and a renewed summary by Roman on how to handle future pull requests:
> Once again, I will repeat the steps that you must take to publish:
- Developers send PR to the `dev` branch.
- We check, write a code review.
- Accept PR.
- Upgrade the build version in [`csproj` line 8](https://github.com/jeremytammik/RevitLookup/blob/dev/RevitLookup/RevitLookup.csproj#L8)
- Log changes in the [changelog](https://github.com/jeremytammik/RevitLookup/blob/dev/Doc/Changelog.md);
multiple lines are supported;
the main thing is that a line does not start with a hyphen '-';
that means the end of the description of the current release.
- Merge the `dev` branch into `master`.
- Release will be generated automatically.
Many thanks to Luiz Henrique and Roman for their deep discussion, insight and implementation!
#### Transform and Quaternions
In
the [Revit API discussion forum](http://forums.autodesk.com/t5/revit-api-forum/bd-p/160) thread
on [getting translation and rotation for a `FamilyInstance`](https://forums.autodesk.com/t5/revit-api-forum/get-translation-and-rotation-for-a-familyinstance-export/m-p/10758975),
Matthew [mhannonQ65N2](https://forums.autodesk.com/t5/user/viewprofilepage/user-id/8377999) Hannon
shares a very nice and succinct explanation of quaternions and how they relate to Revit transformations:
For rotations, what you are trying to do is transform between two different representations of the 3d rotation group (aka the Special Orthogonal group of dimension 3, SO(3)). Revit provides rotations in the form of a 3x3 matrix whose three columns are the BasisX, BasisY, and BasisZ properties of Transform. This rotates vectors by standard matrix multiplication.
However, what you are using in SharpGLTF requires a quaternion. To be precise, it actually requires a unit quaternion, which is a quaternion of length 1. The equation for this is *x²+y²+z²+w²=1* (this is identical to the equation of the unit sphere in 4-dimensional space, known as the 3-sphere because its 'surface' is 3-dimensional). Furthermore, the product of two unit quaternions is also a unit quaternion. Unit quaternions rotate 3d vectors in a more mathematically complicated way that is actually faster to compute (I won't get into the details). A significant consequence of how unit quaternions are used to rotate vectors is that multiplying the unit quaternion by *-1* does not change how the vector is rotated. As such each 3d rotation can be represented by two different quaternions, *q*, and *-q*. As such, the group of unit quaternions is called a 'double cover' of SO(3).
Given an axis you wish to rotate about (as a unit vector, **v**) and the amount you wish to rotate, **θ**, (in radians), the process of constructing a unit quaternion for that rotation is straight forward. The x, y, and z components of the unit quaternion are the x, y, and z components of v, multiplied by sin(θ/2) and the fourth component is cos(θ/2).
If you don't know the axis and angle but only have the rotation matrix (i.e. a Revit Transform), there are algorithms for converting from a 3d rotation matrix to a quaternion, though I won't go into any here. Alternatively, it looks like in the latest version of SharpGLTF, AffineTransform has a constructor that takes a 4x4 matrix. To make such a matrix from a Revit Transform, the first 3 columns should be the BasisX, BasisY, and BasisZ of the Transform, with the fourth member of the column being zero, and the last column should be the Transform's origin, with the fourth member of the column being one.
Many thanks to Matthew for this very nice overview!
Barry @bnewcombe adds: ... Quarternion explanations always seem very confusing;
the best explanation (primer) video I found were
the [10 mins GameDev Quaternion tips](https://youtu.be/1yoFjjJRnLY).
Thank you, Barry!