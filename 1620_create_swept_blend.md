---
ai_optimized: true
author: Jeremy Tammik
complexity_score: 2.5
content_type: qa
optimization_date: '2025-12-11T11:44:16.404481'
original_url: https://thebuildingcoder.typepad.com/blog/1620_create_swept_blend.html
post_number: '1620'
reading_time_minutes: 2
series: general
slug: create_swept_blend
source_file: 1620_create_swept_blend.md
tags:
- elements
- geometry
- parameters
- revit-api
- sheets
title: Create Swept Blend
word_count: 469
---

### Create Swept Blend DirectShape in C++
This solution is shared by my colleague Ryuji Ogasawara:
\*\*Question:\*\* I am trying to create a swept blend geometry and assign it to a `DirectShape` element.
The initial code threw an exception when calling the `GeometryCreationUtilities` `CreateSweptBlendGeometry` method, saying:
> Plane of profile loop is not perpendicular to the sweep path at the specified attachment point.
The Revit SDK sample GeometryCreation_BooleanOperation does not include any example code for `CreateSweptBlendGeometry`.
How can I fix this exception?
\*\*Answer:\*\* The path parameters assume that they should be normalized or that the curve has a range of parameterization from 0 to 1.
To use the unnormalized bounds for the curve, you should probably add these calls:
```csharp
pathParams->Add(pathCurve->GetEndParameter(0));
pathParams->Add(pathCurve->GetEndParameter(1));
```
Here is the working code in .NET C++ CLR:
```csharp
Autodesk::Revit::DB::Element^
DirectShapeCreator::CreateDirectShape(
Autodesk::Revit::DB::Document^ document)
{
Autodesk::Revit::DB::Category^ directShapeCategory
= document->Settings->Categories->Item[
Autodesk::Revit::DB::BuiltInCategory::OST_GenericModel];
if (directShapeCategory == nullptr)
return nullptr;
Autodesk::Revit::DB::DirectShape^ directShape
= Autodesk::Revit::DB::DirectShape::CreateElement(
document, directShapeCategory->Id);
if (directShape != nullptr)
{
// Create a path curve
List^ controlPoints = gcnew List;
controlPoints->Add(gcnew XYZ(0, 0, 0));
controlPoints->Add(gcnew XYZ(0, 0, 10));
controlPoints->Add(gcnew XYZ(0, 10, 10));
controlPoints->Add(gcnew XYZ(0, 10, 20));
List^ weights = gcnew List;
weights->Add(1.0);
weights->Add(1.0);
weights->Add(1.0);
weights->Add(1.0);
Curve^ pathCurve = NurbSpline::CreateCurve(
controlPoints, weights);
// Create a bottom profile
List^ bottomProfilePoints = gcnew List;
bottomProfilePoints->Add(gcnew XYZ(5, 5, 0));
bottomProfilePoints->Add(gcnew XYZ(-5, 5, 0));
bottomProfilePoints->Add(gcnew XYZ(-5, -5, 0));
bottomProfilePoints->Add(gcnew XYZ(5, -5, 0));
CurveLoop^ bottomProfile = gcnew CurveLoop;
bottomProfile->Append(Line::CreateBound(
bottomProfilePoints[0], bottomProfilePoints[1]));
bottomProfile->Append(Line::CreateBound(
bottomProfilePoints[1], bottomProfilePoints[2]));
bottomProfile->Append(Line::CreateBound(
bottomProfilePoints[2], bottomProfilePoints[3]));
bottomProfile->Append(Line::CreateBound(
bottomProfilePoints[3], bottomProfilePoints[0]));
// Create a top profile
List^ topProfilePoints = gcnew List;
topProfilePoints->Add(gcnew XYZ(2, 10 + 2, 20));
topProfilePoints->Add(gcnew XYZ(-2, 10 + 2, 20));
topProfilePoints->Add(gcnew XYZ(-2, 10 - 2, 20));
topProfilePoints->Add(gcnew XYZ(2, 10 - 2, 20));
CurveLoop^ topProfile = gcnew CurveLoop;
topProfile->Append(Line::CreateBound(
topProfilePoints[0], topProfilePoints[1]));
topProfile->Append(Line::CreateBound(
topProfilePoints[1], topProfilePoints[2]));
topProfile->Append(Line::CreateBound(
topProfilePoints[2], topProfilePoints[3]));
topProfile->Append(Line::CreateBound(
topProfilePoints[3], topProfilePoints[0]));
List^ profiles = gcnew List;
// Add above profiles
profiles->Add(bottomProfile);
profiles->Add(topProfile);
// which value to be set exactly? He tried 0 and 1.
List^ pathParams = gcnew List;
pathParams->Add(pathCurve->GetEndParameter(0));
pathParams->Add(pathCurve->GetEndParameter(1));
// Create a swept blend geometry.
Solid^ solid
= GeometryCreationUtilities::CreateSweptBlendGeometry(
pathCurve, pathParams, profiles, nullptr);
List^ gs = gcnew List;
gs->Add(solid);
directShape->AppendShape(gs);
}
return directShape;
}
```
The final result looks like this:
![Swept blend DirectShape](img/create_swept_blend.jpg)