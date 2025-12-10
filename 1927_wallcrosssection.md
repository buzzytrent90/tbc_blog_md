---
post_number: "1927"
title: "Wallcrosssection"
slug: "wallcrosssection"
author: "Jeremy Tammik"
tags: ['csharp', 'parameters', 'revit-api', 'sheets', 'walls']
source_file: "1927_wallcrosssection.md"
original_url: "https://thebuildingcoder.typepad.com/blog/1927_wallcrosssection.html"
---

### WallCrossSection Renaming in the Revit 2022.1 API
Breaking news from the Revit development team:
#### WallCrossSection versus WallCrossSectionDefinition
Last week, we mentioned the unfortunate breaking change inadvertently introduced with the Revit 2022.1 API update
by [renaming `WallCrossSection` to `WallCrossSectionDefinition`](https://thebuildingcoder.typepad.com/blog/2021/11/revit-20221-sdk-revitlookup-build-and-install.html#3) and
suggested a fix for the `BuiltInParameterGroup` enumeration value.
Here is the workaround suggested by the development team to also address the `ForgeTypeId` modification to support both versions of the API:
As you know from
the [Revit API discussion forum](http://forums.autodesk.com/t5/revit-api-forum/bd-p/160) thread
on [Revit API 2022.1 update change `WallCrossSection` to `WallCrossSectionDefinition`](https://forums.autodesk.com/t5/revit-api-forum/revitapi-2022-update-change-wallcrosssection-to/td-p/10720345),
there was a breaking change introduced in Revit 2022.1:
- `BuiltInParameterGroup.PG_WALL_CROSS_SECTION`
- `ForgeTypeId.WallCrossSection`
were renamed to
- `BuiltInParameterGroup.PG_WALL_CROSS_SECTION_DEFINITION`
- `ForgeTypeId.WallCrossSectionDefinition`
respectively.
A solution for the first name change in the enum value was already suggested in the forum discussion thread:
The actual integer value can be used instead to define a constant like this:
```csharp
var PG_WALL_CROSS_SECTION = (BuiltInParameterGroup) (-5000228);
```
This value be used in both Revit 2022.0 and Revit 2022.1 without causing the problem.
A workaround for the second rename, the `WallCrossSection` property of the `ForgeTypeId` class, can be implemented using Reflection in all .NET languages.
Here is a sample code snippet in C#:
```csharp
using System.Reflection;
. . .
ForgeTypeId id = new ForgeTypeId();
Type type = typeof(GroupTypeId);
PropertyInfo propOld = type.GetProperty("WallCrossSection",
BindingFlags.Public | BindingFlags.Static);
if (null != propOld)
{
id = (ForgeTypeId) propOld.GetValue(null, null);
}
else
{
PropertyInfo propNew = type.GetProperty("WallCrossSectionDefinition",
BindingFlags.Public | BindingFlags.Static);
id = (ForgeTypeId) propNew.GetValue(null, null);
}
```
Or, if you prefer a more succinct version, use this:
```csharp
Type type = typeof(GroupTypeId);
PropertyInfo prop = type.GetProperty("WallCrossSection",
BindingFlags.Public | BindingFlags.Static)
?? type.GetProperty("WallCrossSectionDefinition",
BindingFlags.Public | BindingFlags.Static);
ForgeTypeId id = (ForgeTypeId) prop.GetValue(null, null);
```
We tested it here, and it works for both Revit 2022.0 and Revit 2022.1.
![Non-breaking change](img/tech-comics-non-breaking-change.jpeg "Non-breaking change")

Non-breaking change – © [Datamation](https://www.datamation.com)