---
ai_optimized: true
author: Jeremy Tammik
complexity_score: 5.8
content_type: news
optimization_date: '2025-12-11T11:44:16.022745'
original_url: https://thebuildingcoder.typepad.com/blog/1436_tbc_samples_2017.html
post_number: '1436'
reading_time_minutes: 5
series: general
slug: tbc_samples_2017
source_file: 1436_tbc_samples_2017.md
tags:
- elements
- references
- revit-api
- sheets
- transactions
- views
- walls
title: Tbc Samples 2017
word_count: 1018
---

### The Building Coder Samples 2017
Last night, I migrated The Building Coder samples to Revit 2017:
- [Flat Migration](#2)
- [Updated RvtSamples Include File](#3)
- [Automatic Transaction Mode is Obsolete](#4)
- [Obsolete Plane Constructors and NewPlane Methods](#5)
- [Obsolete NewPlane Method Taking a CurveArray Argument](#6)
- [Replace View.SetVisibility by SetCategoryHidden](#7)
- [Use DirectShape ApplicationId and ApplicationDataId](#8)
- [All Obsolete Revit API Usage Eliminated](#9)
#### Flat Migration
The flat migration was trivial:
- Updated the Revit API references to Revit 2017
- Updated the .NET framework target version from 4.5 to 4.5.2
The compilation succeeded right away with [zero errors and 53 warnings](zip/tbc_samples_2017_migr_01.txt) about obsolete API usage.
Here are the [diffs](https://github.com/jeremytammik/the_building_coder_samples/compare/2016.0.127.3...2017.0.127.0) for this first step.
#### Updated RvtSamples Include File
Next, in order to perform a first test run, I updated the RvtSamples include file, i.e., the text
file [BcSamples.txt](https://github.com/jeremytammik/the_building_coder_samples/blob/master/BcSamples.txt) read
by the RvtSamples external application to load the individual external commands, simply updating the BuildingCoder.dll output build path to point to the new 2017-specific folder.
Once that was done, I was up and running in Revit 2017 – RvtSamples lists The Building Coder sample commands in the three alphabetically sorted sections prefixed with 'ADN Bc':
![The Building Coder samples in Revit 2017](img/tbc_samples_2017.png)
So that was quick and easy.
While we are at it, let's fix those 53 obsolete API usage warnings as well.
#### Automatic Transaction Mode is Obsolete
30 of the 53 warnings concern the obsolete automatic transaction mode:
- Warning CS0618: 'TransactionMode.Automatic' is obsolete: 'This mode is deprecated in Revit 2017.'
We have know for long that it will be going away.
In fact, it
was [publicly announced](http://thebuildingcoder.typepad.com/blog/2012/05/read-only-and-automatic-transaction-modes.html) as
far back as 2012.
So this clean-up is long overdue.
Unfortunately, it will cost me some effort to rewrite these 30 commands.
That is also the reason why it is so long overdue :-)
Later... I fixed all 30 of these external commands, reducing the count to [23 warnings](zip/tbc_samples_2017_migr_02.txt) about obsolete API usage.
Here are the [diffs](https://github.com/jeremytammik/the_building_coder_samples/compare/2017.0.127.0...2017.0.127.1) for this clean-up.
#### Obsolete Plane Constructors and NewPlane Methods
All three overloads of the Application.NewPlane method are obsolete:
- NewPlane(CurveArray) Obsolete. Creates a new geometric plane from a loop of planar curves.
- Public method NewPlane(XYZ, XYZ) Obsolete. Creates a new geometric plane object based on a normal vector and an origin.
- Public method NewPlane(XYZ, XYZ, XYZ) Obsolete. Creates a new geometric plane object based on two coordinate vectors and an origin.
I am using the one taking two `XTZ` arguments representing a normal vector and an origin, and the one taking a `CurveArray` argument.
All three overloads of the Plane class constructor are obsolete:
- Plane() Obsolete. Default constructor
- Public method Plane(XYZ, XYZ) Obsolete. Constructs a Plane object from a normal and an origin represented as XYZ objects. Follows the standard conventions for a planar surface. The constructed Plane object will pass through origin and be perpendicular to normal. The X and Y axes of the plane will be defined arbitrarily.
- Public method Plane(XYZ, XYZ, XYZ) Obsolete. Constructs a Plane object from X and Y axes and an origin represented as XYZ objects. The plane passes through "origin" and is spanned by the basis vectors "xVec" and "yVec".
I am using the one taking two `XYZ` arguments representing a normal vector and an origin.
These three obsolete methods generate the following warnings:
- Warning CS0618: 'Application.NewPlane(XYZ, XYZ)' is obsolete: 'This method is obsolete in Revit 2017. Please use Plane.CreateByNormalAndOrigin() instead.'
- Warning CS0618: 'Plane.Plane(XYZ, XYZ)' is obsolete: 'This method is obsolete in Revit 2017. Please use Plane.CreateByNormalAndOrigin() instead.'
- Warning CS0618: 'Application.NewPlane(CurveArray)' is obsolete: 'This method is obsolete in Revit 2017. Please use CurveLoop.GetPlane() instead.'
Let's go and do what the man says...
I replaced all occurrences of the first two warnings, reducing the count to [5 warnings](zip/tbc_samples_2017_migr_03.txt) about obsolete API usage.
Here are the [diffs](https://github.com/jeremytammik/the_building_coder_samples/compare/2017.0.127.1...2017.0.127.2) for this clean-up.
#### Obsolete NewPlane Method Taking a CurveArray Argument
I still have three calls to the obsolete `Application.NewPlane` method taking a `CurveArray` argument, e.g., in CmdWallProfile.cs:
```csharp
CurveArray curves = creapp.NewCurveArray();
foreach( Curve curve in curveLoop2 )
{
curves.Append( curve.CreateTransformed( offset ) );
}
// Create model lines for an curve loop.
Plane plane = creapp.NewPlane( curves ); // 2016
```
The suggested fix is to use the `CurveLoop.GetPlane` method instead.
Luckily, in this case, the variable `curveLoop2` from which we extract the individual curves to add to the CurveArray collection is already a `CurveLoop` instance, so we can call `GetPlane` on it directly:
```csharp
Plane plane = curveLoop2.GetPlane(); // 2017
```
Unfortunately, we still need to retain and set up the `CurveArray`, because that is a required argument in the subsequent call to `NewModelCurveArray`.
#### Replace View.SetVisibility by SetCategoryHidden
The `View.SetVisibility` is replaced by `SetCategoryHidden`.
Here are the calls used in Revit 2016 and Revit 12017, respectively:
```csharp
view.SetVisibility( catHosts, false ); // 2016
view.SetCategoryHidden( catHosts.Id, true ); // 2017
```
#### Use DirectShape ApplicationId and ApplicationDataId
In Revit 2016, the application and application data GUIDs for a DirectShape element were passed into the constructor.
In Revit 2017, they can be set later using the corresponding properties:
```csharp
DirectShape ds = DirectShape.CreateElement(
doc, e.Category.Id,
_direct_shape_appGUID,
appDataGUID ); // 2016
DirectShape ds = DirectShape.CreateElement(
doc, e.Category.Id ); //2017
ds.ApplicationId = _direct_shape_appGUID; // 2017
ds.ApplicationDataId = appDataGUID∫∫; // 2017
```
#### All Obsolete Revit API Usage Eliminated
The final result of the migration and obsolete API clean-up
is [release 2017.0.127.4](https://github.com/jeremytammik/the_building_coder_samples/releases/tag/2017.0.127.4),
compiling with zero errors and warnings.
The newest version is always available
from [The Building Coder samples GitHub repository](https://github.com/jeremytammik/the_building_coder_samples) master branch.