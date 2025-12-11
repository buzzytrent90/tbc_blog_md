---
ai_optimized: true
author: Jeremy Tammik
complexity_score: 6.1
content_type: qa
optimization_date: '2025-12-11T11:44:16.069253'
original_url: https://thebuildingcoder.typepad.com/blog/1458_unitutils_weight.html
post_number: '1458'
reading_time_minutes: 3
series: general
slug: unitutils_weight
source_file: 1458_unitutils_weight.md
tags:
- elements
- parameters
- revit-api
- sheets
- windows
title: The Building Coder
word_count: 682
---

### UnitUtils Converting Units for Unit Weight
Here comes a quick clarification of the units used for the `UnitWeight` built-in parameter `PHY_MATERIAL_PARAM_UNIT_WEIGHT`.
One example usage is to calculate the total weight of rebars in a project.
A developer encountered the following issue in that process:
#### Question
My Revit add-in calculates total weight of rebars in the document using the `UnitWeight` parameter from the material.
Displayed in Revit, I write a value of 7850 Kg/m3 (kg/cubicmeters).
When using this parameter in code, I assume that the value I get from the parameter is in internal units, which I assume to be kiloNewton / cubic feet.
So when I use this, I expect to be able to use the UnitUtils.
This snapshot from the values displayed by the Visual Studio debugger immediate window show the original value and the converted ones:

```
  var parameterVaule = Parameter.AsDouble();
  7154.4631104000009

  var converted = UnitUtils.ConvertFromInternalUnits(parameterVaule ,Autodesk.Revit.DB.DisplayUnitType.DUT_KILONEWTONS_PER_CUBIC_METER);
  77.01 => This is a correct value.

  var converted2 = UnitUtils.ConvertFromInternalUnits(parameterVaule ,Autodesk.Revit.DB.DisplayUnitType.DUT_KILOGRAMS_PER_CUBIC_METER);
  252657.48031496062 => ??? this not correct!
```

The value string shows 77 kN/cubicmeters, which by manual calculation is 7849.13 Kg/cubicmeters.
Why doesn't the `UnitUtils` class return this value?
I assume that the method `ConvertFromInternalUnits` should be able to convert from the default internal units to the wanted unit?
Is this a case where I can't trust the `UnitUtils`?
#### Sample Code and Workaround
Here is a snippet from the calculation of unit weight from a material.
```csharp
///
/// Get the unit weight of a material.
/// summary>
internal static double GetMaterialEgenvekt(
Document doc,
ref string material,
Element rebarelement )
{
var rType = rebarelement.Document.GetElement(
rebarelement.GetTypeId() ) as ElementType;
var paramMaterial = rType.get_Parameter(
BuiltInParameter.MATERIAL_ID_PARAM );
var mat = doc.GetElement( paramMaterial
.AsElementId() ) as Material;
double egenvekt = 0;
if( mat == null ) return egenvekt;
var property = doc.GetElement(
mat.StructuralAssetId ) as PropertySetElement;
if( property != null )
{
var unitWeightParam = property.get_Parameter(
BuiltInParameter.PHY_MATERIAL_PARAM_UNIT_WEIGHT );
// Not In Use - gives wrong value in metric unit.
var unitWeight = UnitUtils.ConvertFromInternalUnits(
unitWeightParam.AsDouble(),
DisplayUnitType.DUT_KILOGRAMS_PER_CUBIC_METER );
// Manual calculation. In use, and calculates correct.
var egenvektFraMaterial
= EgenVektFraNewtonPerSquareFootMeter(
unitWeightParam.AsDouble() );
if( !( egenvekt > 0 ) )
egenvekt = egenvektFraMaterial;
}
material = mat.Name;
return egenvekt;
}
```
The manual calculation in `EgenVektFraNewtonPerSquareFootMeter` is implemented like this:
```csharp
///
/// Calculate the unit weight from NewtonPerSquareFootMeter
/// summary>
double EgenVektFraNewtonPerSquareFootMeter(
double unitweight )
{
double egenvekt = unitweight / 9.81F;
double meterPerFot = 1000
/ GeoHelper.FootMillimeterKonstant;
egenvekt = egenvekt \* Math.Pow( meterPerFot, 2 );
return egenvekt;
}
```
The constant `GeoHelper.FootMillimeterKonstant` is defined thus;
```csharp
public static double FootMillimeterKonstant
= Math.Round( 304.8, 1 );
```
Hope we can sort out the problem =)
I use the manual way today, after a heads up from a customer who ordered about 900 Kg too little rebars in a project!
#### Explanation
The development team took a look at this issue and provided the following explanation:
`UnitUtils.ConvertFromInternalUnits` does not convert from UnitWeight (77 kN/cubicmeters) to Density (7849.13 Kg/cubicmeters).
`UnitUtils.ConvertFromInternalUnits` displays a value from Revit internal units (kg/(ft²·s²) for UnitWeight and kg/ft³ for Density) to a compatible unit (kN/cubicmeters for UnitWeight and Kg/cubicmeters for Density).
In detail:
- UnitWeight is measured in kN/m3
- Density is measured in kg/m3
UnitWeight and Density are two different units.
So it does not make sense to display some UnitWeight values in kg/m3 – there are different units.
Therefore:
```csharp
// Does not work (different units: value in
// UnitWeight units kg/(ft²·s²) displayed as
// value in Density units kg/m³)
var unitWeight = UnitUtils.ConvertFromInternalUnits(
unitWeightParam.AsDouble(),
DisplayUnitType.DUT_KILOGRAMS_PER_CUBIC_METER);
```
and
```csharp
// Works (same units: value in UnitWeight units
// kg/(ft²·s²) displayed as value in UnitWeight
// units kN/m3)
var converted2 = UnitUtils.ConvertFromInternalUnits(
unitWeightParam.AsDouble(),
DisplayUnitType.DUT_KILONEWTONS_PER_CUBIC_METER);
```
If you want to obtain the density in kg/m3 you can use the `PHY_MATERIAL_PARAM_STRUCTURAL_DENSITY` built-in parameter:
```csharp
var densityParam = property.get_Parameter(
BuiltInParameter.PHY_MATERIAL_PARAM_STRUCTURAL_DENSITY );
// Works (same units: value in Density units kg/ft³
// displayed as value in Density units kg/m³)
var converted = UnitUtils.ConvertFromInternalUnits(
densityParam.AsDouble(),
DisplayUnitType.DUT_KILOGRAMS_PER_CUBIC_METER );
--> converted = 7849.04687 double
```
Or use the formula as you did:
```csharp
UnitWeight = Density \* g
```
I hope this clarifies.
Happily, you already solved this properly yourself.