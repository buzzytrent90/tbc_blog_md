---
ai_optimized: true
author: Jeremy Tammik
complexity_score: 4.5
content_type: qa
optimization_date: '2025-12-11T11:44:13.738032'
original_url: https://thebuildingcoder.typepad.com/blog/0319_object_relationship_vb.html
post_number: 0319
reading_time_minutes: 2
series: general
slug: object_relationship_vb
source_file: 0319_object_relationship_vb.htm
tags:
- csharp
- elements
- family
- references
- revit-api
title: Object Relationships in VB
word_count: 483
---

### Object Relationships in VB

We recently presented Saikat Bhattacharya's neat little
[object relationship](http://thebuildingcoder.typepad.com/blog/2010/03/object-relationships.html) analyser and displayer, implemented in C#.
Yesterday I received a very friendly submission from Jose Guia saying "Thought I would share my code as well, ... I converted the project to VB for sharing."

I compiled his solution and it works fine for me.
I did run into two little issues which were easy to solve, both of them involving namespaces:

- Different access to imported namespaces.- Different namespace when exporting the external command.

Jose initially used explicit import statements in the VB source code to access the Revit API functionality:
```csharp
Imports Autodesk
Imports Autodesk.Revit
Imports Autodesk.Revit.Elements
```

I much prefer that approach, since it is similar to the syntax used in C#.
However, when I tried to compile the code like that, it caused a number of errors like these
(copy to an editor to see the full truncated lines):
```csharp
'Element' is not accessible in this context because it is 'Friend'.
'Symbol' is not accessible in this context because it is 'Friend'.
'FamilyBase' is not accessible in this context because it is 'Friend'.
'ElementId' is not accessible in this context because it is 'Friend'.
```

The only way I found to resolve this was to remove the explicit import statements from the code and add global namespace importing requests to the project settings instead:

![VB project references settings](img/ObjRelVbReferences.png)

The global imports are defined at the bottom:

![VB project global import settings](img/ObjRelVbImports.png)

The second issue has to do with the exported namespace containing the external command.
The VB code specifies the same export namespace as the C# version in the source code, ObjRel:
```csharp
Namespace ObjRel
    Public Class Command
        Implements IExternalCommand
```

In spite of this, the Visual Studio environment surreptitiously prepends another namespace specified in the global project settings, the so-called root namespace, in this case ObjRelationsVB:
![VB project root namespace settings](img/ObjRelVbRootNamespace.png)

Therefore, the class name entry in Revit.ini needs to be different for the C# and VB versions:

```
ECName2=Object Relationship
ECDescription2=Object Relationship Sample by Saikat
ECClassName2=ObjRel.Command
ECAssembly2=C:\bin\ObjRel.dll

ECName3=Object Relationship VB
ECDescription3=Object Relationship Sample by Jose
ECClassName3=ObjRelationsVB.ObjRel.Command
ECAssembly3=C:\bin\ObjRelationsVB.dll
```

Alternatively, of course, you can set the VB root namespace to an empty string, I hope.
Or, alternatively, of course, you can use the
[Add-In Manager](http://thebuildingcoder.typepad.com/blog/2010/03/addinmanager.html) to
load the application as suggested by Joe, and avoid dealing with Revit.ini at all.
Once loaded, the VB port obviously presents the same functionality as Saikat's original C# version.

Here is Jose's complete
[ObjRelationsVB](zip/ObjRelationsVB.zip)
source code and Visual Studio solution.

Very many thanks to Jose for sharing this code!