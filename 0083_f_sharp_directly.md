---
ai_optimized: true
author: Jeremy Tammik
complexity_score: 5.9
content_type: qa
optimization_date: '2025-12-11T11:44:13.333135'
original_url: https://thebuildingcoder.typepad.com/blog/0083_f_sharp_directly.html
post_number: 0083
reading_time_minutes: 3
series: general
slug: f_sharp_directly
source_file: 0083_f_sharp_directly.htm
tags:
- csharp
- elements
- revit-api
title: Use F# Directly in Revit
word_count: 663
---

### Use F# Directly in Revit

Still struggling to load a native F# module directly as a Revit add-in?
Ralf Huvendiek had a quick look at this issue and created all three different flavours of external commands implemented in F# that can be called directly as external Revit tools:

1. Defined in a namespace.
2. Defined in a module.
3. Defined in a module within a namespace.

The problem I had when
[calling F# via C#](http://thebuildingcoder.typepad.com/blog/2009/01/hello-f-via-c.html)
is caused by the line specifying 'module BuildingCoder', which generates the strange class names.
But even those strange names can be entered and will work in the Revit.ini file.

The decorated name defined for the Execute method in the derived class by adding the interface prefix to it is also no problem.
Apparently, the CLR ignores the name and uses some other method, maybe a position within a table, to determine what method to look up and call.

Here is Ralf's implementation of the three external command classes and their Execute methods;
please watch out for the overly long lines in the F# source code ... you may have to copy and paste to an editor to see the full text:

#### Namespace

```csharp
#light
namespace RevitAddin
open Autodesk.Revit
type RevitAddin() =
interface Autodesk.Revit.IExternalCommand with
member public this.Execute( cData, strMessage : string byref, aElements ) =
strMessage <- "Hello world from RevitAddin.RevitAddin"
Autodesk.Revit.IExternalCommand.Result.Failed
```

#### Module

```csharp
#light
module RevitAddinModule
type RevitAddin() =
interface Autodesk.Revit.IExternalCommand with
member public this.Execute( cData, strMessage : string byref, aElements ) =
strMessage <- "Hello world from RevitAddinModule.RevitAddin"
Autodesk.Revit.IExternalCommand.Result.Failed
```

#### Module within Namespace

```csharp
#light
namespace RevitAddin
module RevitAddinModule
type RevitAddin() =
interface Autodesk.Revit.IExternalCommand with
member public this.Execute( cData, strMessage : string byref, aElements ) =
strMessage <- "Hello world from RevitAddin.RevitAddinModule.RevitAddin"
Autodesk.Revit.IExternalCommand.Result.Failed
```

The corresponding Revit.ini entries for loading the three external commands thus defined look like this:

```
[ExternalCommands]
ECCount=3

ECName1=F# 1
ECDescription1=F# in namespace
ECAssembly1=...\RevitAddin.dll
ECClassName1=RevitAddin.RevitAddin

ECName2=F# 2
ECDescription2=F# in module
ECAssembly2=...\RevitAddin.dll
ECClassName2=RevitAddinModule+RevitAddin

ECName3=F# 3
ECDescription3=F# in namespace and module
ECAssembly3=...\RevitAddin.dll
ECClassName3=RevitAddin.RevitAddinModule+RevitAddin
```

The three dots need to be replaced by the full path name of your assembly.

Here is the complete Visual Studio 2008 F# solution
[FsRevitAddin.zip](zip/FsRevitAddin.zip).

So nothing more to stop you from checking out this exciting new functional language and creating a Revit-based entry for
[Kean's F# programming contest](http://thebuildingcoder.typepad.com/blog/2009/01/f-contest.html).

Several people asked how I retrieved the full class name including namespace prefix from
[Reflector](http://thebuildingcoder.typepad.com/blog/2008/10/converting-between-vb-and-c-and-net-decompilation.html).
In my installation, the full name of the declaring type is listed in the bottom panel when you select a class or method:

![Reflector displaying the full class name](img/reflector_full_class_name.png)

#### 3DS Max, Maya and MotionBuilder Training

Rendering is another topic of interest to some Revit developers, so please note that the Media and Entertainment team are inviting all interested parties to a series of free online trainings to
[learn 3ds Max, Maya or MotionBuilder APIs in just one hour a day](http://through-the-interface.typepad.com/through_the_interface/2009/01/free-online-training-learn-3ds-max-maya-or-motionbuilder-apis-in-just-one-hour-a-day.html)
during the March, April and May time frame.

#### Lunedi Sera in Verona

Last night was very enjoyable
... I took a yoga class with Marco in the Scuola Massalungo, opposite the faculty of economy in the Zona Universitá, then had an enjoyable meal in the eminently recommendable Osteria al Duca in Via Arche Scaligere 2, where I met Mauro Neri and had my 'realest' Italian conversation so far, for a couple of hours
... perfect preparation for the first Revit API introduction in Italiano today.