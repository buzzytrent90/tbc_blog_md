---
ai_optimized: true
author: Jeremy Tammik
complexity_score: 9.0
content_type: qa
optimization_date: '2025-12-11T11:44:16.998748'
original_url: https://thebuildingcoder.typepad.com/blog/1900_forgetypeid.html
post_number: '1900'
reading_time_minutes: 10
series: elements
slug: forgetypeid
source_file: 1900_forgetypeid.md
tags:
- elements
- family
- parameters
- references
- revit-api
- sheets
- views
- windows
title: Forgetypeid
word_count: 1905
---

### PDF Export, ForgeTypeId and Multi-Target Add-In
This is blog post number 1900, just fyi,
cf. [The Building Coder index and table of contents](http://jeremytammik.github.io/tbc/a/#7).
[Revit 2022 has been released](https://thebuildingcoder.typepad.com/blog/2021/04/revit-2022-released.html) and
the time has come to migrate to the new version.
Updates for [RevitLookup](https://github.com/jeremytammik/RevitLookup),
the [Visual Studio Revit add-in wizards](https://github.com/jeremytammik/VisualStudioRevitAddinWizard)
and [The Building Coder samples](https://github.com/jeremytammik/the_building_coder_samples) are
in the works and not done yet... one-man-band lagging...
all Revit 2021 add-ins should work just fine in Revit 2022 as well, though.
Two important features are the parameter API enhancements and built-in PDF export functionality.
Initial issues with these two have already been discussed:
- [Replace deprecated `ParameterType` with `ForgeTypeId`](#2)
- [Multi-target 2021 and 2022 using MSBuild](#3)
- [PDF export default paper format can fail](#4)
- [PDF export output file naming](#5)
- [Five beginner mistakes](#6)
#### Replace Deprecated ParameterType with ForgeTypeId
David Becroft of Autodesk
and Maxim Stepannikov of BIM Planet, aka Максим Степанников or [architect.bim](https://forums.autodesk.com/t5/user/viewprofilepage/user-id/4552025),
helped address
the [Revit API discussion forum](http://forums.autodesk.com/t5/revit-api-forum/bd-p/160) question
on [Revit 2022 ParameterType.Text to ForgeTypeId](https://forums.autodesk.com/t5/revit-api-forum/revit-2022-parametertype-text-to-forgetypeid/m-p/10225741):
\*\*Question:\*\* Could somebody please help me out with this conversion from the deprecated `ParameterType` to `ForgeTypeId` for the Revit 2022 API?
The 'old' code has a line like this:
```csharp
if( parameter.Definition.ParameterType
== ParameterType.Text ) ...
```
What would be the 2022 equivalent for it?
It seems that the left side may be:
```csharp
if( parameter.Definition.GetDataType() == ????) ....
```
But, for some reason, I cannot find what I have to use on right side of the operation... there must be something I am overlooking.
\*\*Answer:\*\* To perform this check you need to create instance of ForgeTypeId class. Use one of the SpecTypeId's properties to get value to compare with. In your case (for text parameter) you need Number property:

```
>>> element.Parameter[BuiltInParameter.ALL_MODEL_MARK].Definition.GetSpecTypeId() == SpecTypeId.Number
True
```

Also take a look at the conversation
on [how to use `ForgeTypeId`](https://forums.autodesk.com/t5/revit-api-forum/forgetypeid-how-to-use/td-p/9439210).
\*\*Response:\*\* Oh dear!
This is super confusing.
I've just checked and parameter definitions that store TEXT have a `SpecTypeId` = `SpecTypeId.Number`.
Now how do you distinguish between actual Numbers and Text?
It looks like the problem is this:
The `Autodesk.Revit.DB.InternalDefinition` class has:
- `ParameterType` – this can be: Invalid | Text | Integer | Number
- `UnitType` – seemingly, for Text type parameters, this is `UT_Number`
Now with `ParameterType` becoming obsolete, we have to use `Parameter.GetSpecTypeId` that SEEMINGLY corresponds to the `UnitType` member above, and for Text parameters like `Comment`, it has a value of `SpecTypeId.Number`!
The question is: How can I know if a Parameter is Text or not in Revit 2022 – without using Definition.ParameterType ?
Same would apply to `YesNo` type parameters... `SpecTypeId` cannot be used to determine if a `ParameterDefinition` is for a `YesNo` type parameter...
And even more: The ONLY place where I can see if a parameter is a YesNo parameter is in the Parameter.Definition.ParameterType !! If ParameterType is obsolete... how to determine if a parameter is YesNo or something else?
\*\*Answer:\*\* It is a well-known fact that unit type of text is number.
Actually, I don't know why :-)
But you definitely should use the `Number` property.
Each Parameter object also has a `StorageType` property.
In case of a Yes/No parameter, its value is `Integer`.
In case of Text parameter, `String`.
I hope this solves your problem.
\*\*Response:\*\* Well, sorry for my ignorance but seemingly there is still one 'minor' issue left for me to be answered:
```csharp
var option = new ExternalDefinitionCreationOptions(
"ExampleParamForge", SpecTypeId.XXX ???);
var definition = definitionGroup.Definitions.Create(
option );
```
Using SpecTypeId.Number creates a Parameter that has the "Type of Parameter" set to Number (obviously!, right?)
How do I create a simple TEXT Parameter definition using the Revit 2022 API?
There must be something here that I am missing.
\*\*Answer:\*\* To create a text parameter, please use `SpecTypeId.String.Text`.
For context, the `ForgeTypeId` properties directly in the `SpecTypeId` class identify the measurable data types, like `SpecTypeId.Length` or `SpecTypeId.Mass`.
The non-measurable data types are organized into nested classes within `SpecTypeId`, like `SpecTypeId.String.Text`, `SpecTypeId.Boolean.YesNo`, `SpecTypeId.Int.Integer`, or `SpecTypeId.Reference.Material`.
Regarding text parameters that report their type as "Number", here's the history:
- Prior to Revit 2021, a `Definition` had a `UnitType` and a `ParameterType`.
The `UnitType` property was only meaningful for parameters with measurable `ParameterType` values, and a parameter with `ParameterType.Text` would report a meaningless `UnitType.Number` value.
- Revit 2021 deprecated the `UnitType` property and replaced it with the `GetSpecTypeId` method.
But the behaviour remained the same – a parameter with `ParameterType.Text` would have `GetSpecTypeId` == `SpecTypeId.Number`.
- Revit 2022 deprecated the `ParameterType` property and the `GetSpecTypeId` method, replacing them both with the `GetDataType` method.
A parameter with `ParameterType.Text` will report `GetDataType()` == `SpecTypeId.String.Text`.
Side note: The `GetDataType` method can also return a category identifier, indicating a Family Type parameter of that category.
Many thanks to Maxim and David for their clarification!
#### Multi-Target 2021 and 2022 Using MSBuild
Josiah Offord very kindly shared his solution to implement a multi-target add-in for several releases of Revit
using [the Microsoft Build Engine MSBuild](https://docs.microsoft.com/en-us/visualstudio/msbuild/msbuild) in both
a [comment](https://thebuildingcoder.typepad.com/blog/2018/06/multi-targeting-revit-versions-cad-terms-texture-maps.html#comment-5339799009)
on [multi-targeting Revit Versions using `TargetFrameworks`](https://thebuildingcoder.typepad.com/blog/2018/06/multi-targeting-revit-versions-cad-terms-texture-maps.html#2) and in a dedicated [Revit API discussion forum](http://forums.autodesk.com/t5/revit-api-forum/bd-p/160) thread
on [multi-targeting 2021 and 2022 using `MSBuild`](https://forums.autodesk.com/t5/revit-api-forum/multi-target-2021-and-2022-using-msbuild/m-p/10235037):
For those interested, you can configure a `.csproj` to multi-target both 2021 and 2022 on .NET Framework 4.8 using MSBuild.
I posted this on a comment in The Building Coder blog and figured it'd be useful here too.
The first task is to choose what .NET 4.8 add-in you want to actively program against.
There can only be one active add-in per .NET framework as far as I know.
In this example, I'm setting my default to Revit 2022 using the custom `RevitVersion` property if it hasn't been configured yet.
```csharp
...
net461;net47;net472;net48TargetFrameworks>
Debug;ReleaseConfigurations>
bin\$(Configuration)\OutputPath>
trueUseWindowsForms>
2022RevitVersion>
...
PropertyGroup>
```
Next, define configurations changes per each version.
```csharp
x64PlatformTarget>
DEBUG;REVIT2018DefineConstants>
bin\$(Configuration)\2018\OutputPath>
PropertyGroup>
x64PlatformTarget>
$(DefineConstants);REVIT2019DefineConstants>
bin\$(Configuration)\2019OutputPath>
PropertyGroup>
x64PlatformTarget>
$(DefineConstants);REVIT2020DefineConstants>
bin\$(Configuration)\2020\OutputPath>
PropertyGroup>
x64PlatformTarget>
$(DefineConstants);REVIT2021DefineConstants>
bin\$(Configuration)\2021OutputPath>
PropertyGroup>
x64PlatformTarget>
$(DefineConstants);REVIT2022DefineConstants>
bin\$(Configuration)\2022OutputPath>
PropertyGroup>
```
Next, load the proper dll references for each version.
```csharp

C:\Program Files\Autodesk\Revit 2018\AdWindows.dllHintPath>
falseEmbedInteropTypes>
falsePrivate>
Reference>
C:\Program Files\Autodesk\Revit 2018\RevitAPI.dllHintPath>
falseEmbedInteropTypes>
falsePrivate>
Reference>
C:\Program Files\Autodesk\Revit 2018\RevitAPIUI.dllHintPath>
falseEmbedInteropTypes>
falsePrivate>
Reference>
ItemGroup>

C:\Program Files\Autodesk\Revit 2019\AdWindows.dllHintPath>
falseEmbedInteropTypes>
falsePrivate>
Reference>
C:\Program Files\Autodesk\Revit 2019\RevitAPI.dllHintPath>
FalsePrivate>
falseEmbedInteropTypes>
Reference>
C:\Program Files\Autodesk\Revit 2019\RevitAPIUI.dllHintPath>
falseEmbedInteropTypes>
falsePrivate>
Reference>
ItemGroup>

C:\Program Files\Autodesk\Revit 2020\AdWindows.dllHintPath>
falseEmbedInteropTypes>
falsePrivate>
Reference>
C:\Program Files\Autodesk\Revit 2020\RevitAPI.dllHintPath>
falseEmbedInteropTypes>
falsePrivate>
Reference>
C:\Program Files\Autodesk\Revit 2020\RevitAPIUI.dllHintPath>
falseEmbedInteropTypes>
falsePrivate>
Reference>
ItemGroup>

C:\Program Files\Autodesk\Revit 2021\AdWindows.dllHintPath>
falseEmbedInteropTypes>
falsePrivate>
Reference>
C:\Program Files\Autodesk\Revit 2021\RevitAPI.dllHintPath>
falseEmbedInteropTypes>
falsePrivate>
Reference>
C:\Program Files\Autodesk\Revit 2021\RevitAPIUI.dllHintPath>
falseEmbedInteropTypes>
falsePrivate>
Reference>
ItemGroup>

C:\Program Files\Autodesk\Revit 2022\AdWindows.dllHintPath>
falseEmbedInteropTypes>
falsePrivate>
Reference>
C:\Program Files\Autodesk\Revit 2022\RevitAPI.dllHintPath>
falseEmbedInteropTypes>
falsePrivate>
Reference>
C:\Program Files\Autodesk\Revit 2022\RevitAPIUI.dllHintPath>
falseEmbedInteropTypes>
falsePrivate>
Reference>
ItemGroup>
```
At the end, you need to configure an additional build for whatever .NET 4.8 Revit add-in you didn't set as the default above. This is nice because it will catch build errors even though the active .NET 4.8 version is something else.
```csharp

Target>

MSBuild>
Target>
```
Side note:
Using a multi-target solution means you need to keep references to all the old versions of Revit.
Be sure to copy out the needed dlls before removing that Revit version from your machine.
Hope this helps.
Thank you very much, Josiah, for this important and timely advice!
#### PDF Export Default Paper Format Can Fail
The new built-in PDF export is a certainly a very useful feature in Revit 2022 and has gathered a lot of interest.
Unfortunately, it also caused some misunderstandings, and a first problem was discovered and described in the thread
on [Revit 2022 PDF export fails with paper format set as default with other parameters](https://forums.autodesk.com/t5/revit-api-forum/revit-2022-pdf-export-fails-with-paper-format-set-as-default/m-p/10223281).
The title says it all, and the development team explain:
By design, if PaperFormat is default, then PaperPlacement should always be Center.
However, there is no restriction ensuring this on the API side.
We should either silently set PaperPlacement to Center during export, or throw an exception notifying the add-in about this.
Currently, in this case, nothing happens and no warning or error is raised.
#### PDF Export Output File Naming
Another thread question
why [2022 PDF exporter can't use the "sheet number" parameter](https://forums.autodesk.com/t5/revit-api-forum/2022-pdf-exporter-cant-use-quot-sheet-number-quot-parameter/m-p/10220287).
Apparently, you have to be sure that the Sheet Number parameter from the "Parameter Type" drop down menu is selected from the Sheet type fields.
Also, if the sheet has no revision the filename, the words 'Current revision' may be inserted into the filename instead.
The development team confirm this fallback behaviour; if the parameter you selected is empty, it will fill the parameter name (Sample Value) in the filename.
The sheet number in the parameter set is confusing due to the parameter type.
The designed scenario is: the customer will use this parameter only in either sheet or view, not in both of them.
If you select a mixed type of both view and sheet with this parameter, one parameter will fallback to its name due to its absence in the view type.
#### Five Beginner Mistakes
Taking a quick look beyond Revit and .NET development for the desktop, the article
on [5 mistakes beginner web developers make – and how to fix them](https://www.freecodecamp.org/news/common-mistakes-beginning-web-development-students-make) addresses
topics that are of use in a non-web environment as well, and that I actually adhere to pretty strictly myself on all platforms
– possibly excepting the last – I am still practicing that:
- Avoid spaces in file names
- Respect case sensitivity
- Understand file paths
- Name the default page `index`
- Take a break
![Taking a break and reading in the Swiss winter sun](/p/2016/2016-01-03_wildhaus/791_jeremy_reading_cropped.jpg)